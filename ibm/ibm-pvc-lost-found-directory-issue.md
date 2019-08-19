# IBM PVC lost+found directory issue

## Overview
This document is about the `lost+found` directory issue in the IBM pvc.


## Details

When a pvc is created in IBM cloud, a `lost+found` is added on the mount path. According to the IBM support :

```
lost+found folder comes by default when the volume is mounted so it cannot be disabled and is expected behavior. Its not to be used for any data persistence purposes. You can create new folder in mount path and save data. MySQL is commonly used with block storage and it works fine.
```


VOBs store file system objects and metadata. File system objects are files and directories. These objects are stored as elements, each of which contains one or more versions. A file element contains one or more versions of a file. A directory element contains one or more versions of a directory, each of which can contain file elements and other directory elements.

A version is a specific revision of a file or directory. Versions of text files are distinguished by changes in their text. Versions of directories are distinguished by changes in their contents (to account for files and subdirectories that have been renamed, removed, or added).


Every VOB includes a special directory element, lost+found, which is used to hold elements that become stranded when they are not cataloged in any directory version in the VOB.

An element can become stranded when you do any of the following:

- Create new elements, and then cancel the checkout of directory in which they were created
- Delete the last reference to an element by using the rmname command
- Delete the last reference to an element by deleting a directory version with the `rmver`, `rmbranch`, or `rmelem` command


## Issue
`lost+found` directory caused an issue in the deployment of MySQL deployement on IBM cloud and caused this error:

```
--initialize specified but the data directory has files in it. Aborting
```

after debugging, I found out that

```
mysql container’s entrypoint script checks if DATADIR/mysql (in my case it was /var/lib/mysql) exists. If not, it run --initialize, but if DATADIR/mysql contains any files other than ones starting with . or specified with --ignore-db-dir, --initialize will fail with the above error message.
```

## Solution
I resolved the above directory by using an `init-container`, which removes the `lost+found` directory, manifest is given below

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: test-namesapce
spec:
  serviceName: "mysql"
  selector:
    matchLabels:
      app: mysql
  replicas: 1 
  template:
    metadata:
      labels:
        app: mysql
    spec:

      initContainers:
      - image: debian:stable-slim
        name: mysql-volume-cleaner
        command: ["rm", "-r", "/var/lib/mysql/lost+found"]
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mysql-pvc
    
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: mysql-root-password
        ports:
        - containerPort: 3306
          name: tcp
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mysql-pvc
  
  volumeClaimTemplates:
  - metadata:
      name: mysql-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: ibmc-block-bronze
      resources:
        requests:
          storage: 2Gi

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql-svc
  namespace: test-namesapce
spec:
  ports:
  - name: "mysql-port"
    port: 3306
    targetPort: 3306
  selector:
    app: mysql
```

## References

| Description | Link |
|---|---|
| lost+found directory | https://www.ibm.com/support/knowledgecenter/en/SSSH27_9.0.1/com.ibm.rational.clearcase.cc_admin.doc/topics/r_vobadm_stgmgmt_lf.htm |
| VOBs and Versioned Objects | https://www.ibm.com/support/knowledgecenter/SSSH27_9.0.0/com.ibm.rational.clearcase.ccrc.help.doc/topics/c_vobver.htm |
| mysql --intialize issue | https://github.com/docker-library/mysql/issues/290 |
| purpose of lost+folder in linux | https://unix.stackexchange.com/questions/18154/what-is-the-purpose-of-the-lostfound-folder-in-linux-and-unix |