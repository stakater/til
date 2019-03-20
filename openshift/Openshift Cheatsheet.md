# My Openshift Cheatsheet

### Examine the **cluster** quota defined for the environment:

```
$ oc describe AppliedClusterResourceQuota
```

### Install pkgs using yum in a Dockerfile

```
# Install Runtime Environment
RUN set -x && \ 2
    yum clean all && \
    REPOLIST=rhel-7-server-rpms,rhel-7-server-optional-rpms,rhel-7-server-thirdparty-oracle-java-rpms \
    INSTALL_PKGS="tar java-1.8.0-oracle-devel" && \
    yum -y update-minimal --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs \
      --security --sec-severity=Important --sec-severity=Critical && \
    yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    yum clean all
```

### Creates a service to point to an external service addr (DNS or IP)

```
oc create service externalname myservice \
    --external-name myhost.example.com
```

> A typical service creates endpoint resources dynamically, based on the selector attribute of the service. The oc status and oc get all commands do not display these resources. You can use the oc get endpoints command to display them.

> If you use the oc create service externalname --external-name command to create a service, the command also creates an endpoint resource that points to the host name or IP address given as argument.

> If you do not use the --external-name option, it does not create an endpoint resource. In this case, you need to use the oc create -f command and a resource definition file to explicitly create the endpoint resources.

> If you create an endpoint from a file, you can define multiple IP addresses for the same external service, and rely on the OpenShift service load-balancing features. In this scenario, OpenShift does not add or remove addresses to account for the availability of each instance. An external application needs to update the list of IP addresses in the endpoint resource.

### Patching a DeploymentConfig from the CLI

 * this example removes an config attribute using JSON path
```
oc patch dc/mysql --type=json \
    -p='[{"op":"remove", "path": "/spec/strategy/rollingParams"}]'
```

* this example cnhage an existing attribute value using JSON format
```
oc patch dc/mysql --patch \
    '{"spec":{"strategy":{"type":"Recreate"}}}'
```

### Creating a Custom template by exporting existing resources
> The oc export command can create a resource definition file by using the --as-template option. Without the --as-template option, the oc export command only generates a list of resources. With the --as-template option, the oc export command wraps the list inside a template resource definition. After you export a set of resources to a template file, you can add annotations and parameters as desired.

> The order in which you list the resources in the oc export command is important. You need to export dependent resources first, and then the resources that depend on them. For example, you need to export image streams before the build configurations and deployment configurations that reference those image streams.

```
oc export is,bc,dc,svc,route --as-template > mytemplate.yml
```

> Depending on your needs, add more resource types to the previous command. For example, add secret before bc and dc. It is safe to add pvc to the end of the list of resource types because a deployment waits for persistent volume claim to bind.

> The oc export command does not generate resource definitions that are ready to use in a template. These resource definitions contain runtime information that is not needed in a template, and some of it could prevent the template from working at all. Examples of runtime information are attributes such as status, creationTimeStamp, image, and tags, besides most annotations that start with the openshift.io/generated-by prefix.

> Some resource types, such as secrets, require special handling. It is not possible to initialize key values inside the data attribute using template parameters. The data attribute from a secret resource needs to be replaced by the stringData attribute and all key values need to be unencoded.

### Logging Aggregation throubleshooting
 * https://access.redhat.com/articles/3136551

### Process a template, create a new binary build to customize something and them change the DeploymentConfig to use the new Image...

```
oc process openshift//datagrid72-basic | oc create -f -

oc new-build --name=customdg -i openshift/jboss-datagrid72-openshift:1.0 --binary=true --to='customdg:1.0'
oc set triggers dc/datagrid-app --from-image=openshift/jboss-datagrid72-openshift:1.0 --remove
oc set triggers dc/datagrid-app --from-image=customdg:1.0 -c datagrid-app

```

#### List only paramaters of a given template file definition

```
oc process -f mytemplate.yaml --parameters
``` 

### Copy file content from a specific image to local file system

```
docker run registry.access.redhat.com/jboss-datagrid-7/datagrid72-openshift:1.0 /bin/sh -c 'cat /opt/datagrid/standalone/configuration/clustered-openshift.xml' > clustered-openshift.xml
```

### set the default storage-class
```
oc patch storageclass glusterfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Change Default response timeout for a specific route:
```
oc annotate route <route_name> --overwrite haproxy.router.openshift.io/timeout=10s
```

### Add a nodeSelector on RC ou DC
```
oc patch dc|rc <dc_name> -p "spec:                                                                                         
  template:     
    spec:
      nodeSelector:
        region: infra"
```

### Binary Builds
```
oc new-build --binary=true --name=ola2 --image-stream=redhat-openjdk18-openshift --to='mycustom-jdk8:1.0'
oc start-build ola2 --from-file=./target/ola.jar --follow
oc new-app 
```

### Turn off/on DC triggers to do a batch of changes without spam many deployments
```
oc rollout pause dc <dc name>
oc rollout resume dc <dc name> 
```
### get a route URL using OC
```
http://$(oc get route nexus3 --template='{{ .spec.host }}')
```

### Using Nexus repo manager to store deployment artifacts
Maven uses settings.xml in $HOME/.m2 for configuration outside of pom.xml:

```xml
<?xml version="1.0"?>
<settings>
  <mirrors>
    <mirror>
      <id>Nexus</id>
      <name>Nexus Public Mirror</name>
      <url>http://nexus-opentlc-shared.cloudapps.na.openshift.opentlc.com/content/groups/public/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
  <servers>
  <server>
    <id>nexus</id>
    <username>admin</username>
    <password>admin123</password>
  </server>
</servers>
</settings>
```

Maven can automatically store artifacts using -DaltDeploymentRepository parameter for deploy task:

```
mvn deploy -DskipTests=true \
-DaltDeploymentRepository= nexus::default::http://nexus3.nexus.svc.cluster.local:8081/repository/releases
```

### to update a DeploymentConfig in order to change the Docker Image used by a specific container
```
oc project <project>
oc get is

# creates an ImageStream from a Remote Docker Registry image
oc import-image <image name> --from=docker.io/<imagerepo>/<imagename> --all --confirm

oc get istag

OC_EDITOR="vim" oc edit dc/<your_dc>

    spec:
      containers:
      - image: docker.io/openshiftdemos/gogs@sha256:<the new image digest from Image Stream>
        imagePullPolicy: Always

```

### BuildConfig with Source pull secrets
```
oc secrets new-basicauth gogs-basicauth --username=<your gogs login> --password=<gogs pwd>
oc set build-secret --source bc/tasks gogs-basicauth
```

### Adding a volume in a given DeploymentConfig

```
oc set volume dc/myAppDC --add --overwrite --name....
``` 

### Create a configmap file and mount as a volume on DC
```
oc create configmap myconfigfile --from-file=./configfile.txt
oc set volumes dc/printenv --add --overwrite=true --name=config-volume --mount-path=/data -t configmap --configmap-name=myconfigfile
```

### create a secret via CLI
```
oc create secret generic mysec --from-literal=app_user=superuser --from-literal=app_password=topsecret
oc env dc/printenv --from=secret/mysec
oc set volume dc/printenv --add --name=db-config-volume --mount-path=/dbconfig --secret-name=printenv-db-secret
```

### Configure Liveness/Readiness probes on DCs
```
oc set probe dc cotd1 --liveness -- echo ok
oc set probe dc/cotd1 --readiness --get-url=http://:8080/index.php --initial-delay-seconds=2 
```

### Create a new JOB
```
oc run pi --image=perl --replicas=1  --restart=OnFailure \
    --command -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

### CRON JOB
```
oc run pi --image=perl --schedule='*/1 * * * *' \
    --restart=OnFailure --labels parent="cronjobpi" \
    --command -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

### A/B Deployments - Split route trafic between services

```
oc expose service cotd1 --name='abcotd' -l name='cotd'
oc set route-backends abcotd --adjust cotd2=+20%
oc set route-backends abcotd cotd1=50 cotd2=50
```

### to pull an image directly from red hat offcial docker registry
```
docker pull registry.access.redhat.com/jboss-eap-6/eap64-openshift
```

### to validate a openshift/kubernates resource definition (json/yaml file)  in order to find malformed/sintax problems
```
oc create --dry-run --validate -f openshift/template/tomcat6-docker-buildconfig.yaml
```

* to prune old objects
 * https://docs.openshift.com/container-platform/3.3/admin_guide/pruning_resources.html

* to enable cluster GC
 * https://docs.openshift.com/container-platform/3.3/admin_guide/garbage_collection.html

### to get current user Barear Auth Token

```
oc whoami -t
```

### to test Master API 

```
curl -k -H "Authorization: Bearer <api_token>" https://<master_host>:8443/api/v1/namespaces/<projcet_name>/pods/https:<pod_name>:8778/proxy/jolokia/

# get pod memory via jmx
curl -k -H "Authorization: Bearer <api_token>" https://<master_host>:8443/api/v1/namespaces/<projcet_name>/pods/https:<pod_name>:8778/proxy/jolokia//read/java.lang:type\=Memory/HeapMemoryUsage | jq .
```

### to login via CLI `oc`
```
oc login --username=tuelho --insecure-skip-tls-verify --server=https://master00-${guid}.oslab.opentlc.com:8443

### to login as Cluster Admin through master host
oc login -u system:admin -n openshift
```

### to view the cluster roles and their associated rule sets in the cluster policy
```
oc describe clusterPolicy default
```

### add a role to user
```
#local binding
oadm policy add-role-to-user <role> <username>

#cluster biding
oadm policy add-cluster-role-to-user <role> <username>
```

### allow containers run with root user inside openshift
```
oadm policy add-scc-to-user anyuid -z default
```

> for more details consult: https://docs.openshift.com/enterprise/3.1/admin_guide/manage_authorization_policy.html

### to test a POD service locally
```
ip=`oc describe pod hello-openshift|grep IP:|awk '{print $2}'`
curl http://${ip}:8080
```

### to access a POD container shell
```
oc exec -ti  `oc get pods |  awk '/registry/ { print $1; }'` /bin/bash

#new way to do the same:
oc rsh <container-name>
```

### to edit an object/resource
```
oc edit <object_type>/<object_name>

#eg

oc edit dc/myDeploymentConfig
```

### Ataching a new `PersistentVolumeClaim` to a `DeploymentConfig`

```
oc volume dc/docker-registry \
   --add --overwrite \
   -t persistentVolumeClaim \
   --claim-name=registry-claim \
   --name=registry-storage
```

### Docker builder app creation
 
 ```
 oc new-app --docker-image=openshift/hello-openshift:v1.0.6 -l "todelete=yes"
 ```
 
### To create an app using a template (`eap64-basic-s2i`): Ticketmonster demo

```
oc new-app javaee6-demo
oc new-app --template=eap64-basic-s2i -p=APPLICATION_NAME=ticketmonster,SOURCE_REPOSITORY_URL=https://github.com/jboss-developer/ticket-monster,SOURCE_REPOSITORY_REF=2.7.0.Final,CONTEXT_DIR=demo
```

### STI app creation

```
oc new-app https://github.com/openshift/sinatra-example -l "todelete=yes"
oc new-app openshift/php~https://github.com/openshift/sti-php -l "todelete=yes"
```

###  To watch a build process log

```
oc get builds
oc logs -f builds/sti-php-1
```

### To create application using Git repository at current directory:

```
$ oc new-app
```

### To create application using remote Git repository and context subdirectory:

```
$ oc new-app https://github.com/openshift/sti-ruby.git \
    --context-dir=2.0/test/puma-test-app
```

### To create application using remote Git repository with specific branch reference:
```
$ oc new-app https://github.com/openshift/ruby-hello-world.git#beta4
```

> New App From Source Code
> 
>  Build Strategy Detection
> 
>  If new-app finds a Dockerfile in the repository, it uses docker build strategy Otherwise, new-app uses source strategy
>   
>  To specify strategy, set `--strategy flag` to source or docker
>  Example: To force new-app to use docker strategy for local source repository:
  
  ```
  $ oc new-app /home/user/code/myapp --strategy=docker
  ```

### to create a definition generated by `oc new-app` command based on S2I support

```
$ oc new-app https://github.com/openshift/simple-openshift-sinatra-sti.git -o json | \
   tee ~/simple-sinatra.json
```

### To create application from MySQL image in Docker Hub:

```
$ oc new-app mysql
```

### To create application from local registry:

```
$ oc new-app myregistry:5000/example/myimage
```

> If the registry that the image comes from is not secured with SSL, cluster administrators must ensure that the Docker daemon on the OpenShift Enterprise nodes is run with the --insecure-registry flag pointing to that registry. You must also use the `--insecure-registry=true` flag to tell new-app that the image comes from an insecure registry.

### To create application from stored template:

```
$ oc create -f examples/sample-app/application-template-stibuild.json
$ oc new-app ruby-helloworld-sample
```

### To set environment variables when creating application for database image:

```
$ oc new-app openshift/postgresql-92-centos7 \
    -e POSTGRESQL_USER=user \
    -e POSTGRESQL_DATABASE=db \
    -e POSTGRESQL_PASSWORD=password
```

### To output new-app artifacts to file, edit them, then create them using oc create:

```
$ oc new-app https://github.com/openshift/ruby-hello-world -o json > myapp.json
$ vi myapp.json
$ oc create -f myapp.json
```

* To deploy two images in single pod:

```
$ oc new-app nginx+mysql
```

### To deploy together image built from source and external image:

```
$ oc new-app \
    ruby~https://github.com/openshift/ruby-hello-world \
    mysql \
    --group=ruby+mysql
```

### to export all the project's objects/resources as a single template:

```
$ oc export all --as-template=<template_name>
```

> You can also substitute a particular resource type or multiple resources instead of all. Run $ oc export -h for more examples

* to create a new project using `oadm` and defining an admin user
```
$ oadm new-project instant-app --display-name="instant app example project" \
    --description='A demonstration of an instant-app/template' \
    --node-selector='region=primary' --admin=andrew
```

### to create an app using `oc` CLI based on a `template`
```
$ oc new-app --template=mysql-ephemeral --param=MYSQL_USER=mysqluser,MYSQL_PASSWORD=redhat,MYSQL_DATABASE=mydb,DATABASE_SERVICE_NAME=database
```

### to see a list of `env` `vars` defined in a DeploymentConfig object
```
$ oc env dc database --list
# deploymentconfigs database, container mysql
MYSQL_USER=***
MYSQL_PASSWORD=***
MYSQL_DATABASE=***

```

### to manage enviorenmet variables in different ose objects types.

The first adds, with value /data. The second updates, with value /opt.

```
$ oc env dc/registry STORAGE=/data
$ oc env dc/registry --overwrite STORAGE=/opt
```

To unset environment variables in the pod templates:

```
$ oc env <object-selection> KEY_1- ... KEY_N- [<common-options>]
```

> The trailing hyphen (-, U+2D) is required.

This example removes environment variables ENV1 and ENV2 from deployment config d1:
```
$ oc env dc/d1 ENV1- ENV2-
```

This removes environment variable ENV from all replication controllers:
```
$ oc env rc --all ENV-
```

This removes environment variable ENV from container c1 for replication controller r1:

To list environment variables in pods or pod templates:
```
$ oc env rc r1 --containers='c1' ENV-
```

This example lists all environment variables for pod p1:
```
$ oc env <object-selection> --list [<common-options>]
```

```
$ oc env pod/p1 --list
```

### to apply some change (patch)

```
oc patch dc/<dc_name> \
   -p '{"spec":{"template":{"spec":{"nodeSelector":{"nodeLabel":"logging-es-node-1"}}}}}'
```

### to apply a vlome storage

```
oc volume dc/<dc_name> \
          --add --overwrite --name=<volume_name> \
          --type=persistentVolumeClaim --claim-name=<claim_name>
```

### to make a node unschedulable in a cluster

```
oadm manage node <nome do node > --schedulable=false
```

### to create a registry with storage-volume mounted on host
```
oadm registry --service-account=registry \
    --config=/etc/origin/master/admin.kubeconfig \
    --credentials=/etc/origin/master/openshift-registry.kubeconfig \
    --images='registry.access.redhat.com/openshift3/ose-${component}:${version}' \
    --mount-host=<path> --selector=meuselector
```

### to export all resources from a project/namespace as a template
```
oc export all --as-template=<template_name>
```

### to create a build from a Dockerfile

```
# create the build
cat ./path/to/your/Dockerfile | oc new-build --name=build-from-docker --binary --strategy=docker -l app=app-from-custom-docker-build  -D -

#if you need to give some input to your Docker Build process
oc start-build build-from-docker --from-dir=. --follow

# create an OSE app from the docker build image
oc new-app app-from-custom-docker-build -l app=app-from-custom-docker-build

oc expose service app-from-custom-docker-build
```

### to copy files to/from a POD

```
#Ref: https://docs.openshift.org/latest/dev_guide/copy_files_to_container.html

oc rsync /home/user/source devpod1234:/src

oc rsync devpod1234:/src /home/user/source
```

## Cluster nodes CleanUp

1. Desliga todos os containers que vc não tá usando no seu ambiente do openshift
2. Executa em todos os nodes e master o comando: docker rm $(docker ps -a -q)
3. Remove todas as imagens de todos os nodes e master. Para isso loga em cada uma delas via ssh e remove as imagens usando docker rmi <id da image>. Pega as imagens que começa com o ip do registry 172.30...
4. configurar GC: https://docs.openshift.com/enterprise/3.1/admin_guide/garbage_collection.html

##Tips

* internal DNS name of ose/kubernetes services
 * follows the pattern `<service-name>.<project>.svc.cluster.local`


 Object Type	 | Example 
--------------- | ----------------------------------------------
 Default        | <pod_namespace>.cluster.local 
 Services       | <service>.<pod_namespace>.svc.cluster.local
 Endpoints      | <name>.<namespace>.endpoints.cluster.local
 
 > "he only caveat to this, is that if we are using the multi-tenant OVS networking plugin, our cluster administrators will have to make visible our ci project to all other projects:" Ref: https://blog.openshift.com/improving-build-time-java-builds-openshift/
```
$ oadm pod-network make-projects-global ci
```

### Adjust Master Log Level
To adjust openshift-master log level, edit following line of `/etc/sysconfig/atomic-openshift-master` from master VM:

```
OPTIONS=--loglevel=4
```

To make changes valid, restart atomic-openshift-master service:

```
$ sudo -i systemctl restart atomic-openshift-master.service
```

In node machine, to provide filtered information:
```
# journalctl -f -u atomic-openshift-node
```

### Enable EAP clustering/replication

Make sure that your default service account has sufficient privileges to communicate with the Kubernetes REST API.
Add the view role to serviceaccount for the project:

```
$ oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default
```
Examine the first entry in the log file:

```
Service account has sufficient permissions to view pods in kubernetes (HTTP 200). Clustering will be available.
```

## OCP Internal VIP failover for Routers running on Infra nodes

```
oc adm ipfailover ipf-ha-router 
    --replicas=2 --watch-port=80 \
    --selector="region=infra" \
    --virtual-ips="x.0.0.x" \
    --iptables-chain="INPUT" \
    --service-account=ipfailover --create
```

## Creating a new Template
 * Common strategy for building template definitions:
  * Use oc new-app and oc expose to manually create resources application needs
  * Test to make sure resources work as expected
  * Use oc export with -o json option to export existing resource definitions
  * Merge resource definitions into template definition file
  * Add parameters
  * Test resource definition in another project
  > JSON syntax errors are not easy to identify, and OpenShift is sensitive to them, refusing JSON files that most browsers would accept as valid. Use jsonlint -s from the python-demjson package, available from EPEL, to identify syntax issues in a JSON resource definition file.
 
 * Use `oc new-app` with `-o json` option to bootstrap your new template definition file
 ```
oc new-app -o json openshift/hello-openshift > hello.json 
 ```
 
 * Converting the Resource Definition to a Template
  * Change kind attribute from List to Template
  * Make two changes to metadata object:
  * Add name attribute and value so template has name users can refer to
  * Add annotations containing description attribute for template, so users know what template is supposed to do.
  * Rename items array attribute as objects 

## Working with Templates
 * to list all parameters from mysql-persistent template:
```
$ oc process --parameters=true -n openshift mysql-persistent
```
 * Customizing resources from a preexisting Template
 
Example:
```
$ oc export -o json
	-n openshift mysql-ephemeral > mysql-ephemeral.json
... change the mysql-ephemeral.json file ...
$ oc process -f mysql-ephemeral.json \
	-v MYSQL_DATABASE=testdb,MYSQL_USE=testuser,MYSQL_PASSWORD=
	> testdb.json
$ oc create -f testdb.json
```

> oc process uses the -v option to provide parameter values, while oc new-app command uses the -p option.

### Create Definition Files for Volumes

```
ssh master00-$guid
mkdir /root/pvs
```

```
export volsize="5Gi"
for volume in pv{1..25}; \
do \
cat << EOF > /root/pvs/${volume}.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ${volume} 
spec:
  capacity:
    storage: ${volsize} 
  accessModes:
  - ReadWriteOnce 
  nfs: 
    path: /var/export/pvs/${volume} 
    server: 192.168.0.254 
  persistentVolumeReclaimPolicy: Recycle 
EOF
     echo "Created def file for ${volume}"; \
done
```

```
export volsize="10Gi"
for volume in pv{26..50}; \
do \
cat << EOF > /root/pvs/${volume}.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ${volume} 
spec:
  capacity:
    storage: ${volsize} 
  accessModes:
  - ReadWriteOnce 
  nfs: 
    path: /var/export/pvs/${volume} 
    server: 192.168.0.254 
  persistentVolumeReclaimPolicy: Recycle 
EOF
     echo "Created def file for ${volume}"; \
done
```

```
export volsize="1Gi"
for volume in pv{51..100}; \
do \
cat << EOF > /root/pvs/${volume}.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ${volume} 
spec:
  capacity:
    storage: ${volsize} 
  accessModes:
  - ReadWriteOnce 
  nfs: 
    path: /var/export/pvs/${volume} 
    server: 192.168.0.254 
  persistentVolumeReclaimPolicy: Recycle 
EOF
     echo "Created def file for ${volume}"; \
done
```

### Patch PVs definitions

```
for pv in $(oc get pv|awk '{print $1}' | grep pv | grep -v NAME); do oc patch pv $pv -p "spec:      
  accessModes:
  - ReadWriteMany
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Recycle"
```
