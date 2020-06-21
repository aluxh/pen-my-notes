# IPython Magic Commands in Juypter notebooks

The following are notes taken while taking the Course - Data Engineering NanoDegree.

## Using IPython Magic commands to run SQL directly in Juypter notebooks

- Using `ipython-sql` library, and execute `%load_ext sql` to load the library in Juypter notebook.
- Using the following style at the top of each code cell to execute SQL queries:
  - `%sql`: This is for executing one-liner SQL query. And, you can access a python variable using `$`.
  - `%%sql`: This is for executing multi-line SQL queries. But, you cannot access a python variable using `$`

- Running a connection like: `%sql postgresql://postgres:postgres@db:5432/pagila` to connect to the database.

## Using `!` to run commandline in Juypter notebooks

- Add `!` at the beginning of the Juypter cell to run a commandline in shell. The following examples demonstrate how to run postgresql command-line utilities `createdb` and `psql`.

Note: Postgres version of Pagila database can be found [here](https://github.com/devrimgunduz/pagila)

```Python
!PGPASSWORD=student createdb -h 127.0.0.1 -U student pagila
!PGPASSWORD=student psql -q -h 127.0.0.1 -U student -d pagila -f Data/pagila-schema.sql
!PGPASSWORD=student psql -q -h 127.0.0.1 -U student -d pagila -f Data/pagila-data.sql
```

## Some examples of the IPython-magic commands with Juypter notebook and PostgreSQL database

### Connect to a database

Load the `ipython-sql` library, create the connection variable, then connect to database.

```Python
# Load ipython-sql library
%load_ext sql

# Connect to database
DB_ENDPOINT = '127.0.0.1'
DB = 'pagila'  
DB_USER = 'student'  
DB_PASSWORD = 'student'  
DB_PORT = '5432'

# Creating the connection string and keep in python variable:
conn_string = "postgresql://{}:{}@{}:{}/{}".format(DB_USER, DB_PASSWORD, DB_ENDPOINT, DB_PORT, DB)

# Using the python variable
%sql $conn_string
```

### Retrieve the database schema

```SQL
%%sql  
SELECT column_name, data_type  
FROM information_schema.columns  
WHERE table_name = "dimdate';
```

### Create FACT table with foreign keys using REFERENCES constraint

```SQL
%%sql
CREATE TABLE factSales
(
    sales_key SERIAL PRIMARY KEY,
    date_key integer REFERENCES dimdate (date_key),
    customer_key integer REFERENCES dimcustomer (customer_key),
    movie_key integer REFERENCES dimmovie (movie_key),
    store_key integer REFERENCES dimstore(store_key),
    sales_amount numeric
);
```

### Drop tables that contain foreign keys

You need to drop the **table that has foreign keys references** before you can drop other tables.

```SQL
%%sql
DROP TABLE IF EXISTS factsales;
DROP TABLE IF EXISTS dimdate;
DROP TABLE IF EXISTS dimstore;
DROP TABLE IF EXISTS dimmovie;
DROP TABLE IF EXISTS dimcustomer;
```

### ETL process from one database (Normalized) to new tables (Star Schema)

An example to extract date data from a `payment_date` table, then transform and load into the `dimDate` table

```SQL
%%sql
INSERT INTO dimDate (date_key, date, year, quarter, month, day, week, is_weekend)
SELECT DISTINCT(TO_CHAR(payment_date :: DATE, 'yyyyMMDD')::integer)     AS date_key,
       date(payment_date)                                               AS date,
       EXTRACT(year FROM payment_date)                                  AS year,
       EXTRACT(quarter FROM payment_date)                               AS quarter,
       EXTRACT(month FROM payment_date)                                 AS month,
       EXTRACT(day FROM payment_date)                                   AS day,
       EXTRACT(week FROM payment_date)                                  AS week,
       CASE WHEN EXTRACT(ISODOW FROM payment_date) IN (6, 7) THEN true ELSE false END AS is_weekend
FROM payment;
```

### Check the performance of the script

Add `%%time` at the top of the execution block. An example,

```SQL
%%time
%%sql
SELECT dimMovie.title, dimDate.month, dimCustomer.city, sum(sales_amount) AS revenue
FROM factSales
JOIN dimMovie    ON (dimMovie.movie_key      = factSales.movie_key)
JOIN dimDate     ON (dimDate.date_key         = factSales.date_key)
JOIN dimCustomer ON (dimCustomer.customer_key = factSales.customer_key)
GROUP BY (dimMovie.title, dimDate.month, dimCustomer.city)
ORDER BY dimMovie.title, dimDate.month, dimCustomer.city, revenue DESC;
```
