# How to find compatability?

- First go to https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#external-dependencies : just change the 
version and see which external dependencies it needs; it will tell which docker version it supports or is recommended
- Then go to https://coreos.com/releases/ and find what docker version this release holds

Then we can find what it the compatability!
