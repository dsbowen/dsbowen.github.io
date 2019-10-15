---
layout: post
title: "SQLAlchemy-Nav"
date: 2019-10-15 08:59:00
categories: SQLAlchemy
permalink: sqlalchemy-nav
---

<head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
</head>

SQLAlchemy-Nav provides [SQLAlchemy Mixins](https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html) for creating Bootstrap navigation bars. Its Mixins are:

1. ```NavbarMixin``` for creating navigation bars
2. ```BrandMixin``` for adding a brand to the navbar
3. ```NavitemMixin``` for adding nav-items to the navbar
4. ```DropdownitemMixin``` for adding dropdown-items to nav-items

## License

Publications which use this software should include the following citation for SQLAlchemy-Nav and its dependency, [SQLAlchemy-Mutable](https://pypi.org/project/sqlalchemy-mutable/):

Bowen, D.S. (2019). SQLAlchemy-Nav \[Computer software\]. [https://github.com/dsbowen/sqlalchemy-nav](https://github.com/dsbowen/sqlalchemy-nav)

Bowen, D.S. (2019). SQLAlchemy-Mutable \[Computer software\]. [https://github.com/dsbowen/sqlalchemy-mutable](https://github.com/dsbowen/sqlalchemy-mutable)

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/sqlalchemy-nav/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-nav
```

### Requirements

SQLAlchemy-Nav requires [SQLAlchemy](https://www.sqlalchemy.org). To use its output in an html template, include [Bootstrap](https://getbootstrap.com/) css and scripts. Tested on Bootstrap v4.3.1.

### Setup

The following code will get you started with SQLAlchemy-Nav as quickly as possible:

```python
# 1. Import classes from sqlalchemy_nav
from sqlalchemy_nav import BrandMixin, DropdownitemMixin, NavbarMixin, NavitemMixin

# 2. Standard session creation
from sqlalchemy import create_engine, Column, Integer
from sqlalchemy.orm import sessionmaker, scoped_session
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine('sqlite:///:memory:')
session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)
session = Session()
Base = declarative_base()

# 3. Use the SQLAlchemy-Nav Mixins to create database models
class Navbar(NavbarMixin, Base):
    __tablename__ = 'navbar'

class Brand(BrandMixin, Base):
    __tablename__ = 'brand'

class Navitem(NavitemMixin, Base):
    __tablename__ = 'navitem'
    
class Dropdownitem(DropdownitemMixin, Base):
    __tablename__ = 'dropdownitem'

# 4. Create the database
Base.metadata.create_all(engine)
```

## Examples

### Example 1. Use with SQLAlchemy

This example generates html for a Bootstrap Navbar using the SQLAlchemy setup above. You can find the full setup and example [here](https://github.com/dsbowen/sqlalchemy-nav/blob/master/example.py).

```python
bar = Navbar()
Brand(bar=bar, url='/', label='dsbowen.github.io')
Navitem(bar=bar, url='/about', label='About')
item = Navitem(bar=bar, label='Projects')
Dropdownitem(item=item, url='/flask-worker', label='Flask-Worker')
Dropdownitem(item=item, url='/sqlalchemy-mutable', label='SQLAlchemy-Mutable')
Dropdownitem(item=item, url='/sqlalchemy-nav', label='SQLAlchemy-Nav')
```

You can now include `bar.render()` in an html template with Bootstrap css and scripts. The output will be:

<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
 <a class="navbar-brand" href="/">
  dsbowen.github.io
 </a>
 <button aria-controls="navbarSupportedContent1" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbarSupportedContent1" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbarSupportedContent1">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item">
    <a class="nav-link" href="/about">
     About
    </a>
   </li>
   <li class="nav-item">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" id="navbarDropdownNone" role="button">
     Projects
    </a>
    <div aria-labelledby="navbarDropdownNone" class="dropdown-menu">
     <a class="dropdown-item" href="/flask-worker">
      Flask-Worker
     </a>
     <a class="dropdown-item" href="/sqlalchemy-mutable">
      SQLAlchemy-Mutable
     </a>
     <a class="dropdown-item" href="/sqlalchemy-nav">
      SQLAlchemy-Nav
     </a>
    </div>
   </li>
  </ul>
 </div>
</nav>
<br/>

## Example 2: Toy web app with Flask-SQLAlchemy

This example shows how to use SQLAlchemy-Nav to create dynamic navigation bars in web apps. You can find the full setup and example [here](https://github.com/dsbowen/sqlalchemy-nav/blob/master/flask_example.py).

Our web app uses [Flask](http://flask.palletsprojects.com/en/1.1.x/) and [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/).

### Python file

We begin by creating the ```Navbar``` instance before the first app request. I name the navigation bar so that we can find it later using ```query.filter_by(name='name')```. We then use the navbar to keep a tally of how many times each link on the dropdown menu was visited.
 
```python
# 1. Import Mixins from sqlalchemy_nav
from sqlalchemy_nav import BrandMixin, DropdownitemMixin, NavbarMixin, NavitemMixin

# 2. Import Flask classes, methods, and extensions and initialize app
from flask import Flask, render_template, url_for
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# 3. Use the SQLAlchemy-Nav Mixins to create database models
class Navbar(NavbarMixin, db.Model):
    pass

class Brand(BrandMixin, db.Model):
    pass

class Navitem(NavitemMixin, db.Model):
    pass

class Dropdownitem(DropdownitemMixin, db.Model):
    pass

# 4. Create a Navbar instance before the first app request
@app.before_first_request
def before_first_request():
    db.create_all()
    bar = Navbar(name='my navbar')
    Brand(bar=bar, url='https://pypi.org/project/sqlalchemy-nav', label='SQLAlchemy-Nav')
    Navitem(bar=bar, url=url_for('index'), label='Index ')
    item = Navitem(bar=bar, label='Dropdown')
    Dropdownitem(item=item, url=url_for('page1'), label='Page 1 ')
    Dropdownitem(item=item, url=url_for('page2'), label='Page 2 ')
    db.session.add(bar)
    db.session.commit()
    
@app.route('/')
def index():
    # 5. Recover the Navbar instance by name
    bar = Navbar.query.filter_by(name='my navbar').first()
    # 6. Tally visits to the index route
    bar.navitems[0].label += 'x'
    db.session.commit()
    # 7. Pass the Navbar instance to the html template
    return render_template('index.html', bar=bar, content='Index page')

@app.route('/page1')
def page1():
    bar = Navbar.query.filter_by(name='my navbar').first()
    bar.navitems[1].dropdownitems[0].label += 'x'
    db.session.commit()
    return render_template('index.html', bar=bar, content='Page 1')

@app.route('/page2')
def page2():
    bar = Navbar.query.filter_by(name='my navbar').first()
    bar.navitems[1].dropdownitems[1].label += 'x'
    db.session.commit()
    return render_template('index.html', bar=bar, content='Page 2')

if __name__ == '__main__':
    app.run()
```

### Html template

Next, we create an ```index.html``` file in our templates folder. The index template must:
1. Include Bootstrap css and scripts.
2. Execute ```{{ bar.render() }}```.

Recall that ```bar``` is the ```Navbar``` instance we passed to the index template in step 7.

```html
<html>
  <head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
  </head>
  <body>
    {% raw %}{{ bar.render() }}
    {{ content }}{% endraw %}
  </body>
</html>
```

The app will look like:

<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
 <a class="navbar-brand" href="https://dsbowen.github.io/sqlalchemy-nav">
  SQLAlchemy-Nav
 </a>
 <button aria-controls="navbarSupportedContent2" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbarSupportedContent2" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbarSupportedContent2">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item active">
    <a class="nav-link" href="/">
     Index x
    </a>
   </li>
   <li class="nav-item">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" id="navbarDropdown2" role="button">
     Dropdown
    </a>
    <div aria-labelledby="navbarDropdown2" class="dropdown-menu">
     <a class="dropdown-item" href="/page1">
      Page 1
     </a>
     <a class="dropdown-item" href="/page2">
      Page 2 xx
     </a>
    </div>
   </li>
  </ul>
 </div>
</nav>
Index
<br/>

### View the example

To view the working example, run the app:

```bash
$ python my_app.py
```

Alternatively, you can clone the SQLAlchemy-Nav repo and run ```flask_example.py```.

```bash
$ git clone https://github.com/dsbowen/sqlalchemy-nav.git
$ cd sqlalchemy-nav
$ python flask_example.py
```

Then open ```http://localhost:5000``` in your browser.

## Customization and defaults

Change a tag's classes with its `classes` attribute:

```python
bar.classes.remove('navbar-dark')
bar.classes.remove('bg-dark')
bar.classes.append('navbar-light')
bar.classes.append('bg-light')
```

Applied to our earlier example:


<nav class="navbar navbar-expand-lg navbar-light bg-light">
 <a class="navbar-brand" href="/">
  dsbowen.github.io
 </a>
 <button aria-controls="navbarSupportedContent3" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbarSupportedContent3" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbarSupportedContent3">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item">
    <a class="nav-link" href="/about">
     About
    </a>
   </li>
   <li class="nav-item">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" id="navbarDropdownNone" role="button">
     Projects
    </a>
    <div aria-labelledby="navbarDropdownNone" class="dropdown-menu">
     <a class="dropdown-item" href="/flask-worker">
      Flask-Worker
     </a>
     <a class="dropdown-item" href="/sqlalchemy-mutable">
      SQLAlchemy-Mutable
     </a>
     <a class="dropdown-item" href="/sqlalchemy-nav">
      SQLAlchemy-Nav
     </a>
    </div>
   </li>
  </ul>
 </div>
</nav>
<br/>

Add custom html, such as a search bar:

```python
bar.custom_html = """
<form class="form-inline">
    <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search">
    <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
</form>
"""
```

<nav class="navbar navbar-expand-lg navbar-light bg-light">
 <a class="navbar-brand" href="/">
  dsbowen.github.io
 </a>
 <button aria-controls="navbarSupportedContent4" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbarSupportedContent4" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbarSupportedContent4">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item">
    <a class="nav-link" href="/about">
     About
    </a>
   </li>
   <li class="nav-item">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" id="navbarDropdownNone" role="button">
     Projects
    </a>
    <div aria-labelledby="navbarDropdownNone" class="dropdown-menu">
     <a class="dropdown-item" href="/flask-worker">
      Flask-Worker
     </a>
     <a class="dropdown-item" href="/sqlalchemy-mutable">
      SQLAlchemy-Mutable
     </a>
     <a class="dropdown-item" href="/sqlalchemy-nav">
      SQLAlchemy-Nav
     </a>
    </div>
   </li>
  </ul>
  <form class="form-inline">
   <input aria-label="Search" class="form-control mr-sm-2" placeholder="Search" type="search"/>
   <button class="btn btn-outline-success my-2 my-sm-0" type="submit">
    Search
   </button>
  </form>
 </div>
</nav>