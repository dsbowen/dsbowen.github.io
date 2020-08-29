---
title: "Flask worker"
date: 2020-06-15
categories:
    - software
tags:
    - flask
---

Flask-Worker simplifies interaction with a [Redis Queue](https://redis.io/) for executing long-running tasks in a [Flask](https://flask.palletsprojects.com/en/1.1.x/) application. 

Long-running tasks are managed by a Worker, who sends the client a loading page until it completes the task. Upon completing the task, the Worker automatically replaces the client's window with the loaded page.

<a href="https://dsbowen.github.io/flask-worker/" target="_blank">Read the docs</a>.