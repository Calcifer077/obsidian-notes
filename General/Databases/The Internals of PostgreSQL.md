---
title: The Internals of PostgreSQL
source: https://www.interdb.jp/pg/index.html
created: 2026-10-11
tags:
  - database
  - postgresql
---
# Database Cluster, Databases and Tables 

A database cluster is the entire set of databases managed by a PostgreSQL instance, and a database is a logical collection of tables. Tables are the basic structures that store user data. 

## Logical Structure of Database Cluster 

A **database cluster** is a collection of _databases_ managed by a PostgreSQL instance. The term "database cluster" does not means "a group of database servers". A PostgreSQL instance runs on a single host and manages a single database cluster. 

![](../../assets/Pasted%20image%2020261011122111.png)

In above figure, we can see that there is a single database cluster which contains many database objects which contains tables, indexes, sequences, views and functions. In PostgreSQL, database themselves are also considered database objects and are logically isolated from one another. 

All the database objects in PostgreSQL are internally managed by respective **object identifiers (OIDs)**, which are unsigned 4-byte integers. The relations between database objects and their respective OIDs are stored in appropriate system catalogs. 

## Physical Structure of Database Cluster 

The logical structure described above is physically represented by a directory structure and data files. 

A database cluster is basically a single directory, referred to as the **base directory**, except for tablespaces. It contains several subdirectories and many files. Executing the [initdb](http://www.postgresql.org/docs/current/static/app-initdb.html) utility to initialize a new database cluster creates a base directory under the specified location. The path of the base directory is usually set in the environment variable _PGDATA_. 

![](../../assets/Pasted%20image%2020261011122826.png)

A database is a subdirectory under the base directory, and each table or index is at least one file stored under its respective database subdirectory. Several other subdirectories contain specific data and configuration files.

A tablespace in PostgreSQL is a single directory that contains data outside the base directory. 

### Layout of a Database Cluster 

The base directory contains several files and subdirectories that store cluster-wide information and database-specific data. 

Different files serves different purpose. You can check them out at below table:

See table here: [](https://www.interdb.jp/pg/pgsql01/02.html)

### Layout of Databases 

Each database is a subdirectory located under the base directory. The names 