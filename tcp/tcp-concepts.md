# TCP working

This document explains how TCP works and how to improve the performance of the TCP connections.


## How TCP works?

TCP client server architecture works in following ways:

1. There are two kinds of connections:
    * Active Open: An active open is the creation of a connection to a listening port by a client. It uses socket() and connect().

    * Passive Open: A passive open is the creation of a listening socket, to accept incoming connections. It uses socket(), bind(), listen(), followed by an accept() loop.

2. On a server, a process listens on a port. Once it gets a connection, it hands it off to another thread. The communication never hogs the listening port. 

3. Connections are uniquely identified by the OS by the following 5-tuple: (local-IP, local-port, remote-IP, remote-port, protocol). If any element in the tuple is different, then this is a completely independent connection.

4. When a client connects to a server, it picks a random, unused high-order source port. This way, a single client can have up to ~64k connections to the server for the same destination port. Connections are mostly limited from the client side due to limited number of ports because the server can handle around 300K connections at most.

5. When a connection closes the connection remain in the TIME_WAIT state which is equal to 2MSL (maximum segment life {it is the time when the TCP segments remains on the network before they are discarded}). There are two main purpose of TIME_WAIT state:

    * It stops packets that are delayed from being accepted by another socket using the same source address, source port, destination address, destination port combination.

    * It ensures the remote end of a connection has closed the connection. This is especially needed when a last ACK packet is lost.

    It is widely recommended that TCP TIME-WAIT state value not be changed. It is normal for sockets to accumulate when a server is opening and closing sockets faster then the TIME-WAIT state will allow the socket's resources to be absorbed back into the system. Besides the potential for new sockets accepting delayed packets from previous connections, this value is a global value. Being a global value, all sockets created after this value is changed will inherit the new value. You could potentially cause some applications to stop working. Configuring the value of the TCP TIME_WAIT state can only be useful when we transfer small amount of data.

## How to improve the TCP connection performace?

There are three variables that can be used to tweak the tcp connections:

1. tcp_tw_reuse: This allows reusing sockets in TIME_WAIT state for new connections when it is safe from protocol viewpoint. Default value is 0 (disabled). It is generally a safer alternative to tcp_tw_recycle

2. tcp_tw_recycle:  It enables fast recycling of TIME_WAIT sockets. The default value is 0 (disabled). The sysctl documentation incorrectly states the default as enabled. It can be changed to 1 (enabled) in many cases. Known to cause some issues with hoststated (load balancing and failover) if enabled, should be used with caution. It has been [removed](https://stackoverflow.com/questions/6426253/tcp-tw-reuse-vs-tcp-tw-recycle-which-to-use-or-both) from linux kernel 4.12.

3. tcp_fin_timeout: The tcp_fin_timeout variable tells kernel how long to keep sockets in the state FIN-WAIT-2 if you were the one closing the socket. This is used if the other peer is broken for some reason and don't close its side, or the other peer may even crash unexpectedly. Each socket left in memory takes approximately 1.5Kb of memory, and hence this may eat a lot of memory if you have a moderate webserver or something alike.

* Follow the guideline to configure the above variable:
```bash

$ sudo su

# to check the value of above variable

$ cat /proc/sys/net/ipv4/tcp_fin_timeout   # default value is 60 seconds
$ cat /proc/sys/net/ipv4/tcp_tw_reuse  # default value is 0
$ cat /proc/sys/net/ipv4/tcp_tw_recycle  # default value is 0

# to change the value of the above variable
$ echo 10 > /proc/sys/net/ipv4/tcp_fin_timeout
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle

# to reload the variable
$ sysctl -p
$ /etc/init.d/network restart # to restart the network on Redhat (RHEL) / CentOS / Fedora / Suse / OpenSuse machine
$ /etc/init.d/networking restart # to restart the network on the Debian/ubuntu machine 
```

*  To check the connections in different states use the command given below or we can use the grafana full node exporter dashboard:

```bash
netstat -ant | awk '{print $6}' | sort | uniq -c | sort -n
```

* Add this query to get the connection in time state.

* To get the sockets used by TCP:

```bash
ss -s
```