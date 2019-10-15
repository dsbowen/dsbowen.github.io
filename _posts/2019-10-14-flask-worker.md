---
layout: post
title:  "Flask-Worker"
date:   2019-10-14 09:09:41 -0400
categories: Flask
permalink: flask-worker
---

Flask-Worker simplifies interaction with a Redis Queue for executing long-running tasks in a Flask application. 

Long-running tasks are managed by a Worker, who sends the client a loading page until it completes the task. Upon completing the task, the Worker automatically replaces the client's window with the loaded page.

## License

Publications which use this software should include the following citations for Flask-Worker and its dependency, [SQLAlchemy-Mutable](https://pypi.org/project/sqlalchemy-mutable/).

Bowen, D.S. (2019). Flask-Worker [Compluter software]. [https://github.com/dsbowen/flask-worker](https://github.com/dsbowen/flask-worker)

Bowen, D.S. (2019). SQLAlchemy-Mutable [Computer software]. [https://github.com/dsbowen/sqlalchemy-mutable](https://github.com/dsbowen/sqlalchemy-mutable)

This project is licensed under the MIT License [LICENSE](https://github.com/dsbowen/flask-worker/blob/master/LICENSE).

## Getting started

### Installation

Install and update using [pip](https://pip.pypa.io/en/stable/quickstart/):

```bash
$ pip install -U flask-worker
```

### Requirements

Flask-Worker is compatible with [Flask](https://palletsprojects.com/p/flask/) applications. Applications must have a [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) database and a [Flask-SocketIO](https://flask-socketio.readthedocs.io/en/latest/) websocket, and interact with a [Redis Queue](https://python-rq.org/).

### Setup

The following will get you started with Flask-Worker as quickly as possible:

Our initial file structure will be:

```
app.py
factory.py
models.py
static/
  worker_loading.gif
```

In `factory.py`:

```python
# 1. Import the Manager
from flask_worker import Manager

from flask import Flask
from flask_socketio import SocketIO
from flask_sqlalchemy import SQLAlchemy
from redis import Redis
from rq import Queue
import eventlet
import os

db = SQLAlchemy()
eventlet.monkey_patch(socket=True)
socketio = SocketIO(asynch_mode='eventlet')
# 2. Initialize a Manager with the database and socketio
manager = Manager(db=db, socketio=socketio)

def create_app():
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = (
        'sqlite:///'+os.path.join(os.getcwd(), 'data.db')
    )
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    app.config['EXPLAIN_TEMPLATE_LOADING'] = True
    app.redis = Redis.from_url('redis://')
    app.task_queue = Queue('my-task-queue', connection=app.redis)
    db.init_app(app)
    socketio.init_app(app, message_queue='redis://')
    # 3. Initialize the manager with the application
    manager.init_app(app)
    return app
```

In `models.py`:

```python
from factory import db

# 1. Import the worker mixin
from flask_worker import WorkerMixin

import time


class Employer(db.Model):
    """Employer model"""
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)

    # 2. Add a worker to the employer
    # The worker must reference its employer by the attribute name 'employer'
    worker = db.relationship('Worker', uselist=False, backref='employer')

    def __init__(self, name):
        self.name = name
        # 3. Instantiate a worker
        self.worker = Worker(
            method_name='complex_task', kwargs={'seconds': 5}
        )

    def complex_task(self, seconds):
        print('Complex task started')
        for i in range(seconds):
            print('Progress: {}%'.format(100.0*i/seconds))
            time.sleep(1)
        print('Progress: 100.0%')
        print('Complex task finished')


# 4. Create a Worker model with the worker mixin
class Worker(WorkerMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    employer_id = db.Column(db.Integer, db.ForeignKey('employer.id'))
```

In `app.py`

```python
from factory import create_app, db, socketio
from models import Employer

app = create_app()

@app.before_first_request
def before_first_request():
    db.create_all()

# VIEW FUNCTIONS GO HERE

if __name__ == '__main__':
    socketio.run(app)
```

Finally, add a `worker_loading.gif` to your static folder.

### Running the application

To run the application, we will need to run a Redis Queue. In this example, the queue is named 'my-task-queue' (see `factory.py`). Run the Redis Queue with:

```bash
$ rq worker my-task-queue
```

In a separate terminal, run the app with:

```bash
$ python app.py
```

Your app will be running on `http://localhost:5000`. Note that this will only work once we have defined our view functions.

**Note**: Flask-Worker looks for the application instance in `app.app`. You must notify the manager if you choose a different application location or name. Include the following in `factory.py`:

```python
manager.app_import = 'path.to.app.file.app_file.app_instance'
```

## Examples

See the example application [here](https://github.com/dsbowen/flask-worker).

### Convenience method

Before getting started, I define a `get_model` convenience method in `models.py`:

```python
def get_model(model_class, name):
    """Convenience method for database querying"""
    model = model_class.query.filter_by(name=name).first()
    if not model:
        model = model_class(name=name)
        db.session.add(model)
        db.session.flush([model])
    return model
```

This function returns a model of the type `model_class` with the specified `name`. If this model does not yet exist, this function creates it.

Be sure to import this convenience method in `app.py`:

```python
from models import get_model
```

### Example 1: Basic use

Example 1 (no worker) illustrates the basic problem Flask-Worker solves. A model (the employer) must run a long, complex task before the next page loads.

What we want is for the complex task to run once, and for the view function to return a loading page while the complex task is running. While the task is running, additional requests to this route should not cause the complex task to run multiple times.

However, in the following view function, there is no loading page, and every request to the route queues up another run of the complex task.

```python
@app.route('/example1-no-worker')
def example1_no_worker():
    print('Request for /example1-no-worker')
    employer = get_model(Employer, 'employer1-no-worker')
    employer.complex_task(seconds=5)
    return 'Example 1 (no worker) finished'
```

Example 1 illustrates how Flask-Worker solves this problem. Here, the employer uses its worker to execute its complex task.

```python
@app.route('/example1')
def example1():
    print('Request for /example1')
    employer = get_model(Employer, 'employer1')
    worker = employer.worker
    if not worker.job_finished:
        return worker()
    return 'Example 1 finished'
```

The worker only sends the employer's complex task to the Redis Queue once, regardless of how many times the client requests this route. Until the worker finishes its job, it returns a loading page. Once the worker finishes its job, it automatically replaces the loading page with a call to `/example1`. This new call sees that the worker has finished its job, and returns `'Example 1 finished'`.

The result is cached once the worker finishes its job. Future requests from the client will not cause the worker to queue up additional jobs.

### Example 2: Basic use without caching

You can avoid caching with the worker's `reset` method.

In this example, the worker is reset once it finishes its job. Unlike in Example 1, future requests from the client *will* cause the worker to queue up additional jobs.

```python
@app.route('/example2')
def example2():
    print('Request for /example2')
    employer = get_model(Employer, 'employer2')
    worker = employer.worker
    if not worker.job_finished:
        return worker()
    worker.reset()
    db.session.commit()
    return 'Example 2 finished'
```

### Example 3: Callback functions

Example 3 demonstrates the worker's callback function. Once the worker has finished its job, it redirects the client to the callback route.

In this example, the callback function is defined so that requests to the callback route will display the loading page until the worker finishes its job.

```python
@app.route('/example3')
def example3():
    print('Request for /example3')
    employer = get_model(Employer, 'employer3')
    worker = employer.worker
    worker.callback = 'callback_route'
    return worker()

@app.route('/callback_route')
def callback_route():
    print('Request for /callback_route')
    employer = get_model(Employer, 'employer3')
    worker = employer.worker
    if not worker.job_finished:
        return worker()
    worker.reset()
    db.session.commit()
    return 'Example 3 finished'
```

### Example 4: Routing

Flask-Worker also defines a `RouterMixin`. 

Example 4 (no routing) illustrates the basic problem that *routers* solve. A request initiates a series of function calls, among which is the employer's complex task.

What we want is to 'pause' the series of function calls when the complex task is executed. The view function should return a loading page while the complex task is running. Once the worker finishes its complex task, the view function should pick up where it left off with respect to the series of function calls.

However, in the following example, every request reruns the entire series of function calls.

```python
@app.route('/example4-no-router')
def example4_no_router():
    print('Request for /example4-no-router')
    return func1('hello world')

def func1(hello_world):
    print(hello_world)
    return func2('hello moon')

def func2(hello_moon):
    print(hello_moon)
    employer = get_model(Employer, 'employer4-no-router')
    employer.complex_task(seconds=5)
    return func3('hello star')

def func3(hello_star):
    print(hello_star)
    db.session.commit()
    return 'Example 4 (no router) finished'
```

Example 4 illustrates how this problem can be solved using router models defined using Flask-Worker's `RouterMixin`.

First, we define a router model in `router_models.py`.

```python
from factory import db
from models import Employer, get_model

# 1. Import the router mixin and set_route decorator
from flask_worker import RouterMixin, set_route

# 2. Create a Router class with the router mixin.
class Router4(RouterMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)

    def __init__(self, name):
        self.name = name
        # 3. Set `current_route` on initialization
        self.current_route = 'func1'
        self.args = ['hello world']
        super().__init__()

    def func1(self, hello_world):
        print(hello_world)
        return self.func2('hello moon')

    # 4. 'Bookmark' functions with the @set_route decorator
    @set_route
    def func2(self, hello_moon):
        print(hello_moon)
        employer = get_model(Employer, 'employer4')
        worker = employer.worker
        # 5. Run the Employer's complex task with a Worker
        return self.run_worker(
            worker=worker, next_route=self.func3, args=['hello star']
        )

    @set_route
    def func3(self, hello_star):
        print(hello_star)
        return 'Example 4 finished'
```

Second, we add the following to `app.py`:

```python
from router_models import Router4

@app.route('/example4')
def example4():
    print('Request for /example4')
    router = get_model(Router4, 'router4')
    return router.route()
```

Note that we set the router's `current_route` and `args` in `__init__`. `current_route` is the name of the method called by `router.route()`. The router passes its `args` and `kwargs` to this method.

We use the `@set_route` decorator for functions 2 and 3. This decorator creates a 'bookmark' for the current function call. Specifically, it sets the router's `current_route` to the name of the current function and stores the args and kwargs. The next time `router.route()` is called, it will start at the bookmark.

Finally, the router's `run_worker` method manages workers. Until the worker finishes its job, `run_worker` returns a loading page. When finished, it calls the `next_route` with the specified `args` and `kwargs`.

Run this example and navigate to `http://localhost:5000/example4`. As the complex task runs in the background, hit the refresh button a few times. You should see the following output in your terminal:

```
Request for /example4
hello world
hello moon
Request for /example4
hello moon
Request for /example4
hello moon
...
hello star
```

The result is cached once the worker finishes its job. If you hit refresh again, you should see:

```
Request for /example4
hello star
```

### Example 5: Routing without caching

You can avoid caching by defining the router's `reset` method. This method should reset the `current_route`, `args`, and `kwargs`, as well as any workers that were called.

```python
class Router5(RouterMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)

    def __init__(self, name):
        self.name = name
        self.reset()
        super().__init__()

    def reset(self):
        self.current_route = 'func1'
        self.args = ['hello world']
        self.kwargs = {}
        employer = get_model(Employer, 'employer5')
        employer.worker.reset()

    def func1(self, hello_world):
        print(hello_world)
        return self.func2('hello moon')

    @set_route
    def func2(self, hello_moon):
        print(hello_moon)
        employer = get_model(Employer, 'employer5')
        worker = employer.worker
        return self.run_worker(
            worker=worker, next_route=self.func3, args=['hello star']
        )

    def func3(self, hello_star):
        print(hello_star)
        # Reset the Router
        self.reset()
        return 'Example 5 finished'
```

The view function is essentially the same as in Example 4:

```python
from router_models import Router5

@app.route('/example5')
def example5():
    print('Request for /example5')
    router = get_model(Router5, 'router5')
    return router.route()
```

Because we reset the router once the series of function calls has completed, future requests will redirect to `func1`, and will cause the worker to queue up another job.

## Customization and defaults

Change a worker's loading image with:

```python
my_worker.loading_img = 'my-loading-img.gif'
```

Change the default loading image for all workers using the manager:

```python
my_manager.loading_img = 'my-loading-img.gif'
```

Change a worker's loading template with:

```python
my_worker.template = 'my-template.html'
```

Change the default loading template for all workers using the manager:

```python
my_manager.template = 'my-template.html'
```

Loading templates take their worker as a `worker` argument. They should include the worker's script in their head:

```html
<head>
  {% raw %}{{ worker.script() }}{% endraw %}
</head>
```