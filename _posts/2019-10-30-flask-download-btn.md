---
layout: post
title: "Flask-Download-Btn"
date: 2019-10-30 11:08:00
categories: Flask
permalink: flask-download-btn
---

<head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>

Flask-Download-Btn defines a [SQLALchemy Mixin](https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html) for creating [Bootstrap](https://getbootstrap.com/) download buttons in a [Flask](https://palletsprojects.com/p/flask/) application.

Its key features are:
1. *Automatic enabling and disabling*: A download button is automatically disabled on click and re-enabling on download completion.
2. *Web form handling*: Applications can modify a download button on click based on web form responses.
3. *Progress bar*: Download buttons report progress using a progress bar updated with server-sent events.

Additional features include:
1. *Automatic Cross-Site Request Forgery (CSRF) prevention*: Download button routes are automatically protected using CSRF tokens.
2. *Caching*: Download buttons support cached download files.
3. *Customizable styles*: Download buttons and progress bars support custom styling.

Source code: [https://github.com/dsbowen/flask-download-btn](https://github.com/dsbowen/flask-download-btn)

## License

Publications which use this software should include the following citation for SQLAlchemy-Function and its dependencies, [SQLAlchemy-Function](https://dsbowen.github.io/sqlalchemy-function) and [SQLAlchemy-Mutable](https://dsbowen.github.io/sqlalchemy-mutable):

Bowen, D.S. (2019). Flask-Download-Btn\[Computer software\]. [https://dsbowen.github.io/flask-download-btn](https://dsbowen.github.io/flask-download-btn).

Bowen, D.S. (2019). SQLAlchemy-Function \[Computer software\]. [https://dsbowen.github.io/sqlalchemy-function](https://dsbowen.github.io/sqlalchemy-function).

Bowen, D.S. (2019). SQLAlchemy-Mutable \[Computer software\]. [https://dsbowen.github.io/sqlalchemy-mutable](https://dsbowen.github.io/sqlalchemy-mutable).

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/flask-download-btn/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U flask-download-btn
```

### Requirements

Flask-Download-Btn should be used with a [Flask](https://palletsprojects.com/p/flask/) application with a [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) database. It also requires [Bootstrap](https://getbootstrap.com/) CSS and Javascript.

### Setup

The following code will get you started with Flask-Download-Btn as quickly as possible.

Our initial file structure will be:

```
app.py
hello_world.txt
templates/
    index.html
```

In `app.py`:

```python
# 1. Import download button manager and mixins
from flask_download_btn import CreateFileMixin, DownloadBtnManager, DownloadBtnMixin, HandleFormMixin

from flask import Flask, render_template, session
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)
# 2. Initialize download button manager with application and database
download_btn_manager = DownloadBtnManager(app, db=db)

# 3. Create download button model and register it with the manager
@DownloadBtnManager.register
class DownloadBtn(DownloadBtnMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)

# 4. Create CreateFile and HandleForm models
class CreateFile(CreateFileMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    bnt_id = db.Column(db.Integer, db.ForeignKey('download_btn.id'))

class HandleForm(HandleFormMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    bnt_id = db.Column(db.Integer, db.ForeignKey('download_btn.id'))

# 5. Create database tables
db.create_all()

# EXAMPLE CODE HERE

if __name__ == '__main__':
    app.run(debug=True)
```

In `templates/index.html`:

```html
<html>
    <head>
        <!-- 1. Include Bootstrap CSS and Javascript -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
        <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
        <!-- 2. Include download button script -->
        {% raw %}{{ download_btn.script() }}{% endraw %}
    </head>
    <body>
        <!-- 3. Render the download button and progress bar -->
        {% raw %}{{ download_btn.render_btn() }}
        {{ download_btn.render_progress() }}{% endraw %}
    </body>
</html>
```

## Examples

See the full setup and examples [here](https://github.com/dsbowen/flask-download-btn/blob/master/app.py).

### Helper methods

Before our example routes, we define helper varaibles and methods for creating and storing buttons:
1. Download URLs for `hello_world.txt` and `hello_moon.txt`.
2. A method to add and commit a download button to the session.
3. A method to look up a button.
4. A method to clear the session before the first app request.

```python
HELLO_WORLD_URL = 'https://test-bucket2357.s3.us-east-2.amazonaws.com/hello_world.txt'
HELLO_MOON_URL = 'https://test-bucket2357.s3.us-east-2.amazonaws.com/hello_moon.txt'

def add_to_session(btn, key):
    db.session.add(btn)
    db.session.commit()
    session[key] = btn.id

def get_btn(key):
    if key in session:
        return DownloadBtn.query.get(session[key])

@app.before_first_request
def clear_session():
    session.clear()
```

### Example 1: Basic use

Example 1 illustrates the basic use of Flask-Download-Btn. `text` is the button text, and `downloads` is a list of `(URL, attachment_filename)` tuples to download.

```python
@app.route('/')
def index():
    btn = get_btn('example1')
    if not btn:
        btn = DownloadBtn()
        btn.text = 'Download Example 1'
        btn.downloads = [(HELLO_WORLD_URL, 'hello_world.txt')]
        add_to_session(btn, 'example1')
    return render_template('index.html', download_btn=btn)
```

Your download button will look like:

<button id="DownloadBtn-1-btn" type="button" class="btn btn-primary w-100" style="">
    Download Example 1
</button>

In your application, the button is disabled when clicked until it downloads `hello_world.txt`. (The button above is not functional).

### Example 2: Multiple files

Download buttons support downloading multiple files.

```python
@app.route('/example2')
def example2():
    """Example 2: Multiple files"""
    btn = get_btn('example2')
    if not btn:
        btn = DownloadBtn()
        btn.text = 'Download Example 2'
        btn.downloads = [
            (HELLO_WORLD_URL, 'hello_world.txt'), 
            (HELLO_MOON_URL, 'hello_moon.txt')
        ]
        add_to_session(btn, 'example2')
    return render_template('index.html', download_btn=btn)
```

### Example 3: Callback routes

Download buttons can replace the window with a `callback` routes after the files have downloaded.

```python
from flask import url_for

@app.route('/example3')
def example3():
    btn = get_btn('example3')
    if not btn:
        btn = DownloadBtn()
        btn.text = 'Download Example 3'
        btn.downloads = [(HELLO_WORLD_URL, 'hello_world.txt')]
        btn.callback = url_for('download_success')
        add_to_session(btn, 'example3')
    return render_template('index.html', download_btn=btn)

@app.route('/download-success')
def download_success():
    return 'Download Successful'
```

### Example 4: Web form handling

Download buttons can respond to input from web forms using `HandleForm` [function models](https://dsbowen.github.io/sqlalchemy-function). You can access a download button's list of `HandleForm` models with its `handle_form_functions` attribute. On click, the download button executes its `HandleForm` functions in list order.

When executed, `HandleForm` models execute their function (`func`), passing in their button as the first argument and the `flask.request.form` dictionary as the second argument, followed by the `HandleForm` instance's `args` and `kwargs`.

In this example, we create a web form in the `example4.html` template. This form asks clients to select which files they would like to download. Our download button's `HandleForm` function sets its `downloads` to the files selected in the web form.

In `templates/example4.html`:

```html
<html>
    <head>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
        <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
        {% raw %}{{ download_btn.script() }}{% endraw %}
    </head>
    <body>
        <form>
            <div id="selectFiles" name="selectFiles">
                <p>Select files to download.</p>
                <div class="form-check">
                    <input id="helloWorld" name="selectFiles" type="checkbox" class="form-check-input" value="hello_world.txt">
                    <label class="form-check-label" for="helloWorld">Hello World</label>
                </div>
                <div class="form-check">
                    <input id="helloMoon" name="selectFiles" type="checkbox" class="form-check-input" value="hello_moon.txt">
                    <label class="form-check-label" for="helloMoon">Hello Moon</label>
                </div>
            </div>
        </form>
        {% raw %}{{ download_btn.render_btn() }}
        {{ download_btn.render_progress() }}{% endraw %}
    </body>
</html>
```

In `app.py`:

```python
@app.route('/example4')
def example4():
    btn = get_btn('example4')
    if not btn:
        btn = DownloadBtn()
        btn.text = 'Download Example 4'
        HandleForm(btn, func=select_files)
        add_to_session(btn, 'example4')
    return render_template('example4.html', download_btn=btn)

def select_files(btn, resp):
    btn.downloads = []
    files = resp.getlist('selectFiles')
    if 'hello_world.txt' in files:
        btn.downloads.append((HELLO_WORLD_URL, 'hello_world.txt'))
    if 'hello_moon.txt' in files:
        btn.downloads.append((HELLO_MOON_URL, 'hello_moon.txt'))
```

This code creates the following page:

<form>
<div id="selectFiles" name="selectFiles">
    <p>Select files to download.</p>
    <div class="form-check">
        <input id="helloWorld_exmpl4" name="selectFiles" type="checkbox" class="form-check-input" value="hello_world.txt">
        <label class="form-check-label" for="helloWorld_exmpl4">Hello World</label>
    </div>
    <div class="form-check">
        <input id="helloMoon_exmpl4" name="selectFiles" type="checkbox" class="form-check-input" value="hello_moon.txt">
        <label class="form-check-label" for="helloMoon_exmpl4">Hello Moon</label>
    </div>
</div>
</form>
<br>
<button id="DownloadBtn-1-btn" type="button" class="btn btn-primary w-100" style="">
    Download Example 4
</button>

**Note**: If you have multiple web forms on a page, select the form you want the download button to handle by settings its `form_id` attribute.

### Example 5: File creation

A Download Button also has a list of `CreateFile` function models. You can access a download button's `CreateFile` models with its `create_file_functions` attribute. After form handling, the download button executes its `CreateFile` functions in list order.

When executed, `CreateFile` models execute their function (`func`), passing in their button as the first argument, followed by the `CreateFile` instance's `args` and `kwargs`.

As the name suggests, `CreateFile` models are often used to create download files. In principal, these models can perform other functions as well.

`CreateFile` models return a generator, which updates the progress bar using the download button's `reset` and `report` methods. Both methods take optional `stage` (e.g. 'Creating File 1', 'Creating File 2') and `pct_complete` (e.g. 50.0) arguments. 

`reset` refreshes the progress bar for abrupt transitions, such as resetting the progress bar from 100% to 0% when starting a new stage. `report` updates the current progress bar without refreshing it for smooth transitions.

In this example, we assign two `CreateFile` functions to our download button. Because these functions take a long time to run, we cache the results by setting the button's `cache` attribute to `default`.

```python
import time

@app.route('/example5')
def example5():
    btn = get_btn('example5')
    if not btn:
        btn = DownloadBtn()
        btn.text = 'Download Example 5'
        btn.cache = 'default'
        CreateFile(btn, func=create_file1, kwargs={'seconds': 5})
        CreateFile(btn, func=create_file2, kwargs={'centiseconds': 400})
        btn.downloads = [
            (HELLO_WORLD_URL, 'hello_world.txt'), 
            (HELLO_MOON_URL, 'hello_moon.txt')
        ]
        add_to_session(btn, 'example5')
    return render_template('index.html', download_btn=btn)

def create_file1(btn, seconds):
    stage = 'Creating File 1'
    yield btn.reset(stage=stage, pct_complete=0)
    if not btn.downloaded:
        for i in range(seconds):
            yield btn.report(stage, 100.0*i/seconds)
            time.sleep(1)
        yield btn.report(stage, 100.0)
        time.sleep(1)

def create_file2(btn, centiseconds):
    stage = 'Creating File 2'
    yield btn.reset(stage, 0)
    if not btn.downloaded:
        for i in range(centiseconds):
            yield btn.report(stage, 100.0*i/centiseconds)
            time.sleep(.01)
        yield btn.report(stage, 100)
        time.sleep(.01)
```

During file creation, our download button and progress bar look like:

<button id="DownloadBtn-1-btn" type="button" class="btn btn-primary w-100" style="" disabled>
    Download Example 5
</button>
<div id="DownloadBtn-1-progress">
    <div class="progress position-relative" style="height: 25px; background-color: #C8C8C8; margin-top: 10px; margin-bottom: 10px; box-shadow: 0 1px 2px rgba(0, 0, 0, 0.25) inset;">
        <div id="DownloadBtn-1-progress-bar" class="progress-bar" role="progressbar" width="0%" style="transition: width .5s; width: 50%">
            <div id="DownloadBtn-1-progress-text" class="justify-content-center d-flex position-absolute w-100 align-items-center">
            Creating File 1: 50%  
            </div>
        </div>  
    </div>
</div>
<br>

### Example 6: With style

Example 6 combines elements from previous examples with styling.

In `templates/example6.html`:

```html
<html>
    <head>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
        <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
        {% raw %}{{ download_btn.script() }}{% endraw %}
    </head>
    <body>
    <div class="container h-100">
    <div class="row h-100 justify-content-center align-items-center">
        <form>
            {% raw %}{{ download_btn.render_progress() }}{% endraw %}
            <div id="selectFiles" name="selectFiles">
                <p>Select files to download.</p>
                <div class="custom-control custom-checkbox">
                    <input id="helloWorld" name="selectFiles" type="checkbox" class="custom-control-input" value="hello_world.txt">
                    <label class="custom-control-label" for="helloWorld">Hello World</label>
                </div>
                <div class="custom-control custom-checkbox">
                    <input id="helloMoon" name="selectFiles" type="checkbox" class="custom-control-input" value="hello_moon.txt">
                    <label class="custom-control-label" for="helloMoon">Hello Moon</label>
                </div>
            </div>
            <br>
            {% raw %}{{ download_btn.render_btn() }}{% endraw %}
        </form>
    </div>
    </div>
    </body>
</html>
```

In `app.py`:

```python
@app.route('/example6')
def example6():
    btn = get_btn('example6')
    if not btn:
        btn = DownloadBtn()
        btn.btn_classes.remove('btn-primary')
        btn.btn_classes.append('btn-outline-primary')
        btn.progress_classes.append('progress-bar-striped')
        btn.progress_classes.append('progress-bar-animated')
        btn.text = 'Download Example 6'
        btn.cache = 'default'
        HandleForm(btn, func=select_files)
        CreateFile(btn, func=create_file1, kwargs={'seconds': 4})
        CreateFile(btn, func=create_file2, kwargs={'centiseconds': 300})
        btn.download_msg = 'Download Complete'
        add_to_session(btn, 'example6')
    return render_template('example6.html', download_btn=btn)
```

During download, our page will look like:

<div class="container h-100">
<div class="row h-100 justify-content-center align-items-center">
<form>
    <div id="DownloadBtn-1-progress"><div class="progress position-relative" style="height: 25px; background-color: #C8C8C8; margin-top: 10px; margin-bottom: 10px; box-shadow: 0 1px 2px rgba(0, 0, 0, 0.25) inset;">
        <div id="DownloadBtn-1-progress-bar" class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" width="0%" style="transition: width .5s; width: 50%">
        <div id="DownloadBtn-1-progress-text" class="justify-content-center d-flex position-absolute w-100 align-items-center">
        Creating File 1: 50%
        </div>
    </div>
</div>
</div>
    <div id="selectFiles" name="selectFiles">
        <p>Select files to download.</p>
        <div class="custom-control custom-checkbox">
            <input id="helloWorld_exmpl5" name="selectFiles" type="checkbox" class="custom-control-input" value="hello_world.txt">
            <label class="custom-control-label" for="helloWorld_exmpl5">Hello World</label>
        </div>
        <div class="custom-control custom-checkbox">
            <input id="helloMoon_exmpl5" name="selectFiles" type="checkbox" class="custom-control-input" value="hello_moon.txt">
            <label class="custom-control-label" for="helloMoon_exmpl5">Hello Moon</label>
        </div>
    </div>
    <br>
    <button id="DownloadBtn-1-btn" type="button" class="btn w-100 btn-outline-primary" style="" disabled>
        Download Example 6
    </button>
</form>
</div>
</div>
<br>

## Customization and defaults

### Manager defaults

Default settings for all download buttons can be changed by editing `download_btn_manager.default_settings`. For example, to change the default to striped progress bars:

```python
download_btn_manager.default_settings.progress_classes.append('progress-bar-striped')
```

### Styling

A download button's classes and style can be changed with a download button's `btn_classes` and `btn_style` attribute, respectively. `btn_classes` is a list of button classes, while `style` is dictionary mapping CSS property names to values (e.g. `{'height': '20px'}`).

Further customization can be achieved by defining a custom button template and setting a download button's `btn_template` attribute accorudingly:

```python
my_button.btn_template = 'my-btn-template.html'
```

See my [original button template](https://github.com/dsbowen/flask-download-btn/blob/master/flask_download_btn/templates/download_btn/button.html) for custom button template specifications.

A button's `progress_classes`, `progress_style`, and `progress_template` attributes perform analogous functions for its progress bar.

### Cache and CSRF management

Cached cookies and zip files associated with a session can be cleared with a button's `clear_session` method. To clear a button's CSRF token from the user's session, call its `clear_csrf` method.

**Note**: These methods will only clear the cache and CSRF token for the request context in which they are called.

### Transition speed

You can set the initial transition speed of a button's progress bar with its `init_transition_speed` attribute (`'.5s'` by default).

Download buttons then automatically update their transition speed to accomodate the rate of server-sent events (progress reports and resets). You can override this default using a button's `transition_speed` method, as in the following code snipet:

```python
def create_file(btn):
    for i in range(100):
        yield btn.report(stage='Creating File', pct_complete=i)
        yield btn.transition_speed('.5s')
        # Code here
```