title: "Flask页面相关知识"
date: 2016-01-11 11:50:28
tags: ["Flask"]
categories: ["Python"]
---

### 更改jinja2模板的默认修饰符

> [different delimiters in jinja2 + flask](https://gist.github.com/lost-theory/3925738#file-gistfile1-py-L18)

在**Flask**中的*jinja2*模板中输出值的修饰符号与`AngularJs`中的修饰符一样，为了解决冲突，可以自定义默认模板的修饰符:
```py
class CustomFlask(Flask):
    jinja_options = Flask.jinja_options.copy()
    jinja_options.update(dict(
        block_start_string='<%',
        block_end_string='%>',
        variable_start_string='%%', #替换{{
        variable_end_string='%%',   #替换}}
        comment_start_string='<#',
        comment_end_string='#>',
    ))

//默认写法app = Flask(__name__)这里使用了上面自定义的类
app = CustomFlask(__name__)
```
这样输出值的修饰符就变成了`%%`,避开了与`AngularJS`的输出修饰符冲突。

----

### 输出静态文件

> [python flask - serving static files](http://stackoverflow.com/questions/15883874/python-flask-serving-static-files)

```py
from flask import url_for, redirect

@app.route('/')
def home():
    return redirect(url_for('static', filename='index.html'))
```