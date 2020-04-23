---
layout: post
title: Pessimistic locking and Optimistic locking in RDBMS
bigimg: /img/image-header/yourself.jpeg
tags: [Database]
---




<br>

## Table of contents
- [Given problem](#given-problem)
- [Solution of locking in RDBMS](#solution-of-locking-in-rdbms)
- [Pessimistic locking](#pessimistic-locking)
- [Optimistic locking](#optimistic-locking)
- [Wrapping up](#wrapping-up)

<br>

## Given problem

In single tier architecture, we have all things in one computer. It will include user interface, backend business logic, and database. With this architecture, normally we will only have one user to use this application. So, our database only takes care about containing data.

But from two tier architecture, n-tier architecture, we have multiple users in our system. It means that at the same time, we will have multiple requests to interact with our database. There are many problems will raise in this context, especially resolving conflicts of read/write operations between multiple requests at the same table or rows in database.

Belows are some problems that we have to deal with.
- Lost update

    Assuming that we have Person 1 and Person that interact with our database such as Employee table.

    |   Time   |        Person 1        |        Person 2        |
    | -------- | ---------------------- | ---------------------- |
    | T1       | Read row with id = 1   |                        |
    | T2       |                        | Read row with id = 1   |
    | T3       | Write this changed row to DB |                  |
    | T4       |                        | Write this changed row to DB |

    We can find that Person 1's update of the database has been lost by some changes of Person 2. So, we can called it as the lost update syndrome.

    If it happens in the blocks of database, we can take the negative consequence of it.

- Uncommitted dependency problem

    This problem is related to the Commit/Rollback mechanism of the transaction management.

    



- Inconsistent analysis problem



<br>

## Solution of locking in RDBMS

To overcome this problem, we need to protect our data from multiple requests that interact the same table or row. So we need locking.


<br>

## Pessimistic locking






<br>

## Optimistic locking





<br>

## Comparison between Pessimistic locking and Optimistic locking





<br>

## Understanding about transaction management

Database transactions are designed to prevent the database being left in an inconsistent state regardless of the circumstances of the attempted update.

A transaction is a group of database operations that is treated as an atomic unit, for example: they are all completed or none of them is completed.

The commit statement signals the successful end of a series of updates within one transaction. It tells the DBMS to save all the amended data and to terminate the current transaction.

The rollback statement aborts the current transaction. All updates, within the current transaction, are cancelled and the database reverts to its state before the start of transaction.



<br>

## Wrapping up




<br>

Refer:

[http://git@github.com.github.com/blog/2013/05/13/exclusivelock-sharedlock/](http://git@github.com.github.com/blog/2013/05/13/exclusivelock-sharedlock/)
