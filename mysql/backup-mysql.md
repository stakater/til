# Restore MySQL

## Overview

This guide provides guidelines on how to restore mysql using the `SQL.sql` file.


## Steps

Follow this steps to restore mysql database:

* Copy the `SQL.sql` file to the machine where mysql has been installed.

* It requires that database `xyz` should exists in the database initially.

* Run the command given below to backup the database:

```bash
$ mysql -u <username> -p <database_name> < <SQL>.sql
```

* To check whether data has been restored or not follow the steps given below:

```bash

# it will ask for password enter the password
$ mysql -u root  -p

# below is mysql shell
$ mysql> use <db>;

# run this command to check whether records exists or not
$ mysql> select count(*) from <table-name>;
```
