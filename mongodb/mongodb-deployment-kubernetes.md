# Mongodb Deployment on Kubernetes

## Overview

This document will provide guidelines on how to deploy mongodb database on kubernetes cluster(deployed on AWS) using k8s stateful sets.

## Deployment Steps

In this section we will deploy mongodb on kubernetes by following the steps given below:

* Create a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) using the manifest given below, this manifest is specific to AWS.

```yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: mongo-storage-class
provisioner: kubernetes.io/aws-efs
parameters:
  type: gp2
```

Storage class will be create at kubernetes cluster level and will be used by statefulsets to create volumes for each replica. To create storage class use the command given below:

```bash
$ sudo kubectl apply -f storage class
```

To check whether storage class is created or not use the command given below:
```bash
$ sudo kubectl get storageclasses
```

* Create a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) using the manifest given below:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
  labels:
    sma-service: mongo-service
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    role: mongo
```

To create the above service use the command given below:

```bash
$ sudo kubectl apply -f headless-service.yaml -n <namespace-name>
```
To check service is created use the command given below:

```
sudo kubectl get services -n <namespace-name>
```

* Create a [statefulset](https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset) using the manifest given below:

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 1
  template:
    metadata:
      labels:
        db-servicename: mongo
        environment: test
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mongo
          image: mongo:3.4
          command:
            - mongod
            - "--replSet"
            - rs0
            - "--smallfiles"
            - "--noprealloc"
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db

  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "mongo-storage-class"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 2Gi
```

Use the command given below to create the statefulset
```bash
$ sudo kubectl apply -f stateful-sets.yaml -n <namespace-name>
```

To check statefulset is created or not use the command given below:
```bash
$ sudo kubectl get sfs -n <namespace-name>
```



## Notes

### Scaling

* Scaling statefulsets will increase the number of volumes but scaling down will not delete the volumes because it requires the users to copy the data and delete the volumes manually.

* Until now the statefulsets doesn't share volumes.

* An [issue](https://github.com/cvallance/mongo-k8s-sidecar/issues/57) was faced by me and it was causing me to not access mongodb because I was not using the master node of statefulset to access the database. This issue was resolved by running the command given below:
```
$ mongo

$ rs.initialize()
```



### Refrences

| Description  | Links  |
|---|---|
| Google guidelines on mongodb deployment  |  https://codelabs.developers.google.com/codelabs/cloud-mongodb-statefulset/index.html?index=..%2F..index#0 |
| Blog post explaing issue in google guidelines  | http://pauldone.blogspot.com/2017/06/deploying-mongodb-on-kubernetes-gke25.html  |
| k8s Mongodb  | http://k8smongodb.net/  |
|   |   |

