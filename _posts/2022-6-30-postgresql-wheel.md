---
layout: post
comments: true
title:  "Postgresql wheels and pytest with Django"
date:   2022-06-30 00:00:00 +0000
categories: Python, pytest, Django, Postgresql
draft: true
---

Being able to run pytest unit test cases with a database (Postgres in this case), often times needs choices to be made on whether the DB queries need to be mocked or letting the queries run against a test database of sorts.
