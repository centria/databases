---
title: "Introduction"
permalink: /part1/
nav_order: 3
published: true
---

## What is a database?

A *database* is an organized collection of data, generally stored and accessed electronically from a computer system. We can perform *queries* on them and change their content. For example the following are databases:

* Usernames and passwords of web pages
* Products and storage status of a web store
* Banks' information about their customers and their accounts
* Daily weather measurements in different locations
* Flight schedules and booking status for airlines

There are a vast amount of databases, and most people are in contact with multiple databases during an ordinary day.

## Challenges for databases

There are several technical difficulties conserning databases, and it is not easy to build a well functioning database. The most common challenges are:

## Amount of information

Many databases hold large quantities of information, into which multiple queries and changes are made constantly. How to make the database in such a way, that the information can be accessed efficiently?

## Concurrency

Usually a database has multiple users, who can change and get information at the same time. Why do we have to take this into account, when designing our database?

## Surprises

The database should stay coherent in surprising situations, as well. For example, what happens if the power goes out when the data is being changed?

# Developing databases

The development of databases took flight in 1970's and there were multiple different solutions, but one became the most popular: relational model and the **S**tructural **Q**uery **L**anguage, **SQL**. The relational databases have been popular since then, and most of the databases are still based on relational database models.

In this course we will concentrate on relational database from both database users' and the technical point of view. We aim to answer the question, *why* relational model is such a good way to create a database?

Even though relational databases are still dominant, recently some powerful challengers have risen. One reason has been the need for different kinds of databases, which are more suitable for distributed systems, such as enormous websites.

A term often used, *NoSQL*, refers to a database based on something else than the relational model and SQL language. Especially document-oriented databases have been popular lately. Even though we concentrate on the relational model, it is good to keep in mind that alternatives exist.

# Simple database

Before we start to look into existing database systems, let's try and make a database and its handling *by ourselves* in some simple way and see, what kinds of problems we run into.

Let's assume we want to create a database for a bank. The purpose of the database is to store the customer information and the transactions on their accounts.

## Database structure

We will store the database into a text file, `bank.txt`, whose lines reflect the transactions in the database. There are two different transactions: You can create a new account into the bank (the balance) or the balance of an account changes. The content of the file could be something like this:

```
CREATE ACCOUNT
NUMBER: 131778223
OWNER: Uolevi
CREATE ACCOUNT
NUMBER: 175299717
OWNER: Maija
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: 500
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: -100
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: 100
```