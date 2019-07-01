# Mongodb Backup from k8s Pod

## Overview

This guide will provide guidelines on how to backup mongdb from mongodb kubernetes pod.

## Steps

* Access the mongodb primary pod using the command given below:

```bash
$ sudo kubectl -n <namespace-name> exec -it <pod-name> /bin/bash 
```

* Once inside the container, dump the mongo db database using the command given below:

```bash
$ mongodump --db <database-name> -o <output directory>
```
Because of the container environment we cannot create a dump folder at the root level `/`. This issue can be resolved by creating a folder(`mongo-dump`) inside `/tmp` directory. Now copy the path and replace it with output directory section of the above command.


* To copy the dump from mongodb pod to local system use the command given below:
```bash
$ kubectl cp {{namespace}}/{{podname}}:path/to/directory /local/path
```

* To import the database use the command given below:
```bash
$ mongoimport --db music --file /data/dump/music/artists.json
```