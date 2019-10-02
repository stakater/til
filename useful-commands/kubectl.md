# Kubectl

## Note:
When Running Kubectle run if
```
--restart=Never      (creates Pod)
--restart=Always     (creates Deployments)
--restart=onFaliure  (creates Jobs)
```

## Delete Everything in a specific namespace
```
kubectl delete all --all -n $(NAMESPACE)
```

## Create a pod without writing YAML
```
kubectl run nginx --image=nginx --dry-run -o yaml -n default > pod.yaml
```

## Run a command instantly in a pod using kubectl
```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml --command  -- env > envpod.yaml
kubectl apply -f envpod.yaml
```

## Define a ResourceQuota object with CPU, Memory and Pod limits using kubectl
```
kubectl create quota myquota --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```

## Create a pod and allow a port on the container
```
kubectl run nginx --image=nginx --restart=Never --port 80 --dry-run -o yaml
```

## Change the image of a running pod using kubectl
```
kubectl set image pod/nginx nginx=nginx:1.7.1
``` 

## Create a temporary (create/run/delete) pod and run a command for stdout
```
kubectl run busybox --image=busybox --rm -it --restart=Never --command -- wget http://google.com
```

## Get IP for a pod
```
kubectl get pods nginx -o wide
OR
kubectl get pods nginx -o jsonpath='{.status.podIP}'
```

## If pod crashed and restarted. Get logs from previous instance
```
kubectl logs nginx -p
```

## Define environment variables using kubectl
```
kubectl run nginx --image=nginx --restart=Never --env var1=val1
```

## Show labels of all pods
```
kubectl get pods --show-labels
```

## Label a pod 
```
kubectl label pods/nginx app=new
```

## select pods with label `app`
```
kubectl get pods -l app
```

## Remove label `app` if exist
```
kubectl label -l app app-
```

## Add annotation on the pod
```
kubectl annotate pods/nginx icon-url=http://goo.gl/XXBTWq 
```

## Remove the annotations from pod
```
kubectl annotate pods/ngnix icon-url-
```

## Create a deployment using kubectl
```
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run -o yaml
```

## Check how the deployment rollout is going
```
kubectl rollout status deployment/nginx
```

## Undo the latest rollout for deployment
```
kubectl rollout undo deployment/nginx 
```

## Undo rollout to a specific revision 2
```
kubectl rollout undo deployment/nginx --to-revision 2 
```

## Scale up deployments
```
kubectl scale --replicas 3 deployment/nginx
```

## AutoScale a deployment for min 5 and max 10 if CPU usage > 80%
```
kubectl autoscale deployment/nginx --min 5 --max 10 --cpu-percent=80
```

## Get pods for a specific replica set.
Replica sets have naming convention `{deployment-name}-{hash}`. Pods created by the replicaset have labels for this hash i.e. pod created by the replicaset `nginx-12345678` would have label `pod-template-hash=12345678`
```
kubectl get pods -l pod-template-hash=12345678
```

## Create a job with parallelism/completion
Create a job via kubeclt then add `completions/parallelism` under job.spec
```
kubectl create job --dry-run -o yaml --image=busybox -- /bin/sh -c 'echo Hello' > job.yaml
```
Edit the job.yaml to place completions/parallelism values under spec

## Cordon off a node for maintenance
```
kubectl cordon my_node
```

## Evict pods (re-schedule on other nodes)
```
kubectl drain my_node
```