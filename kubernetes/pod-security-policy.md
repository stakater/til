# Pod Security Policy

[Pod Security Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) (admission controller) enables declarative fine-grained authorization policies for pod creation and updates for users or service accounts. It is a cluster-level resource that controls security sensitive aspects of the pod specification.

#### Admission Controller

Admission controllers intercept requests to the kube-apiserver. The interception occurs before a requested object is persisted but after the request is authenticated and authorized. This enables us to see who or what the requested object came from and validate whether what's being asking for is appropriate. Admission controllers are enabled by adding them to the `kube-apiserver` flag `--enable-admission-plugins`.

## How to define policies

Pod Security Policy should be applied using this pattern:

- Initially, create a restrictive policy, to disable everything.
- Later Create permissive policies to enable access of `resources` for `service accounts` and `users` based on the use case. 


## How to enable Pod Security Policy

- Enable PSP by appending `PodSecurityPolicy` keyword in the `--enable-admission-plugins` element of `spec.containers.command` list, provided in `/etc/kubernetes/manifests/kube-apiserver.yaml` file. It will enable the Pod Security Policy but without the existance of any `policies` it will cause the new pod creation(including re-creating pods from a scheduling event) to `fail`.

Example:

```bash
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodSecurityPolicy
```

- In the example given below, I will provide an example on how to use it:

* Create an nginx deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
```

No pod will be created because of the unavalibility of any psp policy.


```bash
kubectl get po,rs,deploy

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-811c4cff45   1         0         0       9s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   0/1     0            0           9s
```

Delete the deployment
```bash
kubectl delete deploy nginx-deployment
```

- Now we will create a default restrictive policy to provide restrictive access, it will disable all the privileged access like `hostNetwork` etc. Detailed policy settings can be found on this [link](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#policy-reference)

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restrictive
spec:
  privileged: false
  hostNetwork: false
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  hostPID: false
  hostIPC: false
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - 'configMap'
  - 'downwardAPI'
  - 'emptyDir'
  - 'persistentVolumeClaim'
  - 'secret'
  - 'projected'
  allowedCapabilities:
  - '*'
```

Run this command to apply the above policy:
```bash
sudo kubectl apply -f <default-restrictive-policy>.yaml
```

- Now a permissive policy will be created for privileged creation rights in k8s cluster:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: permissive
spec:
  privileged: true
  hostNetwork: true
  hostIPC: true
  hostPID: true
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  hostPorts:
  - min: 0
    max: 65535
  volumes:
  - '*'
```
Run this command to apply the above policy:
```bash
sudo kubectl apply -f <default-restrictive-policy>.yaml
```

- Now when the policies(restrictive(default) and permissive) have been defined, we need to implement them on the `users` and `service accounts` using kubernetes RBAC. `ClusterRoles` will be created to use each of these policies.

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-restrictive
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - restrictive
  verbs:
  - use
```

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-permissive
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - permissive
  verbs:
  - use
```

With the ClusterRoles in place, let’s start by creating access to use the “default” restrictive policy.

Create a ClusterRoleBinding that grants the psp-restrictive ClusterRole to all controller (system) service accounts.

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-default
subjects:
- kind: Group
  name: system:serviceaccounts
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: psp-restrictive
  apiGroup: rbac.authorization.k8s.io
```

Create the nginx deployment again. As the `policies` have been defined now pods can be created

```
kubectl get po,rs,deploy

pod/nginx-hostnetwork-deployment-7c74c7d654-tl4v4   1/1     Running   0          3s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-7c74c7d654   1         1         1       3s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   1/1     1            1           3s
```

But if we try to do something that is disallowed in policy(restricitve), pods will not be deployed. To test it, we will create a new deployement that will try to use the `hostNetwork`(a privilaged access that has been disallowed in default restrictive policy). A new nginx deployment manifest is given below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
```

Before the creation of this deployment, delete the previous one.

New pod will not be created due to the use of privilaged resource access:

```bash
kubectl get po,rs,deploy

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-597c4cff45   1         0         0       9s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   0/1     0            0           9s
```

By looking at the logs of replica set controller, we can concluded that replica set controller doesn't have the right to create a pod with privilaged access.

```bash
kubectl describe replicaset.extensions/nginx-hostnetwork-deployment-597c4cff45

Events:
  Type     Reason        Age                 From                   Message
  ----     ------        ----                ----                   -------
  Warning  FailedCreate  35s (x14 over 76s)  replicaset-controller  Error creating: pods "nginx-hostnetwork-deployment-597c4cff45-" is forbidden: unable to validate against any pod security policy: [spec.securityContext.hostNetwork: Invalid value: true: Host network is not allowed to be used]
```

- Privileged rights can be provided to the controller by granting permissive pod security policy to their service accounts. In the example given below privileged access is being granted to the `service account` of `controllers` for `kube-system` namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: psp-permissive
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-permissive
subjects:
- kind: ServiceAccount
  name: daemon-set-controller
  namespace: kube-system
- kind: ServiceAccount
  name: replicaset-controller
  namespace: kube-system
- kind: ServiceAccount
  name: job-controller
  namespace: kube-system
```

Apply this RoleBinding by using this command:

```
sudo kubectl apply -f <privilaged-access-to-controller's-service-account>.yaml
```

Create a new deployment in `kube-system` namespace.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: kube-system
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
```

Now, it can be seen that the pod has been deployed because of the implementation of `permissive` policy in the `kube-system` namespace.


# Scenario 2

How to make an exception for one workload to use `permissive` policy while `restrictive` policy has been applied on the namespace?

The above scenario can be resolved by doing steps given below:

- Create a new service account in that namespace:

```bash
sudo kubectl create --namespace=kube-system serviceaccount specialsa"
```

- Create a role binding for the new service account:
```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: specialsa-psp-permissive
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-permissive
subjects:
- kind: ServiceAccount
  name: specialsa
  namespace: default
```


- Create a new deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: kube-system
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
	  serviceAccount: specialsa
```

It can be seen that pod has been created in the kube-system namespace by using this command:

```bash
watch -n 1 sudo kubectl --namespace=kube-system get po,rs,deploy

```

```bash

NAME                                                                 READY   STATUS       RESTARTS   AGE
pod/nginx-hostnetwork-deployment-5f884fc6c4-8kgc6                     1/1   Running          0       130m

NAME                                                                 DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-5f884fc6c4          1         1         1     130m

NAME                                                                READY   UP-TO-DATE   AVAILABLE       AGE
deployment.extensions/nginx-hostnetwork-deployment                   1/1         1          1           130m

```





### Refrences
Link to article [link](https://octetz.com/posts/setting-up-psps)