# Restore Mongodb using mongorestore utility

## Overview

This guide provides guidelines on how to restore mongodb using mongorestore utility

## Steps

Follow the steps given in this guideline for data restoration:

* Move the `dumps` folder to the location when mongodb instance is running.

* Run the comamnd given below to restore the database:

```bash

$ mongorestore dumps/
```

* To check whether data has been restored or not follow the steps given below:
```bash

$ mongo

# mongo shell steps
$ mongo> show dbs;

# choose a database
$ mongo> use <xyz>;

# to print tables
$ mongo> show collections;

# to get the records
$ mongo> db.<table-name>.find()
 
```
