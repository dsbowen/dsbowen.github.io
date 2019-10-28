---
layout: post
title: "SQLAlchemy-Function"
date: 2019-10-27 22:42:00
categories: SQLAlchemy
permalink: sqlalchemy-function
---

SQLAlchemy-Function defines a [SQLALchemy Mixin](https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html) for creating `Function` models.

A `Function` model has a parent (optional), a function, arguments, and keyword arguments. When called, the `Function` model executes its function, passing in its parent (if applicable), its arguments, and its keyword arguments.

## License

Publications which use this software should include the following citation for SQLAlchemy-Function and its dependency, [SQLAlchemy-Mutable](https://pypi.org/project/sqlalchemy-mutable/):

Bowen, D.S. (2019). SQLAlchemy-Function \[Computer software\]. [https://github.com/dsbowen/sqlalchemy-functionv](https://github.com/dsbowen/sqlalchemy-function)

Bowen, D.S. (2019). SQLAlchemy-Mutable \[Computer software\]. [https://github.com/dsbowen/sqlalchemy-mutable](https://github.com/dsbowen/sqlalchemy-mutable)

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/sqlalchemy-function/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-function
```

### Requirements

SQLALchemy-Function requires a [SQLAlchemy](https://www.sqlalchemy.org) database.

### Setup

The following code will get you started with SQLAlchemy-Function as quickly as possible:

```python
# 1. import FunctionMixin and FunctionBase
from sqlalchemy_function import FunctionMixin

# 2. Standard session creation
from sqlalchemy import create_engine, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship, sessionmaker, scoped_session
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.ext.orderinglist import ordering_list

engine = create_engine('sqlite:///:memory:')
session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)
session = Session()
Base = declarative_base()

# 3. Create a Function model with the FunctionMixin
class Function(FunctionMixin, Base):
    __tablename__ = 'function'
    id = Column(Integer, primary_key=True)

# 4. Create the database
Base.metadata.create_all(engine)
```

## Examples

Full examples can be found at [https://github.com/dsbowen/sqlalchemy-function](https://github.com/dsbowen/sqlalchemy-function/blob/master/examples.py).

### Example 1: Basic use

This example defines a function, `foo`, and creates an instance of a `Function` model, `f`. When called, `f` executes `foo` with its arguments and keyword arguments.

```python
def foo(*args, **kwargs):
    print('My arguments are:', args)
    print('My keyword arguments are:', kwargs)
    return 'hello world'

f = Function(func=foo, args=['hello moon'], kwargs={'hello': 'star'})
session.add(f)
session.commit()
print(f())
```

Output:

```
My arguments are: ('hello moon',)
My keyword arguments are: {'hello': 'star'}
hello world
```

### Additional setup for Exampels 2-4: Function parents

Examples 2-4 illustrate how to `Function` parents. When called, an instance of a `Function` model with a `'parent'` attribute will call its function, passing in its parent as the first argument (followed by its arguments and keyword arguments).

A parent inherits the `FunctionBase` mixin. It must call its `_set_function_relationships` method before setting any `Function` attributes.

The `FunctionMixin` also defines an `index` column, which can be used to order `Function` models in an `ordering_list`.

```python
class Child(FunctionMixin, Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))

# 1. Define a Parent model with the FunctionBase mixin
class Parent(FunctionBase, Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)

    # 2. Fuction models must reference their parent with a 'parent' attribute
    functions = relationship(
        'Child',
        backref='parent',
        order_by='Child.index',
        collection_class=ordering_list('index')
    )

    # 3. Call _set_function_relationships before setting function attributes
    def __init__(self):
        self._set_function_relationships()

Base.metadata.create_all(engine)
```

### Example 2: Function models with parents (basic use)

This example defines a `Function` model instance and its parent.

```python
def foo(parent, *args, **kwargs):
    print('My parent is:', parent)
    print('My arguments are:', args)
    print('My keyword arguments are:', kwargs)
    return 'hello world'

p = Parent()
session.add(p)
session.commit()
f = Function(
    parent=p, func=foo, args=['hello moon'], kwargs={'hello': 'star'}
)
print(f())
```

Output:

```
My parent is: <__main__.Parent object at 0x7fa5c2cf2588>
My arguments are: ('hello moon',)
My keyword arguments are: {'hello': 'star'}
hello world
```

### Examples 3-4: Automatic conversion of functions to Function models

When setting a `Function` attribute, parents automatically convert functions to `Function` models.

For relationships which do not use a list, the following commands are equivalent:

```python
model.function = foo
model.function = Function(parent=model, func=foo)
```

For relationships which use a list, the following commands are equivalent:

```python
model.functions = foo
model.functions = [foo]
model.functions = Function(parent=model, func=foo)
model.functions = [Function(parent=model, func=foo)]
```

We can see these conversions in the following examples:

```python
p.functions.clear()
p.functions = foo
print(p.functions)
print(p.functions[0]())
```

Output:

```
[<__main__.Child object at 0x7fa5c2d15898>]
My parent is: <__main__.Parent object at 0x7fa5c2cf2588>
My arguments are: ()
My keyword arguments are: {}
hello world
```

```python
def bar(parent):
    return 'goodbye world'

p.functions.clear()
p.functions = [foo, bar]
print(p.functions)
[print(f()) for f in p.functions]
```

Output:

```
[<__main__.Child object at 0x7fa5c2cb1198>, <__main__.Child object at 0x7fa5c2cb1e10>]
My parent is: <__main__.Parent object at 0x7fa5c2cf2588>
My arguments are: ()
My keyword arguments are: {}
hello world
goodbye world
```