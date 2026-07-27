---
title: Export PostgreSQL Table to CSV File
source: https://neon.com/postgresql/tutorial/export-postgresql-table-to-csv-file
created: 2026-07-27
tags:
  - database
  - postgresql
---
In this note, we will learn various techniques to export data from PostgreSQL tables to CSV files.

We will use the same `persons` table as in [Import CSV Files into PostgreSQL Table](Import%20CSV%20Files%20into%20PostgreSQL%20Table.md).

![](../../assets/Pasted%20image%2020260727205318.png)

## Export data from a table to CSV using the `COPY` statement

The `COPY` statement allows us to export data from a table to a CSV file.

For example, if we want to export the data of the `persons` table to a CSV file named `persons_db.csv` in the `C:\temp` folder, we can do so using the following:

```PostgreSQL
COPY persons TO 'C:\temp\persons_db.csv' DELIMITER ',' CSV HEADER;
```

Output:

```PostgreSQL
COPY 2
```

Sometimes, we may want to export data from some columns of a table to a CSV file. We can achieve this by specifying column names after table name inside a bracket.

```PostgreSQL
COPY persons(first_name,last_name,email)
TO 'C:\temp\persons_partial_db.csv' DELIMITER ',' CSV HEADER;
```

![](../../assets/Pasted%20image%2020260727210004.png)

If we don't want to export the header, which contains the column names of the table, we can do so by simply omitting the `HEADER` flag.

```PostgreSQL
COPY persons(email)
TO 'C:\temp\persons_email_db.csv' DELIMITER ',' CSV;
```

![](../../assets/Pasted%20image%2020260727210114.png)

Notice that the CSV file name that you specify in the `COPY` command must be written directly by the server.

It means that the CSV file must reside on the database server machine, not your local machine.

## Export data from a table to a CSV file using the \copy command

If we have access to a remote PostgreSQL database server, but you don't have sufficient privileges to write to a file on it, we can use the PostgreSQL built-in command `\copy`.

The `\copy` command runs the `COPY` statement behind the scenes. However, instead of the server writing the CSV file, psql writes the CSV file and transfers data from the server to your local file system.

To use `\copy` command, you need to have sufficient privileges to your local machine. It does not require PostgreSQL superuser privileges.

For example, if you want to export all data from the `persons` table into `persons_client.csv` file, you can execute the `\copy` command from the psql client as follows:

```
\copy (SELECT * FROM persons) to 'C:\temp\persons_client.csv' with csv
```

