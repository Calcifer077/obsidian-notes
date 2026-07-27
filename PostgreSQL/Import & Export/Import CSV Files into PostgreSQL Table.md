---
title: Import CSV Files into PostgreSQL Table
source: https://neon.com/postgresql/tutorial/import-csv-file-into-posgresql-table
created: 2026-07-27
tags:
  - database
  - postgresql
---
In this note, we will learn various ways to import a CSV file into a PostgreSQL table.

First, create a table named `persons`:

```PostgreSQL
CREATE TABLE persons (
  id SERIAL,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  dob DATE,
  email VARCHAR(255),
  PRIMARY KEY (id)
);
```

Second, prepare a CSV data file with the following (same format as the table in which we want to import) format:

![](../../assets/Pasted%20image%2020260727163410.png)

Let's say the path of the file is: `C:\sampledb\persons.csv`

## Import using `COPY` statement

To import this CSV file into the `persons` table, we can use `COPY` statement as follows:

```PostgreSQL
COPY persons(first_name, last_name, dob, email)
FROM 'C:\sampledb\persons.csv'
DELIMITER ','
CSV HEADER;
```

PostgreSQL gives back the following message:

```PostgreSQL
COPY 2
```

It means that two rows have been copied. We can check it in the `persons` table.

Let's dive into the `COPY` statement in more detail:

First, we specify the table with column names after the `COPY` keyword. The order of the columns must be the same as the ones in the CSV file. In case the CSV file contains all columns of the table, we don't need to specify them explicitly, for example:

```PostgreSQL
COPY sample_table_name
FROM 'C:\sampledb\sample_data.csv'
DELIMITER ','
CSV HEADER;
```

Second, we put the CSV file path after the `FROM` keyword. Because CSV file format is used, we need to specify `DELIMITER` as well as `CSV` clauses.

Third, specify the `HEADER` keyword to indicate that the CSV file contains a header. When the `COPY` command imports data, it ignores the header of the file. `HEADER` is the first row (the column names) of a file during data transfer.

Notice that the file must be read directly by the PostgreSQL server, not by the client application. Therefore, it must be accessible by the PostgreSQL server machine. Also, you need to have superuser access to execute the `COPY` statement successfully.

## Import CSV file into a table using pgAdmin

In case we need to import a CSV file from our computer into a table on the PostgreSQL database server, we can use the pgAdmin.

The following statement [truncates](https://neon.com/postgresql/tutorial/postgresql-truncate-table) the `persons` table so that you can re-import the data.

```PostgreSQL
TRUNCATE TABLE persons
RESTART IDENTITY;
```

First, right-click the `persons` table and select the **Import/Export…** menu item:

![](../../assets/Pasted%20image%2020260727164339.png)

Second, (1) switch to import, (2) browse to the import file, (3) select the format as CSV, (4) select the delimiter as comma (`,`):

![](../../assets/Pasted%20image%2020260727164400.png)

Third, click the columns tab, uncheck the id column, and click the OK button:

![](../../assets/Pasted%20image%2020260727164414.png)

Finally, wait for the import process to complete. The following shows the dialog that informs us of the progress of the import:

![](../../assets/Pasted%20image%2020260727164509.png)

