# Emdee five for life
![Screenshot from 2021-08-02 16-55-32](https://user-images.githubusercontent.com/87865134/127843003-e9c23773-af52-400b-a4db-21b5d747bef2.png)

### - enter md5 of string and submit  
![Screenshot from 2021-08-02 16-56-34](https://user-images.githubusercontent.com/87865134/127843159-cf79a16f-e904-46f9-ba5d-5896bfc98463.png)
  
  - ***Too slow*** alert
    > => automatic submit after get request 

### - use python write script
```python
import requests
import re
import hashlib

url = 'http://139.59.166.56:31420/'

req = requests.session()

rget = req.get(url)

out = re.search(r"<h3 align='center'>(.*?)</h3>", rget.text)

hash = hashlib.md5(out.group(1).encode("utf-8")).hexdigest()

rpost = req.post(url, data = {'hash': hash})

print(rpost.text)
```

### - run script and get flag
![Screenshot from 2021-08-02 17-00-15](https://user-images.githubusercontent.com/87865134/127843676-9bf83f9f-1ab3-4936-b214-d24f4e5c7823.png)


### - Reference:
  - [python requests](https://docs.python-requests.org/en/master/user/quickstart/#make-a-request)
  - [python hashlib](https://docs.python.org/3/library/hashlib.html)
  - [python re](https://docs.python.org/3/library/re.html)
