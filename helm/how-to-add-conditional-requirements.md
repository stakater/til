# How to add conditional requirements

In our requirements.yaml file we can add conditional dependancy like this: 

```
  - name: postgresql
    version: 0.8.3
    repository: https://kubernetes-charts.storage.googleapis.com/
    condition: postgresql.enabled
```    

It will only download this dependancy if postgresql.enabled is true in values.yaml