# Networking

## dnsmasq

dnsmasq can also be configured to cache DNS queries for improved DNS lookup speeds to previously visited sites.

## What is IP Masquerade?

IP Masquerade is a networking function in Linux similar to the one-to-many (1:Many) NAT (Network Address Translation) servers found in many commercial firewalls and network routers. For example, if a Linux host is connected to the Internet via PPP, Ethernet, etc., the IP Masquerade feature allows other "internal" computers connected to this Linux box (via PPP, Ethernet, etc.) to also reach the Internet as well. Linux IP Masquerading allows for this functionality even though these internal machines don't have an officially assigned IP address.

MASQ allows a set of machines to invisibly access the Internet via the MASQ gateway. To other machines on the Internet, the outgoing traffic will appear to be from the IP MASQ Linux server itself. In addition to the added functionality, IP Masquerade provides the foundation to create a HEAVILY secured networking environment. With a well built firewall, breaking the security of a well configured masquerading system and internal LAN should be considerably difficult to accomplish.

Masquerading rules are set by several components in k8s i.e.:

- kubelet
- weave
- kube-proxy
- docker

Interesting to know that:

IP MASQ == PROXY == NAT Gateway

Nice read: https://www.tldp.org/HOWTO/IP-Masquerade-HOWTO/what-is-masq.html
