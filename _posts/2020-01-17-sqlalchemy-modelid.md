---
layout: post
title: "SQLAlchemy-ModelId"
date: 2020-01-17
categories: SQLAlchemy
permalink: sqlalchemy-modelid
---

SQAlchemy-ModelId defines a base with a `model_id` property for [SQLAlchemy](https://www.sqlalchemy.org/) models.

The `model_id` property distinguishes model instances with the same identity from different tables.

## License

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/sqlalchemy-modelid/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-modelid
```

### Requirements

SQLAlchemy-ModelId requires a [SQLAlchemy](https://www.sqlalchemy.org) database.

### Setup

The following code will get you started with SQLAlchemy-ModelId as quickly as possible:

```python
# 1. Import ModelIdBase
from sqlalchemy_modelid import ModelIdBase

# 2. Standard session creation
from sqlalchemy import create_engine, Column, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, scoped_session

engine = create_engine('sqlite:///:memory:')
session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)
session = Session()
Base = declarative_base()

# 3. Subclass `ModelIdBase` to add a `model_id` property to any model
class Model(ModelIdBase, Base):
    __tablename__ = 'model'
    id = Column(Integer, primary_key=True)

# 4. Create the database
Base.metadata.create_all(engine)
```

## Example

Full setup and example can be found at [https://github.com/dsbowen/sqlalchemy-modelid](https://github.com/dsbowen/sqlalchemy-modelid/blob/master/example.py).

```python
my_model = Model()
session.add(my_model)
session.commit()
print(my_model.model_id)
```

Output:

```
model-1
```