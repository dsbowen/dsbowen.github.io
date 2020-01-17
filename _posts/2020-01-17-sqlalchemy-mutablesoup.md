---
layout: post
title: "SQLALchemy-MutableSoup"
date: 2020-01-17
categories: SQLAlchemy
permalink: sqlalchemy-mutablesoup
---

SQLAlchemy-MutableSoup defines a mutable [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) [SQLAlchemy](https://www.sqlalchemy.org/) database type.

## License

Publications which use this software should include the following citation:

Bowen, D.S. (2020). SQLAlchemy-MutableSoup \[Computer software\]. [https://dsbowen.github.io/sqlalchemy-mutablesoup](https://dsbowen.github.io/sqlalchemy-mutablesoup)

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/sqlalchemy-mutablesoup/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-mutablesoup
```

### Requirements

SQLAlchemy-MutableSoup requires a [SQLAlchemy](https://www.sqlalchemy.org) database.

### Setup

The following code will get you started with SQLAlchemy-MutableSoup as quickly as possible:

```python
# 1. Import from SQLAlchemy-MutableSoup
from sqlalchemy_mutablesoup import MutableSoupType

# 2. Standard session creation
from sqlalchemy import create_engine, Column, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, scoped_session

engine = create_engine('sqlite:///:memory:')
session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)
session = Session()
Base = declarative_base()

# 3. Create a model class with a `MutableSoupType` Column
class Model(Base):
    __tablename__ = 'model'
    id = Column(Integer, primary_key=True)
    soup = Column(MutableSoupType)

# 4. Create the database
Base.metadata.create_all(engine)

# 5. Additional setup
from bs4 import BeautifulSoup

model = Model()
session.add(model)
```

## Examples

Full examples can be found at [https://github.com/dsbowen/sqlalchemy-mutablesoup](https://github.com/dsbowen/sqlalchemy-mutablesoup/blob/master/examples.py).

### Example 1: Setting the soup from a string

`MutableSoupType` Columns can be set using a string. You can then manipulate them as if they were `BeautifulSoup` objects.

```python
model.soup = '<p>Hello World.</p>'
print(model.soup)
print(
    '`soup` is a BeautifulSoup object?', 
    isinstance(model.soup, BeautifulSoup)
)
```

Output:

```
<p>Hello World.</p>
`soup` is a BeautifulSoup object? True
```

### Example 2: Setting the soup from a BeautifulSoup object

`MutableSoupType` Columns can also be set to `BeaitifulSoup` objects.

```python
model.soup = BeautifulSoup('<p>Hello World.</p>', 'html.parser')
print(model.soup)
```

Output:

```
<p>Hello World.</p>
```

### Example 3: Setting an element

SQLAlchemy-MutableSoup contains a `set_element` convenience method. Then contents of a soup element, selected by a `parent_selector` (in css selector format), is set to a value, `val`.

As in the above examples, `val` can be a string or appropriate object from the bs4 package.

```python
model.soup.set_element(parent_selector='p', val='Hello Moon.')
session.commit()
print(model.soup)
```

Output:

```
<p>Hello Moon.</p>
```

### Example 4: Setting an element (advanced)

In this example, we want to set the content of a soup element which may or may not exist. Call this the 'target'. If the target exists, it would be a descendent of the 'parent' element. Otherwise, we want to generate a new target element and append it to the parent.

We can achieve this with the following:

```python
def gen_span_tag(*args, **kwargs):
    print('My args are:', args)
    print('My kwargs are:', kwargs)
    return BeautifulSoup('<span></span>', 'html.parser')

model.soup.set_element(
    parent_selector='p',
    val='Span text',
    target_selector='span',
    gen_target=gen_span_tag,
    args=['hello world'],
    kwargs={'hello': 'moon'}
)
session.commit()
print(model.soup)
```

Output:

```
My args are: ('hello world',)
My kwargs are: {'hello': 'moon'}
<p>Hello Moon.<span>Span text</span></p>
```

### Example 5: Using the changed method

The `set_element` method registers changes with the database automatically. Other changes must be registered with the database using the `changed` method. See [Mutation Tracking](https://docs.sqlalchemy.org/en/13/orm/extensions/mutable.html) for more details.

Without the `changed` method:

```python
model.soup.select_one('p')['style'] = 'color:red;'
session.commit()
print(model.soup)
```

Output:

```
<p>Hello Moon.<span>Span text</span></p>
```

With the `changed` method:

```python
model.soup.select_one('p')['style'] = 'color:red;'
model.soup.changed()
session.commit()
print(model.soup)
```

Output:

```python
<p style="color:red;">Hello Moon.<span>Span text</span></p>
```