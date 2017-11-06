---
title: Flask入門
tags:
id: flask-basics
---

[Flask](http://flask.pocoo.org/)

# 環境構築

## インストール

```sh
$ pip install Flask
$ FLASK_APP=hello.py flask run
 * Running on http://localhost:5000/
```

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```
