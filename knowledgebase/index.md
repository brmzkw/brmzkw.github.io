---
layout: default
---

When you have to work on a stack that hasn't been designed by you, it might be
difficult to understand and fix bugs if you don't really know what each sub
component does.

For instance, last week, I've been asked why it was impossible to install one
of our Python packages using our local Pypi mirror: the package was not found
even though it was succesfully uploaded on the mirror.

The reason? The last version of pip (8.0.3) introduces a
[fix](https://github.com/pypa/pip/commit/8e236dd6a09bd2f70f9d4fc886da8c354d4c58f2)
to be [PEP 503](https://www.python.org/dev/peps/pep-0503/) compliant. The
package we wanted to install contains a dot in its name and our local Pypi
mirror, [Pyshop](https://github.com/mardiros/pyshop) wasn't canonicalizing the
package name correctly because it wasn't following the PEP. The temporary fix
was to use pip 8.0.2 until the upgrade of Pyshop.

To install the package, you only need to know to type `pip install <package
name>`. But when something goes wrong, you really want to know how pip works,
and Pyshop, and Pypi, and...

And you know what? It's always the same. You don't know why your SQL request
returns a bad result? Maybe because your SQLAlchemy relationship has been
misconfigured because... ... ...

Not knowing how to debug an issue or feeling lost are really frustrating
situations. To avoid them, knowing at least on surface what does each component
of your stack helps a lot to give you the direction where to search the
solution.

A problem with [Slumber](slumber)? Knowing it's a library on top of requests,
which is itself on top of urllib can really be helpful the day you have a
problem.


### Programming

* [Dependency injection](dependency_injection)


### Python

* [Slumber](slumber)
* setuptools
* virtualenv
* Flask
* SQLAlchemy
* mock
* voluptuous
* Sentry
* RabbitMQ
* Pypi/Pyshop/Devpi


### Javascript

* promises ($q, angular)


### Linux

* cgroups, netns
* LXC
* BGP
* Makefile


### Libraries

* DPDK


### Softwares

* Supervisord
* Elasticsearch
* Logstash
* Kibana
* DHCRelay
* Graphite
* Collectd
* Grafana
* Jenkins
* Shinken
* Docker
* PostgreSQL: ARRAY, JSON, strictness (compared to MySQL)


### Sysadmin

* SaltStack
