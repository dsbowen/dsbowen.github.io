---
layout: post
title: "SQLAlchemy-Nav"
date: 2019-10-15 08:59:00
categories: SQLAlchemy
permalink: sqlalchemy-nav
---

<head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" ></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"></script>
</head>

SQLAlchemy-Nav provides [SQLAlchemy Mixins](https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html) for creating navigation bars compatible with [Bootstrap 4](https://getbootstrap.com/docs/4.4/components/navbar/). Its Mixins are:

1. ```NavbarMixin``` for creating navigation bars
2. ```NavitemMixin``` for adding nav-items to the navbar
3. ```DropdownitemMixin``` for adding dropdown-items to nav-items

Readers can find the source code at [https://github.com/dsbowen/sqlalchemy-nav](https://github.com/dsbowen/sqlalchemy-nav).

## License

Publications which use this software should include the following citation for SQLAlchemy-Nav, as well as its dependencies [SQLAlchemy-MutableSoup](https://dsbowen.github.io/sqlalchemy-mutablesoup) and [SQLAlchemy-OrderingItem](https://dsbowen.github.io/sqlalchemy-orderingitem):

Bowen, D.S. (2019). SQLAlchemy-Nav \[Computer software\]. [https://github.com/dsbowen/sqlalchemy-nav](https://dsbowen.github.io/sqlalchemy-nav).

Bowen, D.S. (2020). SQLAlchemy-MutableSoup [Computer software]. [https://dsbowen.github.io/sqlalchemy-mutablesoup](https://dsbowen.github.io/sqlalchemy-mutablesoup).

Bowen, D.S. (2019). SQLAlchemy-OrderingItem [Computer software]. [https://dsbowen.github.io/sqlalchemy-orderingitem](https://dsbowen.github.io/sqlalchemy-orderingitem).

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/sqlalchemy-nav/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-nav
```

### Requirements

SQLAlchemy-Nav requires a [SQLAlchemy](https://www.sqlalchemy.org) database. To use its output in an HTML template, include [Bootstrap](https://getbootstrap.com/) css and scripts. Tested on Bootstrap versions 4.3.1-4.4.1.

### Setup

The following code will get you started with SQLAlchemy-Nav as quickly as possible:

```python
# 1. Import bases from sqlalchemy_nav
from sqlalchemy_nav import NavbarMixin, NavitemMixin, DropdownitemMixin

# 2. Standard session creation
from sqlalchemy import create_engine
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

class Navitem(NavitemMixin, Base):
    __tablename__ = 'navitem'
    
class Dropdownitem(DropdownitemMixin, Base):
    __tablename__ = 'dropdownitem'

# 4. Create the database
Base.metadata.create_all(engine)
```

## Examples

Full examples can be found at [https://github.com/dsbowen/sqlalchemy-nav](https://github.com/dsbowen/sqlalchemy-nav).

### Example 1. Basic use

This example generates HTML for a Bootstrap Navbar using the SQLAlchemy setup above.

```python
navbar = Navbar(label='Home', href='https://dsbowen.github.io')
# Navbars are fixed to the top by default; we remove it here for demonstration
navbar.body.select_one('nav')['class'].remove('fixed-top')
Navitem(navbar, label='About', href='/about')
navitem = Navitem(navbar, dropdown=True, label='Projects')
Dropdownitem(navitem, label='Flask-Worker', href='/flask-worker')
Dropdownitem(navitem, label='SQLAlchemy-Mutable', href='/sqlalchemy-mutable')
session.add(navbar)
session.commit()
print(navbar.render().prettify())
```

Output:

<nav class="navbar navbar-expand-lg navbar-light bg-light">
 <a class="navbar-brand" href="https://dsbowen.github.io">
  Home
 </a>
 <button aria-controls="navbar-1" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbar-1" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbar-1">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item">
    <a class="nav-link" href="/about">
     About
    </a>
   </li>
   <li class="nav-item dropdown">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="" id="navitem-2" role="button">
     Projects
    </a>
    <div aria-labelledby="navitem-2" class="dropdown-menu">
     <a class="dropdown-item" href="/flask-worker">
      Flask-Worker
     </a>
     <a class="dropdown-item" href="/sqlalchemy-mutable">
      SQLAlchemy-Mutable
     </a>
    </div>
   </li>
  </ul>
 </div>
</nav>
<br>

### Example 2: Customization

The HTML of all SQLAlchemy-Nav mixins is contained in a `body` attribute. The `body` is can be treated as a `BeautifulSoup` object (see [SQLAlchemy-MutableSoup](https://dsbowen.github.io/sqlalchemy-mutablesoup)). This gives the programmer full control over the Navbar HTML. 

In this example, we add a search bar to the above navbar.

```python
from bs4 import BeautifulSoup

searchbar_html = '''
<form class="form-inline">
    <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search">
    <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
</form>
'''
searchbar = BeautifulSoup(searchbar_html, 'html.parser')
navbar.body.select_one('nav').append(searchbar)
navbar.body.changed() # See SQLAlchemy-MutableSoup for details
session.commit()
print(navbar.render().prettify())
```

Output:

<nav class="navbar navbar-expand-lg navbar-light bg-light">
 <a class="navbar-brand" href="https://dsbowen.github.io">
  Home
 </a>
 <button aria-controls="navbar-1" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbar-1" data-toggle="collapse" type="button">
  <span class="navbar-toggler-icon">
  </span>
 </button>
 <div class="collapse navbar-collapse" id="navbar-1">
  <ul class="navbar-nav mr-auto">
   <li class="nav-item">
    <a class="nav-link" href="/about">
     About
    </a>
   </li>
   <li class="nav-item dropdown">
    <a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="" id="navitem-2" role="button">
     Projects
    </a>
    <div aria-labelledby="navitem-2" class="dropdown-menu">
     <a class="dropdown-item" href="/flask-worker">
      Flask-Worker
     </a>
     <a class="dropdown-item" href="/sqlalchemy-mutable">
      SQLAlchemy-Mutable
     </a>
    </div>
   </li>
  </ul>
 </div>
 <form class="form-inline">
  <input aria-label="Search" class="form-control mr-sm-2" placeholder="Search" type="search"/>
  <button class="btn btn-outline-success my-2 my-sm-0" type="submit">
   Search
  </button>
 </form>
</nav>
<br>

### Example 3: Toy web app with Flask-SQLAlchemy

This example shows how to use SQLAlchemy-Nav to create dynamic navigation bars in web apps.

Our web app uses [Flask](http://flask.palletsprojects.com/en/1.1.x/) and [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/).

#### Folder structure

For our app, we need a Python file `app.py` and an `index.html` file in a `templates` folder.

```
app.py
templates/
    index.html
```

#### Python file

We begin with a standard setup, then create the ```Navbar``` instance before the first app request. I name the navigation bar so that we can find it later using ```query.filter_by(name='name')```. Finally, we use the navbar to keep a tally of how many times each link on the dropdown menu was visited.
 
```python
# 1. Import Mixins from sqlalchemy_nav
from sqlalchemy_nav import NavbarMixin, NavitemMixin, DropdownitemMixin

# 2. Import Flask classes, methods, and extensions and initialize app
from flask import Flask, Markup, render_template, url_for
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# 3. Use the SQLAlchemy-Nav Mixins to create database models
class Navbar(NavbarMixin, db.Model):
    def render(self):
        return Markup(str(super().render()))

class Navitem(NavitemMixin, db.Model):
    pass

class Dropdownitem(DropdownitemMixin, db.Model):
    pass

# 4. Create a Navbar instance before the first app request
@app.before_first_request
def before_first_request():
    db.create_all()
    navbar = Navbar(name='my-navbar', label='Home', href='https://dsbowen.github.io')
    Navitem(navbar, label='Index ', href='/')
    navitem = Navitem(navbar, label='Pages', dropdown=True)
    Dropdownitem(navitem, label='Page 1 ', href='/page1')
    Dropdownitem(navitem, label='Page 2 ', href='/page2')
    db.session.add(navbar)
    db.session.commit()
    
@app.route('/')
def index():
    # 5. Recover the Navbar instance by name
    navbar = Navbar.query.filter_by(name='my-navbar').first()
    # 6. Tally visits to the index route
    index_item = navbar.navitems[0]
    index_item.label = index_item.label + 'x'
    db.session.commit()
    # 7. Pass the Navbar instance to the html template
    return render_template('index.html', navbar=navbar, content='Index page')

@app.route('/page1')
def page1():
    navbar = Navbar.query.filter_by(name='my-navbar').first()
    page1_item = navbar.navitems[1].dropdownitems[0]
    page1_item.label = page1_item.label + 'x'
    db.session.commit()
    return render_template('index.html', navbar=navbar, content='Page 1')

@app.route('/page2')
def page2():
    navbar = Navbar.query.filter_by(name='my-navbar').first()
    page2_item = navbar.navitems[1].dropdownitems[1]
    page2_item.label = page2_item.label + 'x'
    db.session.commit()
    return render_template('index.html', navbar=navbar, content='Page 2')

if __name__ == '__main__':
    app.run(debug=True)
```

#### HTML template

Next, we create an ```index.html``` file in our templates folder. The index template must:
1. Include Bootstrap css and scripts.
2. Execute ```{% raw %}{{ navbar.render() }}{% endraw %}```.

Recall that ```navbar``` is the ```Navbar``` instance we passed to the index template in step 7.

```html
<html>
  <head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" ></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"></script>
  </head>
  <body>
    {% raw %}{{ navbar.render() }}
    {{ content }}{% endraw %}
  </body>
</html>
```

The app will look like:

<nav class="navbar navbar-expand-lg navbar-light bg-light">
<a class="navbar-brand" href="https://dsbowen.github.io">Home</a>
<button aria-controls="navbar-1" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler" data-target="#navbar-1" data-toggle="collapse" type="button">
<span class="navbar-toggler-icon"></span>
</button>
<div class="collapse navbar-collapse" id="navbar-1">
<ul class="navbar-nav mr-auto"><li class="nav-item active">
<a class="nav-link" href="/">Index x</a>
</li><li class="nav-item dropdown">
<a aria-expanded="false" aria-haspopup="true" class="nav-link dropdown-toggle" data-toggle="dropdown" href="" id="navitem-2" role="button">Pages</a>
<div aria-labelledby="navitem-2" class="dropdown-menu"><a class="dropdown-item" href="/page1">Page 1 </a><a class="dropdown-item" href="/page2">Page 2 </a></div>
</li></ul>
</div>
</nav>
Index page
<br>

#### Run the example

To run the working example, run the app:

```bash
$ export FLASK_APP=<app>
$ flask run
```

Alternatively, you can clone the SQLAlchemy-Nav repo and run.

```bash
$ git clone https://github.com/dsbowen/sqlalchemy-nav.git
$ cd sqlalchemy-nav
$ export FLASK_APP=flask_example
$ flask run
```

Then open ```http://localhost:5000``` in your browser.