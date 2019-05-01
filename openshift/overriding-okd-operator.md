# Overriding OKD Operator

## Scenario

Lets say you are using OKD's prometheus operator and right now the latest version available with OKD is 0.22.2 while the latest prometheus operator version available in 0.29.0. So how do we use the latest v0.29.0 of prometheus operator in openshift?

## Solution

We can achieve this by first getting the `ClusterServiceVersion` of the current version. You can do this by creating a subscription for the desired operator and then going to `k8s/all-namespaces/clusterserviceversions` in the browser.
Select the subscription you created and copy its yaml file. Now make the changes that you feel necessary and also the following
- Change version
- Remove fields like selflink, uuid etc.

Now you can simply create this new CSV by doing `oc apply` and use by referencing this new CSV in the subscription.
