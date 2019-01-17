# How to install OpenShift Container Platform from Red Hat Satellite 6?

## Environment
- OpenShift Container Platform 3.9
- Red Hat Satellite 6.3

## Issue

- In secured/disconnected environment where all systems are managed by Red Hat Satellite 6, what is the way to install OpenShift Container Platform using Red Hat Satellite 6?
- How to install OpenShift Container Platform from Red Hat Satellite 6?
- Is it possible to install disconnected OpenShift Container Platform using Red Hat Satellite 6?
- Can OpenShift Connect to Red Hat Satellite 6 to download all the Red Hat Docker Images?

## Resolution

1. On Red Hat Satellite 6 Server ensure that valid OpenShift subscription is present in manifest. If not, then attach OpenShift subscription to Satellite profile on customer portal and refresh [manifest](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html/content_management_guide/managing_subscriptions/) on Satellite Server.
2. [Enable](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html/content_management_guide/importing_red_hat_content#Importing_Red_Hat_Content-Selecting_Red_Hat_Repositories_to_Synchronize) following required RPM repositories on Satellite and [sync](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html/content_management_guide/importing_red_hat_content#Importing_Red_Hat_Content-Synchronizing_Red_Hat_Repositories) them.

```
rhel-7-server-rpms
rhel-7-server-extras-rpms
rhel-7-server-ose-3.9-rpms
rhel-7-fast-datapath-rpms
rhel-7-server-ansible-2.4-rpms
```

From Satellite web UI, enable following repositories and sync.

```
RPMs

Red Hat Enterprise Linux Server ->  Red Hat Enterprise Linux 7 Server - Extras (RPMs) -> Red Hat Enterprise Linux 7 Server - Extras RPMs x86_64 

Red Hat Enterprise Linux Server -> Red Hat Enterprise Linux 7 Server - Optional (RPMs) -> Red Hat Enterprise Linux 7 Server - Optional RPMs x86_64 7Server 

Red Hat OpenShift Container Platform -> Red Hat OpenShift Container Platform 3.9 (RPMs) ->  Red Hat OpenShift Container Platform 3.9 RPMs x86_64 

Red Hat Enterprise Linux Server -> Red Hat Enterprise Linux 7 Server (RPMs) ->  Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server 

Red Hat Gluster Storage Server for On-premise -> Red Hat Gluster Storage 3 Server (RPMs) -> Red Hat Gluster Storage 3 Server RPMs x86_64 7Server

Red Hat Enterprise Linux Fast Datapath -> Red Hat Enterprise Linux Fast Datapath (RHEL 7 Server) (RPMs) -> Red Hat Enterprise Linux Fast Datapath RHEL 7 Server RPMs x86_64 7Server 

Other

Red Hat Ansible Engine ->  Red Hat Ansible Engine 2.4 RPMs for Red Hat Enterprise Linux 7 Server ->  Red Hat Ansible Engine 2.4 RPMs for Red Hat Enterprise Linux 7 Server x86_64 
```

3. Create [Content View](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html/content_management_guide/managing_content_views) on Satellite Server or add required repositories to existing content view and publish/promote the content view in order to make the content available for OCP cluster systems.
4. Ensure that all OCP cluster system met [prerequisites](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#install-config-install-prerequisites).
5. [Register](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html-single/managing_hosts/#sect-Red_Hat_Satellite-Managing_Hosts-Registration-Registering_a_Host) your OCP cluster systems with Red Hat Satellite 6.
6. [Subscribe](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#host-registration) OCP cluster system with valid OpenShift subscription and enable required repositories on them.
7. Install [required packages](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#installing-base-packages). Also install [Docker](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#installing-docker) package.
8. Configure [Docker storage](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#configuring-docker-storage) on OCP systems.
9. [Ensure SSH host access](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#ensuring-host-access) for all OCP systems.
10. On Satellite Server, create [custom product](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html-single/content_management_guide/#Importing_Custom_Content-Creating_a_Custom_Product) for Red Hat Container Catalog.
11. [Create repositories](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.3/html-single/content_management_guide/#importing_container_images_from_the_red_hat_container_catalog) in custom product that links to the Red Hat Container Catalog registry (http://registry.access.redhat.com/). Then Synchronize with the registry’s repository.

For example:

```
Content --> Product --> Select custom product created for container catalog registry --> Create Repository -->
Name: ose-ansible
Label: ose-ansible
Type: docker
URL: http://registry.access.redhat.com/
Upstream Repository Name: openshift3/ose-ansible
Click Save
Content --> Product --> Custom Product --> check mark repositories to sync --> Sync Now
```

```
Following are the required repositories for OCP installation.
openshift3/ose-ansible
openshift3/ose-web-console
openshift3/ose-cluster-capacity
openshift3/ose-deployer
openshift3/ose-docker-builder
openshift3/oauth-proxy
openshift3/ose-docker-registry
openshift3/ose-egress-http-proxy
openshift3/ose-egress-router
openshift3/ose-f5-router
openshift3/ose-haproxy-router
openshift3/ose-keepalived-ipfailover
openshift3/ose-pod
openshift3/ose-sti-builder
openshift3/ose
openshift3/container-engine
openshift3/node
openshift3/openvswitch
rhel7/etcd
openshift3/ose-recycler
```

```
For the additional centralized log aggregation and metrics aggregation components:
openshift3/logging-auth-proxy
openshift3/logging-curator
openshift3/logging-elasticsearch
openshift3/logging-fluentd
openshift3/logging-kibana
openshift3/metrics-cassandra
openshift3/metrics-hawkular-metrics
openshift3/metrics-hawkular-openshift-agent
openshift3/metrics-heapster
openshift3/prometheus
openshift3/prometheus-alert-buffer
openshift3/prometheus-alertmanager
openshift3/prometheus-node-exporter
cloudforms46/cfme-openshift-postgresql
cloudforms46/cfme-openshift-memcached
cloudforms46/cfme-openshift-app-ui
cloudforms46/cfme-openshift-app
cloudforms46/cfme-openshift-embedded-ansible
cloudforms46/cfme-openshift-httpd
cloudforms46/cfme-httpd-configmap-generator
rhgs3/rhgs-server-rhel7
rhgs3/rhgs-volmanager-rhel7
rhgs3/rhgs-gluster-block-prov-rhel7
rhgs3/rhgs-s3-server-rhel7
```

```
For the service catalog, openshift Ansible broker, and template service broker features:
openshift3/ose-service-catalog
openshift3/ose-ansible-service-broker
openshift3/mediawiki-apb
openshift3/postgresql-apb
```

12. On OCP cluster systems disable default registries and allow Satellite 6 and any other internal registry in docker configuration file /etc/sysconfig/docker

For example:

```
BLOCK_REGISTRY='--block-registry registry.access.redhat.com --block-registry docker.io'
ADD_REGISTRY='--add-registry satellite.example.com:5000'
INSECURE_REGISTRY='--insecure-registry 172.30.0.0/16 --insecure-registry satellite.example.com:5000'
```

Restart docker service after making changes to docker configuration file.

```
# systemctl restart docker.service
```

NOTE: Ensure TCP port 5000 is open on Red Hat Satellite 6 Server.

13. [Create/edit inventory file](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#configuring-ansible) and add following variables in it to fetch required images from Satellite for OCP installation.

```
[OSEv3:vars]
openshift_docker_additional_registries=satellite.example.com:5000 
openshift_docker_insecure_registries=satellite.example.com:5000 
openshift_docker_blocked_registries=registry.access.redhat.com,docker.io 
oreg_url=satellite.example.com:5000/organization-containers-ose-${component}:${version} 
openshift_examples_modify_imagestreams=true
openshift_cockpit_deployer_prefix=satellite.example.com:5000/organization-containers-
openshift_cockpit_deployer_version=latest
openshift_web_console_prefix=satellite.example.com:5000/organization-containers-ose- 
openshift_web_console_version=latest
```

For metrics, logging, prometheus and service-broker use following variables.

```
openshift_metrics_image_prefix=satellite.example.com:5000/organization-containers- 
openshift_metrics_image_version=latest 
openshift_logging_image_prefix=satellite.example.com:5000/organization-containers- 
openshift_logging_image_version=latest 
ansible_service_broker_image_prefix=satellite.example.com:5000/organization-containers- 
ansible_service_broker_image_tag=latest
ansible_service_broker_etcd_image_prefix=satellite.example.com:5000/organization-containers- 
ansible_service_broker_etcd_image_tag=latest 
openshift_service_catalog_image_prefix=satellite.example.com:5000/organization-containers- 
openshift_service_catalog_image_version=latest 
template_service_broker_prefix=satellite.example.com:5000/organization-containers- 
template_service_broker_version=latest
```

For additional variables refer example [inventory file](https://github.com/openshift/openshift-ansible/blob/release-3.9/inventory/hosts.example).

NOTE:

- Replace satellite.example.com with your Satellite Server's FQDN
- Replace 'organization' with actual organization name under which the product and image repositories are created
- Replace version 'latest' to desired image version
- To get exact repository url go to custom product on Satellite 6, then click on 'Repositories'. Click on the image repository and check 'Published At'.

14. Once inventory file is created, continue with [installation](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#running-the-advanced-installation).

```
# ansible-playbook [-i /path/to/inventory] \
/usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml

# ansible-playbook [-i /path/to/inventory] \
/usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml
```

Once installation completes [verify the installation](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.9/html-single/installation_and_configuration/#advanced-verifying-the-installation).

## Reference

- https://access.redhat.com/solutions/3435901
