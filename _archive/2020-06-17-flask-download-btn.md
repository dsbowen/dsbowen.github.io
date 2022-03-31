---
title: "Flask download button"
date: 2020-06-17
categories:
    - software
tags:
    - flask
---

Flask-Download-Btn defines a [SQLALchemy Mixin](https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html) for creating [Bootstrap](https://getbootstrap.com/) download buttons in a [Flask](https://palletsprojects.com/p/flask/) application.

Its features include:

1. **Automatic enabling and disabling.** A download button is automatically disabled on click and re-enabled on download completion.
2. **CSRF protection.** The download button checks for a CSRF authentication token to ensure the client has permission to download the requested file.
3. **Web form handling.** Download buttons are responsive to web forms.
4. **Pre-download operations.** Download buttons can easily perform operations before files are downloaded, making it easy to create temporary download files.
5. **Progress bar.** Update your clients on download progress with server sent events.

<a href="https://dsbowen.github.io/flask-download-btn/" target="_blank">Read the docs</a>.