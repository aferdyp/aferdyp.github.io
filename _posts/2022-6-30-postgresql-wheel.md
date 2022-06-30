---
layout: post
comments: true
title:  "Postgresql wheels and pytest with Django"
date:   2022-06-30 00:00:00 +0000
categories: Python, pytest, Django, Postgresql
draft: true
---

Being able to run unit test cases with pytest along with a database (Postgres in this case), often times needs choices to be made on whether the DB queries need to be mocked or letting the queries run against a test database of sorts.

Setting up a database while also making sure that the right versions are altogether and the data is as needed is sometimes a balancing act. In order to try and mitigate some of these issues, I decided to try out the [postgresql-wheel](https://github.com/michelp/postgresql-wheel). The package comes with a "bundled" version of postgresql that suffices to run most Django ORM queries. There are similar projects like [pytest-postgresql](https://github.com/ClearcodeHQ/pytest-postgresql) that also let you get test cases working with pytest but they need a previous install of Postgresql (or atleast all of the postgres dependencies like libpq, pg_config, etc). This is not required with postgresql-wheel since it comes with batteries included.

