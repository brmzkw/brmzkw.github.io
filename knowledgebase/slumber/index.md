---
layout: default
---

[Slumber](https://github.com/samgiles/slumber) is a Python module built on top
of [requests](http://docs.python-requests.org/en/master/) that helps to consume
RESTful APIs.


First, you need to create an API object:

```python
from slumber import API

api = API(base_url='https://api.github.com', append_slash=False)
```

`append_slash=False` is required because Github API refuses requests ending
with a slash, and Slumber adds one by default.

Then, you need to construct your query:

```python
# Equivalent to GET https://api.github.com/users/brmzkw/repos
api.users.brmzkw.repos.get()
```

Slumber automatically unserializes the HTTP response.


Slumber can also help to generate dynamically a request without having to build
the URL manually, through the use of getattr. For example:

```python

def get_repositories(api, name):
    return getattr(api.users, name).repos


api = API(base_url='https://api.github.com', append_slash=False)
print get_repositories(api, 'brmzkw').get()
```
