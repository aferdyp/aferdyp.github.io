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
- The current packages in pypi will run on most Linux systems
- The packages are versioned such that verion x.y.z implies that Postgres version x.y comes bundled. (for the moment, there is an issue with version above 14.1.0). So for now this blog uses 14.1.0. (As a FYI - it's not that hard to have your own build with say a later version of Postgres. You can get a copy of the artifact/package via the output from Github actions. However, you won't be able to get it in pypi)

**Installing postgresql-wheel**

```bash
pip install postgresql-wheel==14.1.0
```
