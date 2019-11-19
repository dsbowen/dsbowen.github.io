---
layout: post
title: "SQLAlchemy-OrderingItem"
date: 2019-11-19 07:49:00
categories: SQLAlchemy
permalink: sqlalchemy-orderingitem
---

SQLAlchemy-OrderingItem provides an OrderingItem base for children of [orderinglist](https://docs.sqlalchemy.org/en/13/orm/extensions/orderinglist.html) relationships. Children of `orderinglist` relationships will exhibit more intuitive behavior when setting their parent attribute.

Readers can find the source code at [https://github.com/dsbowen/sqlalchemy-orderingitem](https://github.com/dsbowen/sqlalchemy-orderingitem).

## License

Publications which use this software should include the following citation for SQLAlchemy-OrderingItem:

Bowen, D.S. (2019). SQLAlchemy-OrderingItem \[Computer software\]. [https://dsbowen.github.io/sqlalchemy-orderingitem](https://dsbowen.github.io/sqlalchemy-orderingitem).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart):

```
$ pip install -U sqlalchemy-orderingitem
```

### Requirements

SQLAlchemy-OrderingItem requires a [SQLAlchemy](https://www.sqlalchemy.org) database.

### Setup

Set up your `orderinglist` relationship as usual. The only additional setup is giving the child of the `orderinglist` relationship the `OrderingItem` subclass.

```python
# 1. Import OrderingItem
from sqlalchemy_orderingitem import OrderingItem

# 2. Create session (standard)
from sqlalchemy import Column, ForeignKey, Integer, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.ext.orderinglist import ordering_list
from sqlalchemy.orm import sessionmaker, relationship

engine = create_engine('sqlite:///:memory:')
Session = sessionmaker(bind=engine)
session = Session()
Base = declarative_base()

class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)

    # 3. Declare orderinglist relationships
    children = relationship(
        'Child', 
        backref='parent',
        order_by='Child.index',
        collection_class=ordering_list('index')
    )
    orderingitem_children = relationship(
        'OrderingItemChild', 
        backref='parent',
        order_by='OrderingItemChild.index',
        collection_class=ordering_list('index')
    )

# This model is for demonstrating behavior without the OrderingItem subclass
class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    index = Column(Integer)


# 4. Subclass OrderingItem for children of orderinglist relationships
class OrderingItemChild(OrderingItem, Base):
    __tablename__ = 'ordering_item_child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    index = Column(Integer)


# 5. Create the database tables (standard)
Base.metadata.create_all(engine)
```

*Note*: `OrderingItem` expects that the zeroth `order_by` column is the column on which the `orderinglist` sorts items.

## Examples

Full examples can be found at [https://github.com.dsbowen/sqlalchemy-orderingitem](https://github.com.dsbowen/sqlalchemy-orderingitem/blob/master/examples.py).

We begin by creating instances of `Parent`, `Child`, and `OrderingItemChild` models and commit them to the session:

```python
p = Parent()
c = Child()
oic = OrderingItemChild()
session.add_all([p, c, oic])
session.commit()
```

### Example 1: Setting a child's parent attribute to a Parent instance

Ordinary behavior:

```python
c.parent = p
print(c.index)
```

Outputs:

```
None
```

With OrderingItem subclass:

```python
oic.parent = p
print(oic.index)
```

Outputs:

```
0
```

### Example 2: Setting a child's parent attribute to None

Ordnary behavior:

```python
p.children = [c]
c.parent = None
print(c.index)
```

Outputs:

```
0
```

With OrderingItem subclass:

```python
p.orderingitem_children = [oic]
oic.parent = None
print(oic.index)
```

Outputs:

```
None
```