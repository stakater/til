# Openshift Cluster Testing

### Overview

This document have the guideline on how to reproduce the connection timeout issue on the openshift cluster.


### Pre-requisites

Go through the document and links provided in the [til/tcp](../tcp) folder.

### Test Guidelines

Follow these guideline to run the tests on the openshift cluster:

* To get the default value of tcp kernel variables:

```bash
$ sudo su

# to check the value of above variable
$ cat /proc/sys/net/ipv4/tcp_tw_reuse  # default value is 0
$ cat /proc/sys/net/ipv4/tcp_tw_recycle  # default value is 0
```

* Increase the limit of open files so that we don't hit the `too many open files issue`. This issue is caused when a process tries to open too many files

```bash
# to get the default value which in 1024. It will print all other limits
$ ulimit -a

# To increase the limit in the current session
$ ulimit -n 65000
```

* To change the value of these variables:

```bash
# to change the value of the above variable
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse # assign 1 to enable it
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle # assign 1 to enable it 

# to reload the variable
$ sysctl -p
$ /etc/init.d/network restart # to restart the network on Redhat (RHEL) / CentOS / Fedora /suse / OpenSuse machine

$ /etc/init.d/networking restart # to restart the network on the Debian/ubuntu machine 
```

* To install the apache benchmark load test tool use the command given below:

```bash
# to install on debian and ubuntu based systems
$ sudo apt-get install apache2-utils
```

```bash
# redhat system
$ sudo yum install apache2-utils
```

* To use the load test tool use the commands given below:

```
$ ab -n <number of requests> -c <number of concurrent requests> -g graph.data <test url>
```

* To generate a graph on the client side we will use gnuplot tool:

```bash

# installation
$ sudo apt-get install gnuplot  # to install on ubuntu system
$ sudo yum install gnuplot  # to install on the redhat system

# usage
$ gnuplot

$ gnuplot> set terminal dumb

# it will generate a graph on the client side
gnuplot> plot "<name of the datafile used provided to ab>" using 9  w l
```

* To generate graph on the server side we will use the following grafana graphs:

1. Node exporter full.

2. [Grafana Micrometer](https://github.com/making/prometheus-kustomize/blob/master/base/grafana-micrometer.yml).