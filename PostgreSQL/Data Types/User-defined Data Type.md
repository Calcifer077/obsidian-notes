---
title: PostgreSQL User-defined Data Types
source: https://neon.com/postgresql/tutorial/user-defined-data-types
created: 2026-10-24
tags:
  - postgresql
---
Besides built-in data types, PostgreSQL allows you to create user-defined data types through the following statements:

- `CREATE DOMAIN` creates a user-defined data type with constraints such as [`NOT NULL`](https://neon.com/postgresql/tutorial/postgresql-not-null-constraint), [`CHECK`](https://neon.com/postgresql/tutorial/postgresql-check-constraint), etc.
- `CREATE TYPE` creates a composite type used in [stored procedures](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-procedure) as the data types of returned values.

## CREATE DOMAIN statement 

In PostgreSQL, a domain is a data type with optional constraints e.g., `NOT NULL` and `CHECK`. A domain has a unique name within the schema scope.

Domains are useful for centralizing the management of fields with common constraints. For example, some tables may have the same column that do not accept NULL and spaces.

The following statement create a table called `mailing_list`:

```PostgreSQL
CREATE TABLE mailing_list (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR NOT NULL,
    last_name VARCHAR NOT NULL,
    email VARCHAR NOT NULL,
    CHECK (
        first_name !~ '\s'
        AND last_name !~ '\s'
    )
);
```

In the above table both, `first_name` and `last_name` share same constraints. To do DIY or reuse the logic you can create a domain.

The following statement uses the `CREATE DOMAIN` to create a new domain called `contact_name` with the `VARCHAR` datatype and do not accept `NULL` and spaces.

```PostgreSQL
CREATE DOMAIN contact_name AS
   VARCHAR NOT NULL CHECK (value !~ '\s');
```

And you use the `contact_name` as the datatype of the `first_name` and `last_name` columns as a regular built-in type:

```PostgreSQL
CREATE TABLE mailing_list (
    id serial PRIMARY KEY,
    first_name contact_name,
    last_name contact_name,
    email VARCHAR NOT NULL
);
```

If you insert invalid values like space in `first_name`:

```PostgreSQL
INSERT INTO mailing_list (first_name, last_name, email)
VALUES('Jame V','Doe','james.doe@example.com');
```

PostgreSQL will issue a error above.

To change or remove a domain, you use the `ALTER DOMAIN` or `DROP DOMAIN` respectively.

To view all domains in the current database, you use the `\dD` command as follows:

```
test=#\dD
                                     List of domains
 Schema |     Name     |       Type        | Modifier |               Check
--------+--------------+-------------------+----------+-----------------------------------
 public | contact_name | character varying | not null | CHECK (VALUE::text !~ '\s'::text)
(1 row)
```

### Getting domain information 

To get all domains in a specific schema, you use the following query:

```PostgreSQL
SELECT typname
FROM pg_catalog.pg_type
  JOIN pg_catalog.pg_namespace
  	ON pg_namespace.oid = pg_type.typnamespace
WHERE
	typtype = 'd' and nspname = '<schema_name>';
```

The following statement returns domains in the `public` schema of the current database:

```PostgreSQL
SELECT typname
FROM pg_catalog.pg_type
  JOIN pg_catalog.pg_namespace
  	ON pg_namespace.oid = pg_type.typnamespace
WHERE
	typtype = 'd' and nspname = 'public';
```

![](../../assets/Pasted%20image%2020261024174618.png)

## PostgreSQL CREATE TYPE 

The `CREATE TYPE` statement allows you to create a composite type, which can be used as the return type of a function.

Suppose you want to have a function that returns several values : `film_id`, `title`, and `release_year`. The first step is to create a type e.g., `film_summary` as follows:

```PostgreSQL
CREATE TYPE film_summary AS (
    film_id INT,
    title VARCHAR,
    release_year SMALLINT
);
```

Second, use the `film_summary` data type as the return type of a function:

```PostgreSQL
CREATE OR REPLACE FUNCTION get_film_summary (f_id INT)
    RETURNS film_summary AS
$$
SELECT
    film_id,
    title,
    release_year
FROM
    film
WHERE
    film_id = f_id ;
$$
LANGUAGE SQL;
```

Third, call the `get_film_summary()` function:

```PostgreSQL
SELECT * FROM get_film_summary (40);
```

![](../../assets/Pasted%20image%2020261024180213.png)

To change a user-defined type, you use the `ALTER TYPE` statement. To remove a user-defined type, you use the `DROP TYPE` statement.

If you use the `psql` program, you can list all user-defined types in the current database using the `\dT` or `\dT+` command:

```
dvdrental=# \dT
         List of data types
 Schema |     Name     | Description
--------+--------------+-------------
 public | contact_name |
 public | film_summary |
 public | mpaa_rating  |
(3 rows)
```

