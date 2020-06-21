# Notes about PostgreSQL database

Some notes taken while learning and using PostgreSQL Database during Data Engineering Nanodegree, and daily work.

## Using columnar storage extension, *cstore_fdw*, in PostgreSQL

The extension is `cstore_fdw` by **citus_data**: [github link](https://github.com/citusdata/cstore_fdw)

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
---------------------------------
-- leave code below as is
SERVER cstore_server
OPTIONS(compression 'pglz');

```
