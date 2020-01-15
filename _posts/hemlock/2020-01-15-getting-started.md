---
layout: post
title: "Getting Started"
date: 2020-01-15
categories: Hemlock
permalink: hemlock/getting-started
---

Here are the prerequisites I recommend to take advantage of the full range of tools Hemlock offers. Unless otherwise specified, I recommend you download the latest stable version of the following:

**Essential**
1. *Python3 and pip3*. Python is Hemlock's primary language. pip allows you to install Python packages, including Hemlock itself. I recommend [Python3.6](https://www.python.org/downloads/release/python-366/), the version you will use for Heroku deployment (see below).
2. *Code editor*. I work in [Visual Studio (VS) Code](https://code.visualstudio.com/), but any code editor will do.

**Strongly recommended**
1. *Heroku and Heroku-CLI*. [Heroku](https://heroku.com/) is an inexpensive and accessible service for deploying web applications. The Hemlock command line interface builds on the [Heroku command line interface (CLI)](https://devcenter.heroku.com/articles/heroku-cli).
2. *Git*. [Git](https://git-scm.com/) is a version control system which I use to 'push' applications to Heroku. Relatedly, I recommend [Github](https://github.com/) for backing up Hemlock projects and sharing them with collaborators.

With the above software, you are ready to create, share, and deploy Hemlock projects.

The software below is encouraged for debugging, file storage, and Redis testing. These are less essential. If you're eager to get started with Hemlock, you can come back to these if and when you need them.

**Encouraged**
1. *Google Chrome and Chromedriver*. Hemlock's custom debugging tool requires [Google Chrome](https://www.google.com/chrome/) and [Chromedriver](https://chromedriver.chromium.org/downloads) to run locally.
2. *Google Cloud and Cloud SDK*. Hemlock easily integrates with [Google Cloud](https://cloud.google.com/) for storing statics (such as images to display during a survey) and user uploaded files. The Hemlock command line interface builds on [Cloud Software Development Kit (SDK)](https://cloud.google.com/sdk/).

**Advanced (Windows only)**
1. *Ubuntu on WSL*. Hemlock seamlessly interfaces with [Redis](https://redis.io) to run complex background processes during surveys. Because Windows cannot natively run Redis, I recommend [Ubuntu](https://ubuntu.com/) on [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Redis runs natively on Mac and Linux.

## Detailed instructions for Windows users

Overview:
1. Install Ubuntu and Windows Subsystem for Linux (WSL).
2. Install Python3 and pip3 on Ubuntu.
3. Use pip3 to install Hemlock-CLI
4. Use Hemlock-CLI to install other recommended software (Visual Studio Code, Google Chrome, etc.)

### 1. Ubuntu and WSL

Follow the [Microsoft documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to enable WSL and install the latest version of Ubuntu.

Open an Ubuntu terminal. You will be prompted to create a username and password.

### 2. Python3 and pip3

Ubuntu should include Python3.6. To verify, enter the following in your Ubuntu terminal window:

```
$ python3 --version
```

Expected output:

```
Python 3.6.x
```

Update your package lists with:

```
$ sudo apt-get update
```

Install pip3 with:

```
$ sudo apt-get install -f -y python3-pip
```

Finally, verify pip3 installation with:

```
$ pip3 --version
```

Expected output:

```
pip x.x.x from /usr/lib/python3/dist-packages (python 3.6)
```

### 3. Hemlock-CLI

pip install the Hemlock command line interface (CLI) with:

```
$ pip3 install hemlock-cli
```

Verify `hemlock-cli` installation with:

```
$ hlk --version
```

Expected output:

```
hemlock-cli 0.0.x
```

### 4. Recommended software

Install other recommended software with `hlk install [options]`. The options specify which recommended software tools you want to download. For example, to install Visual Studio Code, Heroku-CLI, Git, Google Chrome and Chromedriver, and Cloud SDK, run:

```
$ hlk install --vscode --heroku-cli --git --chrome --cloud-sdk
```

Before installing Heroku-CLI, Git, or Cloud SDK, make sure you have Heroku, Github, and Google Cloud accounts, respectively.

Verify the installations with:

```
$ code --version
$ heroku --version
$ git --version
$ google-chrome --version
$ chromedriver --version
$ gcloud --version
```

## Detailed instructions for Mac and Linux in progress