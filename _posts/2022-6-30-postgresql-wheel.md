---
layout: post
comments: true
title:  "Postgresql wheels and pytest with Django"
date:   2022-06-30 00:00:00 +0000
categories: Python, pytest, Django, Postgresql
draft: true
---

Being able to run unit test cases with pytest along with a database (Postgres in this case), often times needs choices to be made on whether the DB queries need to be mocked or letting the queries run against a test database of sorts.

Setting up a database while also making sure that the right versions are altogether and the data is as needed is sometimes a balancing act. In order to try and mitigate some of these issues, I decided to try out the [postgresql-wheel](https://github.com/michelp/postgresql-wheel) package. The package comes with a "bundled" version of postgresql that suffices to run most Django ORM queries. There are similar projects like [pytest-postgresql](https://github.com/ClearcodeHQ/pytest-postgresql) that also let you get test cases working with pytest but they need a previous install of Postgresql (or atleast all of the postgres dependencies like libpq, pg_config, etc). This is not required with postgresql-wheel since it comes with batteries included.

**Prerequisites (there are a few tweaks that can be made to get it running on MacOS)**
- Python 3.7 to 3.9
- The current `postgresql-wheel` packages in pypi will run on most Linux systems
- The packages are versioned such that verion x.y.z implies that Postgres version x.y comes bundled. (for the moment, there is an issue with version above 14.1.0). So for now this blog uses 14.1.0. (As a FYI - it's not that hard to have your own build with say a later version of Postgres. You can get a copy of the artifact/package via the output from Github actions from possibly your own fork of the repo. However, you won't be able to get it in pypi)

**Sample requirements**

```bash
Django==4.0.5
postgresql-wheel==14.1.0
psycopg2-binary==2.9.3
pytest==7.1.2
pytest-django==4.5.2
```

**settings.py (these will be overriden)**

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'pollsdb',
        'USER': '',
        'PASSWORD': '',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
```

**conftest.py**

```python
from pathlib import Path
from tempfile import TemporaryDirectory

import pytest
from django.db import connections
from postgresql import initdb, pg_ctl, psql
import shutil

import psycopg2
from psycopg2.extensions import ISOLATION_LEVEL_AUTOCOMMIT


def run_sql(sql, user, host, port):
    conn = psycopg2.connect(database='postgres', user=user, host=host, port=str(port))
    conn.set_isolation_level(ISOLATION_LEVEL_AUTOCOMMIT)
    cur = conn.cursor()
    cur.execute(sql)
    conn.close()


def run_psql(sql_file, user, host, port):
    print('running psql.....')
    cmd_str = psql(f'-d pollsdb -U {user} -h {host} -p {port} -a -f {sql_file}')
    print(cmd_str)


@pytest.fixture(scope='session')
def django_db_setup():
    log = "testdatabase.log"
    port = 5678

    pgdata = TemporaryDirectory()
    host = pgdata.name
    user = "postgres"

    log = Path(log)
    try:
        log.unlink()
    except FileNotFoundError:
        pass

    initdb(f'-D {pgdata.name} --auth=trust --no-sync -U postgres')
    pg_ctl(f'-D {pgdata.name} -o "-p {port} -k {pgdata.name} -h \\"\\"" -l {log} start')

    from django.conf import settings

    settings.DATABASES['default']['HOST'] = host
    settings.DATABASES['default']['PORT'] = str(port)
    settings.DATABASES['default']['USER'] = user

    run_sql('DROP DATABASE IF EXISTS pollsdb', user=user, host=host, port=port)
    run_sql('CREATE DATABASE pollsdb', user=user, host=host, port=port)
    sql_file = '/path/to/pollsddl.sql'
    run_psql(sql_file=sql_file, user=user, host=host, port=port)

    yield

    for connection in connections.all():
        connection.close()

    run_sql('DROP DATABASE pollsdb', user=user, host=host, port=port)
    pg_ctl(f'-D {pgdata.name} stop')
    try:
        print(f'\n Clean-up of {pgdata.name}....')
        shutil.rmtree(pgdata.name)
    except OSError as e:
        print(f'{e.filename} - {e.strerror}')
```

**Sample test case**

```python
import pytest
from polls.models import Question


@pytest.mark.django_db
def test_questions():
    assert Question.objects.count() == 0
```

A couple of gotchas -
- If you are trying to restore from a previous pg_dump backup, use the `run_psql` variant rather than `run_sql`. As mentioned here [how to execute non sql commands in psycopg2](https://stackoverflow.com/questions/53077083/how-to-execute-non-sql-commands-in-psycopg2), you would need to be wary of psql specific commands
