---
layout: post
title: PostgreSQL vs. MySQL vs. SQLite
description: Database comparison on popular RDBMS
summary: Database comparison on popular RDBMS.
comments: true
tags: [database]
---

One of questions that I asked myself when I need to do select some (RDBMS) DB technology for my project is: which DB is the most appropriate to my project. My head pop out these three RDBMS: PostgreSQL, MySQL and SQLite.

Many projects I deal with are web-based. So I can eliminate SQLite from my choice (It's file based and I think it isn't good on scalable). But MySQL and PostgreSQL are still in my choices. I familiar with MySQL so I chose MySQL as DB in my many projects. But I still wonder when do I need to use PostgreSQL. And today I do some research on this question.

- [SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems](https://www.digitalocean.com/community/tutorials/sqlite-vs-mysql-vs-postgresql-a-comparison-of-relational-database-management-systems)
- [System Properties Comparison MySQL vs. PostgreSQL](https://db-engines.com/en/system/MySQL%3BPostgreSQL)

After reading those two articles. (my research isn't deep more than reading one on two articles, sorry :D) I realize that MySQL is gain more popularity than PostgreSQL in term of easy to use and has capability to handle several projects. But it some projects that has more complex query, complex procedure, fully-implemented SQL from standards and need more data integrity. PostgreSQL is a good option to use.

Choose wisely! :D