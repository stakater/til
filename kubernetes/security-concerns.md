# Security Concerns

## 1. System separation
- Communication in to the cluster
We have to have system separation, which means the only specific consumers are allowed to access a specific application in the cluster.
 
- Communication out from the cluster
We have to have system separation, which means that applications (executing in the cluster) only are allowed specific access to applications outside the cluster. This also means not all applications are allowed to have the same access to systems outside the cluster.
 
## 2. Traceability (on IP level?)
It should be possible to trace where traffic comes from, both in to the cluster and out from the cluster.
 
## 3. Access protection
An application is responsible for access protection (requiring authentication of some kind).
 
## 4. Someone breaking out of the container

> How do we make sure that for instance internet traffic does not violate security for other applications? It can be gaining sensitive data, using resources so that other applications can’t run properly etc.

- https://medium.com/@chrismessiah/docker-and-kubernetes-in-high-security-environments-d851645e8b99
- https://www.youtube.com/watch?v=GLwmJh-j3rs&feature=youtu.be&t=14s

> Simply put, containers are just processes, and as such they are governed by the kernel like any other process. Thus any kernel-land vulnerability which yields arbitrary code execution can be exploited to escape a container.
