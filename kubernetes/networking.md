# Networking

## dnsmasq

dnsmasq  is  a  lightweight  DNS,  TFTP,  PXE, router advertisement and DHCP server. It is intended to provide coupled DNS and DHCP service to a LAN.
       
dnsmasq can also be configured to cache DNS queries for improved DNS lookup speeds to previously visited sites.

Dnsmasq accepts DNS queries and either answers them from a small, local, cache or forwards them  to  a real, recursive, DNS server. It loads the contents of /etc/hosts so that local hostnames which do not appear in the global DNS can  be  resolved  and  also  answers  DNS queries for DHCP configured hosts. It can also act as the authoritative DNS server for one or more domains, allowing local names to appear in the global DNS. It can be configured to do DNSSEC validation.

dnsmasq make dynamic entry in /etc/resolv.conf

### What is DHCP?

DHCP (Dynamic Host Configuration Protocol) is a protocol used to provide quick, automatic, and central management for the distribution of IP addresses within a network.

DHCP is also used to configure the proper subnet mask, default gateway, and DNS server information on the device.

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

## What is Network Manager?

Network Manager aims for Network Connectivity which "Just Works". The computer should use the wired network connection when it's plugged in, but automatically switch to a wireless connection when the user unplugs it and walks away from the desk. Likewise, when the user plugs the computer back in, the computer should switch back to the wired connection. The user should, most times, not even notice that their connection has been managed for them; they should simply see uninterrupted network connectivity.

## A vs AAAA record?

An A and AAAA record are actually primary DNS records.  They associate a domain name with a specific IP address, so that when a user types in a web address, such as "www.rackspace.com" their browser knows where to go for the actual website.

The difference between A and AAAA is this: A is IPv4 and AAAA is the current IPv6 record.  Beyond that, there is no difference.  Later on, IPv6 records will be changed to A6 records.

## Networking Layers

Network connectivity has two main networking layers: 
- the cluster network for node interaction, and 
- the software defined network (SDN) for pod interaction.
