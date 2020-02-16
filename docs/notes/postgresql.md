# Notes from Udacity DEND for PostgreSQL

## Using Jyupter Notebook to run SQL

- You can use ipython-sql library. And, load it into the Jyupter notebook using `%load_ext sql`.
- To execute SQL queries, you write on of the following at the top of your cell.
        - `%sql`: 
                - For a one-liner SQL query
                - You can access a python variable using `$`
        - `%%SQL`:
                - For a multi-line SQL query
                - You can **NOT** access a python variable using `$`
- Running a connection like: `postgresql://postgres:postgres@db:5432/pagila` to connect to the database.

## Creating a database and fill it with data

- Adding `!` at the beginning at the Jyupter cell runs a command in shell. For example, we're not running python code but we are running the `createdb` and `psql` postgresql command-line utilities.
- An example:

```Python
!PGPASSWORD=student createdb -h 127.0.0.1 -U student pagila
!PGPASSWORD=student psql -q -h 127.0.0.1 -U student -d pagila -f Data/pagila-schema.sql
!PGPASSWORD=student psql -q -h 127.0.0.1 -U student -d pagila -f Data/pagila-data.sql
```

## Connect to a database

Load the ipython-sql, create the connection variable, then connect to database.

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

## Retrieve the database schema

```SQL
%%sql  
SELECT column_name, data_type  
FROM information_schema.columns  
WHERE table_name = "dimdate';
```

## Create FACT table with foreign keys using REFERENCES constraint

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

## Drop tables that contain foreign keys

You need to drop the table that has foreign keys references before you can drop other tables.

```SQL
%%sql 
DROP TABLE IF EXISTS factsales;
DROP TABLE IF EXISTS dimdate;
DROP TABLE IF EXISTS dimstore;
DROP TABLE IF EXISTS dimmovie;
DROP TABLE IF EXISTS dimcustomer;
```

## ETL process from one database (Normalized) to new tables (Star Schema)

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

## Check the performance of the script

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

## Using columnar storage extension in PostgreSQL

The extension is cstore_fdw by citus_data: [github link](https://github.com/citusdata/cstore_fdw)

An example:

```SQL
%%sql
-- load extension first time after install
CREATE EXTENSION cstore_fdw;

-- create server object
CREATE SERVER cstore_server FOREIGN DATA WRAPPER cstore_fdw;

-- create foreign table, i.e. customer reviews column
DROP FOREIGN TABLE IF EXISTS customer_reviews_col
------
CREATE FOREIGN TABLE customer_reviews_col (
        customer_id TEXT,
        review_date DATE,
        review_rating INTEGER,
        review_votes INTEGER,
        review_helpful_votes INTEGER,
        product_id CHAR(10),
        product_title TEXT,
        product_sales_rank BIGINT,
        product_group TEXT,
        product_category TEXT,
        product_subcategory TEXT,
        similar_product_ids CHAR(10)[]
)
-------------
-- leave code below as is
SERVER cstore_server
OPTIONS(compression 'pglz');

```