# Troubleshoot wrong dependency path in jenkins pipeline

If you are facing this type of error

```text
cd /go/src/github.com/Stakater/IngressMonitorController
cannot find package "github.com/stakater/IngressMonitorController/pkg/config" in any of:
	/go/src/github.com/Stakater/IngressMonitorController/vendor/github.com/stakater/IngressMonitorController/pkg/config (vendor tree)
	/usr/local/go/src/github.com/stakater/IngressMonitorController/pkg/config (from $GOROOT)
	/go/src/github.com/stakater/IngressMonitorController/pkg/config (from $GOPATH)
```

Check the value of owner in your jenkins organization. Sometimes value of owner of jenkins organization is wrong/different which can cause wrong installation path of tool. For example in above case, the missing package `github.com/stakater/IngressMonitorController/pkg/config` has `stakater` in small case in its path while golang is trying to find this dependency in `/go/src/github.com/Stakater/IngressMonitorController` with `Stakater` in capital case in its path. 
Updating the owner of Stakater organization to `stakater` in small case and scanning the organization again will fix this issue.