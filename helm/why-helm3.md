# Helm 3

Currently Helm 3 is being developed that has a number of great improvements:

- remove the server side component, `Tiller`, so that helm install uses the current user/ServiceAccount’s RBAC
- releases become namespace aware avoiding the need to come up with globally unique release names
