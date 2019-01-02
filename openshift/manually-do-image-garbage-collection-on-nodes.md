# Manually do docker image garbage collection on nodes in case of failure

Found an issue earlier on one of openshift cluster nodes that it was throwing out the following error:

```
E1228 09:45:25.697214  112052 remote_image.go:130] RemoveImage "sha256:025f74807c1e294419c09ad7881a51f32c3ea8527ff23b64e2e37aedc9e6a335" from image service failed: rpc error: code = 2 desc = Error response from daemon: {"message":"conflict: unable to delete 025f74807c1e (must be forced) - image is referenced in one or more repositories"}
```

This was a node level event error that I verified via `oc describe node node-name` and there was this event:

```sh
Events:
  FirstSeen	LastSeen	Count	From					SubObjectPath	Type		Reason		Message
  ---------	--------	-----	----					-------------	--------	------		-------
  50d		3m		14275	kubelet, node-name			Warning		ImageGCFailed	(combined from similar events): wanted to free 26811983462 bytes, but freed 0 bytes space with errors in image deletion: [rpc error: code = 2 desc = Error response from daemon: {"message":"conflict: unable to delete fff3d5f84b92 (must be forced) - image is referenced in one or more repositories"}, rpc error: code = 2 desc = Error response from daemon: {"message":"conflict: unable to delete ece0ddae4bba (must be forced) - image is referenced in one or more repositories"}
```

Upon further investigation, found out that there were GC thresholds set to a lower percent as compared to the value set for `DiskPressure`.

This failure can occur when there are custom registry images for deployment configs. And the GC is not sure that whether they should be deleted or not. This issue has been fixed in a newer version of Openshift. However a manual fix is mentioned here: https://github.com/openshift/openshift-docs/blob/enterprise-3.7/day_two_guide/topics/pruning_images_and_containers.adoc#pruning-old-images-and-containers


Basically we just have to ssh into the node and delete the images manually.