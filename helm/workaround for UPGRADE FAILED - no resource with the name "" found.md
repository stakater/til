# Workaround fix for UPGRADE FAILED: No resource with name "" found

1. Run helm list and find out latest revision for affected chart

```
NAME        REVISION UPDATED                  STATUS  CHART              NAMESPACE
fetlife-web 381      Thu Mar 15 19:46:00 2018 FAILED  fetlife-web-0.1.0  default
```

2. Go from there and find latest revision with DEPLOYED state

```
kubectl -n kube-system edit cm fetlife-web.v381
kubectl -n kube-system edit cm fetlife-web.v380
kubectl -n kube-system edit cm fetlife-web.v379
kubectl -n kube-system edit cm fetlife-web.v378
```

3. Once you find last `DEPLOYED` revision, change its state from `DEPLOYED` to `SUPERSEDED` and save the file

4. Try to do `helm upgrade` again, if it's successful then you are done!

5. If you encounter upgrade error like this:

```
Error: UPGRADE FAILED: "fetlife-web" has no deployed releases
```

then edit the status for very last revision from `FAILED` to `DEPLOYED`

```
kubectl -n kube-system edit cm fetlife-web.v381
```

6. Try to do `helm upgrade` again, if it fails again just flip the table...
