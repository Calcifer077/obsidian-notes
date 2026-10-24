---
title: PostgreSQL HSTORE Data Type
source: https://neon.com/postgresql/tutorial/hstore
created: 2026-10-24
tags:
  - postgresql
---
The hstore module implements the hstore data type for storing key-value pairs in a single value. The keys and values are text strings only.

## Enable PostgreSQL hstore extension 

Before working with the hstore data type, you need to enable the hstore extension which loads the `contrib` module to your PostgreSQL instance.

```PostgreSQL
CREATE EXTENSION hstore;
```

## Create a table with hstore data type

```PostgreSQL
CREATE TABLE books (
	id serial primary key,
	title VARCHAR (255),
	attr hstore
);
```

Here, `attr` stores attributes of the book such as ISBN, weight, and paperback. 

## Insert data into table

```PostgreSQL
INSERT INTO books (title, attr)
VALUES
  (
    'PostgreSQL Tutorial', '"paperback" => "243",
     "publisher" => "postgresqltutorial.com",
     "language"  => "English",
     "ISBN-13"   => "978-1449370000",
     "weight"    => "11.2 ounces"'
  ),  (
    'PostgreSQL Cheat Sheet', '
	"paperback" => "5",
	"publisher" => "postgresqltutorial.com",
	"language"  => "English",
	"ISBN-13"   => "978-1449370001",
	"weight"    => "1 ounces"'
  );
```

The data that we insert into the hstore column is a list of comma-separated key =>value pairs. Both keys and values are quoted using double quotes (“”).

>Go to source, they are pretty simple examples. (I don't want to copy paste. 🦥)

