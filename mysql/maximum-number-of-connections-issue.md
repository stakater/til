# Issue with maximum number of connections limit in MYSQL

SQL sets a limit on the number of concurrent connections it will allow, the default value being `150`. Whenever it reaches
this limit it will stop accepting any more connections which acts as a blocker for all the services that are consuming sql
and would like to open new connections. If you expect more connections than this limit you need to update `max_connections` 
variable in MYSQL configuration.

## Access database and first review the current value using the following query: 
`show variables like 'max_connections';`

## Update this attribute:
1. Directly in SQL(Not recommended):
`SET GLOBAL max_connections = 150;`

2. If you are runing container directly:
`docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=test -p 3306:3306 -d mysql:8 --max_connections=10000`

3. By specifying args in it's corresponding manifest:

```yaml
...
...
containers:
  - name: mysql
    image: mysql:8.0
    args: ["--max_connections=10000"]
    ports:
      - containerPort: 3306
        name: mysql
    env:
      - name: MYSQL_ROOT_PASSWORD
...
...
```

4. MYSQL stores these environment variable in configuration file that exists at `/etc/my.cnf` or `/etc/mysql/my.cnf`.
We can directly mount our custom configuration to use our pre-set values.

* Create configmap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: namespace-name
data:
  sql_config: |
    [mysql]
    max_connections=10000
    max-allowed-packet=67108864
```

* Consume this configmap in your mysql manifest:

```yaml
...
...
volumes:
  - name: mysql-config-volume
    configMap:
      name: mysql-config
containers:
  - name: mysql
    image: mysql:8.0
    ports:
      - containerPort: 3306
        name: mysql
    volumeMounts:
      - name: mysql-config-volume
        mountPath: /etc/mysql/my.cnf
        subPath: sql_config
    env:
      - name: MYSQL_ROOT_PASSWORD
...
...
```


## Verify that the value has been updated using the same command again:
`show variables like 'max_connections';`
