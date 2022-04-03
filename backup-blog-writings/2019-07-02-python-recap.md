---
layout: post
title:  "[Dev] Brief Recap of Python"
date:   2019-07-02
tags: [Dev]
---

In python, there are, largely speaking, two versions, 2 and 3. Unlike most programming languages, there are no compatibility guaranteed. In other words, if you write a code in Python 3 and try to run in Python 2 environment, it may not work.

Here I am using Python 3.

### Installation

There are a number of ways of doing this. You can install in command line, download a executable file, and so on. I would recommend using Anaconda. Anaconda is an open-source python distribution that is focused on the data science package. It comes with Python and a number of data science libraries like NumPy, pandas, Matplotlib, TensorFlow, etc. Visit the website to install it.

* https://www.anaconda.com/distribution/

After installing it, open up a terminal session (or command prompt in Windows system) and type `python --version` to make sure you have installed the right version.

```
$ python --version
Python 3.7.3
```

### Class in Python 3

Just like many other programming languages, a Python 3 class contains a constructor, a destructor, methods, and member variables. Let's see an example.

```python
class MyClass:
    def __init__(self, arg1, arg2, ...):        # Constructor
        ...
    def __del__(self):                          # Destructor
        ...

    def firstMethod(self, arg1, arg2, ...):     # Method 1
        ...
    def secondMethod(self, arg1, arg2, ...):    # Method 2
        ...
```
Note that in the constructor part, additional arguments are not necessary.
```python
def __init__(self):
  ...
```
would suffice for most cases.
Notice also that there are no braces of any kind that wraps up the scope (or namespace). Rather, it is a handy feature of python3 that it uses spaces in separating one scope from another.
