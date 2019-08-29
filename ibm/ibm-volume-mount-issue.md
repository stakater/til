# IBM Volume mount issue


During the deployment of statefulsets, deployment we faced an issue in which volume was not being mounted inside pod, although the same manifest was running before without any issue. 

The manifest is given below:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql-svc
  namespace: dev
spec:
  ports:
  - name: "mysql-port"
    port: 3306
    targetPort: 3306
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: dev
spec:
  serviceName: "mysql"
  selector:
    matchLabels:
      app: mysql
  replicas: 1
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: mysql
    spec:   
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "app"
        effect: "NoSchedule"

      initContainers:
      - image: busybox
        name: mysql-volume-cleaner
        args: [/bin/sh, -c, 'rm -rf /var/lib/mysql/lost+found || true']
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mysql-pvc

      containers:
      - image: stakater/mysql-backup-restore-s3:ibm
        name: mysql-backup-restore
        volumeMounts:
        - mountPath: /backup
          name: mysql-pvc    
        env:
        - name: MYSQL_DB
          value: "ams"
        - name: CRON_TIME
          value: "0 */1 * * *"
        - name: MYSQL_USER
          value: "root"
        - name: MYSQL_PASS
          valueFrom:
            secretKeyRef:
              name: mysql
              key: mysql-root-password
        - name: S3_BUCKET_NAME
          valueFrom:
            secretKeyRef:
              name: aws-secrets
              key: mysql_bucket
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-secrets
              key: aws_access_key_id
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-secrets
              key: aws_secret_access_key
        - name: AWS_DEFAULT_REGION
          valueFrom:
            secretKeyRef:
              name: aws-secrets
              key: aws_default_region
        - name: MYSQL_HOST
          value: "127.0.0.1"
        - name: MYSQL_PORT
          value: "3306"
        - name: RESTORE
          value: "true"

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
        resources: {}          
  volumeClaimTemplates:
  - metadata:
      name: mysql-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: ibmc-block-bronze
      resources:
        requests:
          storage: 2Gi

```

while the error that pod was generating is given below:

```
pod has unbound immediate PersistentVolumeClaims (repeated 3 times)
Unable to mount volumes for pod "mysql-0_test-namespace(8257f190-c986-11e9-8780-2ab117253e6f)": timeout expired waiting for volumes to attach or mount for pod "test-namespace"/"mysql-0". list of unmounted volumes=[mysql-pvc]. list of unattached volumes=[mysql-pvc default-token-p2qgn]
```

Initially, we thought that the issue is with our configurations but after deploying different manifest in different namespace we found the same issue. This issue was found on date 27 and 28 August. 


