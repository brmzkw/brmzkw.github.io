---
layout: default
---

From the official [documentation] [virtualenv doc]:

> A Virtual Environment is a tool to keep the dependencies required by
> different projects in separate places, by creating virtual Python
> environments for them.  It solves the “Project X depends on version 1.x but,
> Project Y needs 4.x” dilemma, and keeps your global site-packages directory
> clean and manageable.


Create the virtualenv:

```
mkdir -p ~/virtualenvs
virtualenv ~/virtualenvs/my-super-api
```

Enter in the virtualenv:

```
source ~/virtualenvs/my-super-api/bin/activate
```

From there, the `python` binary you are using is actually located at
`~/virtualenvs/my-super-api/bin/python`.


There are two directories you need to look into:

* `<venv>/bin` is the directory where the `activate` script and setuptools
  entrypoints are located into.
* `<venv>/lib/python2.7/site-packages/` is the directory where your modules are
  installed. If you need to look at (or debug) the code of the module you're
  using, search for them in this path.


[virtualenv doc]: http://docs.python-guide.org/en/latest/dev/virtualenvs/


### BEWARE

When you enter in a virtualenv with `source <venv>/bin/activate`, your `$PATH`
is prepended with `<venv>/bin`.
If for any reason your virtualenv is unreachable (because the folder has been
deleted for instance), the Python version used will be the next one your shell
will succeed to load from the `$PATH`, ie. the Python of your **system**.

If you're in a strange case where you are sure to be in a virtualenv but you
can't import some modules, double check you're really using the correct Python:
`which python` should return the Python executable from the environment.


### Final note

You should (almost) never install something system wide, and instead always use
virtualenvs.
