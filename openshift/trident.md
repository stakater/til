Keep in mind that the trident pod talks to the kubernetes service in the default project to reach the api.

As at one point I saw these errors in the pod:

```
E1101 13:51:29.080316       1 reflector.go:304] github.com/netapp/trident/frontend/kubernetes/plugin.go:228: Failed to watch *v1.StorageClass: Get https://100.244.0.1:443/apis/storage.k8s.io/v1/storageclasses?resourceVersion=71427920&timeoutSeconds=365&watch=true:  dial tcp 100.244.0.1:443: getsockopt: connection refused
E1101 13:51:29.080579       1 reflector.go:304] github.com/netapp/trident/frontend/kubernetes/plugin.go:227: Failed to watch *v1.PersistentVolume: Get https://100.244.0.1:443/api/v1/persistentvolumes?resourceVersion=71427372&timeoutSeconds=503&watch=true:  dial tcp 100.244.0.1:443: getsockopt: connection refused
```

I then killed the pod and then logs were bit cleaner!

To troubleshoot do following.

First try to create a Pod with a PVC:

```
oc create -f counter-pod.yaml -n=trident
```

This creates a pod and a pvc in trident namespace/project:

counter-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 200s; done']
    volumeMounts:
    - name: counter-storage
      mountPath: /etc/config
  volumes:
  - name: counter-storage
    persistentVolumeClaim:
     claimName: counter-storage-pv
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: counter-storage-pv
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: nfs-ssd
```

If all is OK then trident pod should log something like this:

```
time="2018-11-12T08:50:19Z" level=info msg="Kubernetes frontend provisioned a volume and a PV for the PVC." PV=trident-counter-storage-pv-f7eb7 PVC=counter-storage-pv PV_volume=trident-counter-storage-pv-f7eb7 
```
