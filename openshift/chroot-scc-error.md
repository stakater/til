If we face permissions issues when pod is not able to run ```chroot``` command. Most probably ```SecurityContextConstraint``` is blocking that operation.

in order to resolve this, do the following:

*) Get the scc name that your pod is using by typing:

``` 
   oc get po <pod-name> -o yaml | grep scc 
```

*) Edit scc that pod is using.

```
    oc edit scc <scc-name>
```

*) Add following lines in your scc configuration.

```
defaultAddCapabilities:
  - SYS_CHROOT
```

*) Save the configuration.
