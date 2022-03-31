---
title: "Fast autoML"
date: 2021-02-06
categories:
    - software
tags:
    - machine learning
---

Most autoML packages aim for exceptional performance but need to train for an exceptional amount of time. Fast-autoML aims for reasonable performance in a reasonable amount of time. In my work, I find this is a great tool for fast iteration in initial stages of data exploration.

Anecdotally, I've also found that fast-autoML performs as well as or better than autoML packages (auto-sklearn, TPOT, MLBox) for many smaller datasets that I work with.

Fast-autoML includes additional utilities, such as tools for comparing model performance by repeated cross-validation.

<a href="https://dsbowen.github.io/fast-automl/" target="_blank">Read the docs</a>.