# What is need of internal dns?

In order to answer this question, I am going to use a simple example. When it comes to communication between applications in 
the VM world (e.g. Red Hat Virtualization, VMware,), each application uses the IP address of the guest, which typically 
does not change. However, in the container-centric world, the container gets a new IP address whenever it spins up.

Kubernetes, which orchestrates containers, needs a static IP for the container. The Kubernetes Service Object, which provides 
a Service IP, was created to do this, among other things. The Service Object has a hostname and an IP address. The hostname 
will not change, but the IP address will change if the Service Object is recreated. With this Service hostname, stable 
internal communication between pods (services) becomes possible.

Because the Service IP can change, we don’t want to specify a static IP address in the application environment. What if the 
Service Object is recreated without the application team being notified? A new Service IP will be assigned to the service 
and the applications that use the Service IP will fail.

For these reasons, we normally use the Service hostname to avoid the use of a hardcoded IP address in the application. The 
internal DNS in k8s provides this functionality. It uses dynamic DNS, so whenever the Service Object is recreated, the DNS 
will be updated with new records. With this feature, each Service Object hostname has a unique and dynamic Service IP address. 
As a result, we recommend using the hostname to communicate between internal services.
