---
title: The Database Versioning Showdown
layout: post
---

Versioning your database schema is a critical component of mature software development practices. Depending on your database there are number of tools available to version your schema. While most are adequate for the job of version control, many are not designed for *deployment* of schema changes.

A lot of development shots are practicing some form of continuous deployment (CD). Particularly those who are building software-as-a-service (SaaS). The ability to go from source to a working application, including the database, is very important. 