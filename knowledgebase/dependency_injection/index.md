---
layout: default
---

Let's say you want to create a function to get the users from your database
having a Gmail account.

Of course, you don't want to put credentials in your code but rather in a
.ini configuration file, parsed with the standard module ConfigParser.


### Setup

The following script creates the database `example.sqlite3` which
contains some users, and the configuration file `config.cfg` used later.

This script should be run prior to the following examples, and requires
sqlalchemy to be installed.

```python
import sys

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker


database = 'sqlite:///example.sqlite3'


# Create config file.
open('config.cfg', 'w+').write('''[default]

database = %s
''' % (database,))


engine = create_engine(database)
Base = declarative_base(bind=engine)


class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    email = Column(String, nullable=False)


# Create example.sqlite3.
Base.metadata.create_all()

# Create the SQLALchemy Session.
Session = sessionmaker(bind=engine)
session = Session()

# Create some users.
for tld in ('yahoo.com', 'gmail.com', 'hotmail.com', 'opensmtpd.org'):
    for username in ('Alice', 'Bob', 'Jack', 'John', 'Zoey'):
        email = username.lower() + '@' + tld
        user = User(email=email)
        session.add(user)

# Save the database.
session.commit()

```


### Problem solving

Let's write the first version of our function:

```python
import ConfigParser

from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import sessionmaker


def get_gmail_users():

    # Parse configuration string
    config = ConfigParser.RawConfigParser()
    config.read('config.cfg')
    connstr = config.get('default', 'database')

    # Create SQLAlchemy engine.
    engine = create_engine(connstr)

    # Introspect the database to feed a SQLAlchemy metadata object. This way,
    # metadata contains the list of tables/columns of the database.
    metadata = MetaData(bind=engine, reflect=True)

    # Create a SQLAlchemy session.
    Session = sessionmaker(engine)
    session = Session()

    # Build the SQL query: SELECT * FROM users WHERE email LIKE '%gmail.com'.
    users_table = metadata.tables['users']
    query = session.query(users_table).filter(
        users_table.c.email.endswith('gmail.com')
    )

    # Execute query.
    return query.all()


print get_gmail_users()
```

As expected, it returns the list of Gmail users. Even if this code is quite
simple and does what was expected, it is so thightly coupled that it is really
hard to imagine adding more features to it.

For instance, let's allow the user to specify the configuration file to parse.


### Configuration file customization


```python
import os

import ConfigParser

from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import sessionmaker


def get_gmail_users():

    # Get the configuration file to use from environment, defaulting to
    # "config.cfg".
    config_filename = os.getenv('CONFIG_FILE', 'config.cfg')

    # Parse configuration string
    config = ConfigParser.RawConfigParser()
    config.read(config_filename)
    connstr = config.get('default', 'database')

    # Create SQLAlchemy engine.
    engine = create_engine(connstr)

    # Introspect the database to feed a SQLAlchemy metadata object. This way,
    # metadata contains the list of tables/columns of the database.
    metadata = MetaData(bind=engine, reflect=True)

    # Create a SQLAlchemy session.
    Session = sessionmaker(engine)
    session = Session()

    # Build the SQL query: SELECT * FROM users WHERE email LIKE '%gmail.com'.
    users_table = metadata.tables['users']
    query = session.query(users_table).filter(
        users_table.c.email.endswith('gmail.com')
    )

    # Execute query.
    return query.all()


print get_gmail_users()
```

Here, we got the configuration filename from the environment.

In the case you want to parse a .yaml file instead of the .ini, you would have
to completely change the configuration parsing part of your function.

Here's come dependency injection. Your function needs to query the database. It
doesn't need to know how to parse a configuration file. Instead, you can just
give the configuration filename as argument.

In other words, "dependency injection" are just scary words meaning "pass your
requirements in arguments".

Let's improve the code:

```python
import os

import ConfigParser

from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import sessionmaker


def get_gmail_users(connstr):

    # Create SQLAlchemy engine.
    engine = create_engine(connstr)

    # Introspect the database to feed a SQLAlchemy metadata object. This way,
    # metadata contains the list of tables/columns of the database.
    metadata = MetaData(bind=engine, reflect=True)

    # Create a SQLAlchemy session.
    Session = sessionmaker(engine)
    session = Session()

    # Build the SQL query: SELECT * FROM users WHERE email LIKE '%gmail.com'.
    users_table = metadata.tables['users']
    query = session.query(users_table).filter(
        users_table.c.email.endswith('gmail.com')
    )

    # Execute query.
    return query.all()


def get_config():
    config_filename = os.getenv('CONFIG_FILE', 'config.cfg')
    config = ConfigParser.RawConfigParser()
    config.read(config_filename)
    return config


config = get_config()
print get_gmail_users(config.get('default', 'database'))
```

With this version, we only have to update `get_config` to change parse a yaml
file instead of a ini file. Simple as that. Our whole application doesn't need
to know where its configuration comes from.


### Our function still does too much

`get_gmail_users` still does too much. It connects to the database, introspects
tables and creates the SQLAlchemy session. The only thing we need to query
Gmail users is to have the SQLAlchemy session, nothing more.


```python
import os

import ConfigParser

from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import sessionmaker


def get_gmail_users(metadata, session):
    users_table = metadata.tables['users']
    query = session.query(users_table).filter(
        users_table.c.email.endswith('gmail.com')
    )
    return query.all()


def get_config():
    config_filename = os.getenv('CONFIG_FILE', 'config.cfg')
    config = ConfigParser.RawConfigParser()
    config.read(config_filename)
    return config


def get_sessionmaker(connstr):
    engine = create_engine(connstr)
    metadata = MetaData(bind=engine, reflect=True)
    return metadata, sessionmaker(engine)


config = get_config()
metadata, Session = get_sessionmaker(config.get('default', 'database'))
session = Session()
print get_gmail_users(metadata, session)
```

With this slight refactoring, our code becomes much easier to update and
unittest.


### Let's get Yahoo! users

To make our code even more generic, we can pass the TLD in arguments. And to
prevent breaking the API, we can even still set gmail.com as the default!

```python
import os

import ConfigParser

from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import sessionmaker


def get_users_by_tld(metadata, session, tld='gmail.com'):
    users_table = metadata.tables['users']
    query = session.query(users_table).filter(
        users_table.c.email.endswith(tld)
    )
    return query.all()


def get_config():
    config_filename = os.getenv('CONFIG_FILE', 'config.cfg')
    config = ConfigParser.RawConfigParser()
    config.read(config_filename)
    return config


def get_sessionmaker(connstr):
    engine = create_engine(connstr)
    metadata = MetaData(bind=engine, reflect=True)
    return metadata, sessionmaker(engine)


config = get_config()
metadata, Session = get_sessionmaker(config.get('default', 'database'))
session = Session()
print get_users_by_tld(metadata, session, 'yahoo.com')
```


### Conclusion

You get the idea. Your functions ideally always do one thing, and one thing
right. With such approach, it's far easier to refactor and add features.

A good way to ensure you're on the right path is to document your functions. If
you can't express easily what your function does, it's probably too much
complicated and you can split it into easy pieces.
