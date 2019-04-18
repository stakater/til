# Operators in Openshift

Openshift has built in support for quite a few useful operators. To be able to use the operators, first we have to enable Operator Lifecycle Manager. To install it with ansible, just run:

```bash
ansible-playbook -i <inventory_file> /usr/share/ansible/openshift-ansible/playbooks/olm/config.yml
```

You can read more [here](https://docs.okd.io/latest/install_config/installing-operator-framework.html#installing-olm-using-ansible_installing-operator-framework) if you wish to configure pulling images from redhat registry for operator containers.

Once the above playbook has run, in the `cluster console` tab of okd console, a new menu `Operators` appears in the navigation section which can be used to subscribe to different available operators.
If you wish to subscribe via command line, then a sample subscription looks like this

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generateName: couchbase-enterprise-
  namespace: couchbase-test
spec:
  source: certified-operators
  name: couchbase-enterprise
  startingCSV: couchbase-operator.v1.0.0
  channel: preview
```

Once you create the subscription, the operator is automatically created.
