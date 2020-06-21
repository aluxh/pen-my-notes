# Notes about ODBC drivers, and how to connect via R

Notes taken while learning and using ODBC drivers to connect to databases in R Shiny apps during my daily work.

## Setting up ODBC drivers in Linux

The following is how I setup ODBC drivers using AWS EC2, usually Ubuntu or Debian.

### Install necessary drivers

To begin, install UnixODBC, which is required for all databases. Then, install common DB drivers.

```bash
# Install the unixODBC library
apt-get install unixodbc unixodbc-dev --install-suggests

# SQL Server ODBC Drivers (Free TDS)
apt-get install tdsodbc
  
# PostgreSQL ODBC ODBC Drivers
apt-get install odbc-postgresql
  
# MySQL ODBC Drivers
apt-get install libmyodbc
  
# SQLite ODBC Drivers
apt-get install libsqliteodbc
```

### Setup the db connections

The following are the files for setting up the DSN information (DSN = Data Source Name).

- **odbcinst.ini:** define driver options
- **odbc.ini:** define connection options

To find out where the configuration files are stored in your system:

```bash
odbcinst -j
```

Example of **odbcinst.ini**

```config
[PostgreSQL Driver]
Driver          = /usr/local/lib/psqlodbcw.so

[SQLite Driver]
Driver          = /usr/local/lib/libsqlite3odbc.dylib
```

Example of **odbc.ini**

The **driver**'s name must match the name listed in the `odbcinst.ini`

```config
[PostgreSQL]
Driver              = PostgreSQL Driver
Database            = test_db
Servername          = localhost
UserName            = postgres
Password            = password
Port                = 5432

[SQLite]
Driver          = SQLite Driver
Database        = /tmp/testing
```

## Setting up ODBC drivers in Windows

1. Open **ODBC Data Source Administator (64 bit) tool**
2. Under User DSN, press **Add**.
3. Select **PostgreSQL Unicode(x64)**.
4. Fill up the database details, and select **Test** to test the connection to database.

## Connecting to Database in R

Install odbc library

```R
install.packages("odbc")
```

Then, connect to the database using the DSN configuration files

```R
con <- DBI::dbConnect(odbc::odbc(), dsn="PostgreSQL")
```
