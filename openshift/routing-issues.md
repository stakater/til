# Routing Issues

If you are having issues with routes to your application gather the following information.

```
$ oc logs dc/router -n default
$ oc get dc/router -o yaml -n default
$ oc get route <NAME_OF_ROUTE> -n <PROJECT>
$ oc get endpoints --all-namespaces 
$ oc exec -it $ROUTER_POD -- ls -la 
$ oc exec -it $ROUTER_POD -- find /var/lib/haproxy -regex ".*\(.map\|config.*\|.json\)" -print -exec cat {} \; > haproxy_configs_and_maps
```

Use the information above to check that our basic configuration files have the proper routing information and time-stamps. Meaning that updates are happening.
If you do see that the router configs / files are not updated (in some time) and you don't see any "Observed a panic" messages in the logs. Collect an strace of the router process, to look for such messages, that may not have been logged properly.
Note: This assumes you are system:admin and that you are in the default project (it may work with other users who have cluster-admin permissions).

If the pod is serving routes and is reachable via the standard http(s) ports. Then we should verify that the DNS domain names assigned to your applications resolve to that node's address where the router is running. If your application's domains is /apps.example.com/ you can check with dig for an ANSWER containing the nodes IP address.

```
$ dig +norecurse \*.apps.example.com
```

If certificates are not being served out correctly check the logs for errors, make sure the if present that the vip or proxy in front of the routers off loads SSL to the router (SSL Passthrough). To confirm what certificates are being severed out run the following.

```
# echo -n | openssl s_client -connect 192.168.1.5:443 -servername myawesome.apps.example.com 2>&1 | openssl x509 -noout -text
# curl -kv https://myawesome.apps.example.com 
```

Where 192.168.1.5 is the IP of the node where the router is located, and "myawesome.apps.example.com" is the hostname passed to the route.

- [Setup the router to log http request to an rsyslog endpoint](https://docs.openshift.com/container-platform/3.5/admin_guide/router.html#viewing-logs)
