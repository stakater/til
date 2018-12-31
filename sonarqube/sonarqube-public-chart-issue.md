# Sonarqube public chart issue

*   Public chart version: 0.10.3
    Sonarqube image tag: 6.7.6-community

    Doesn't start sonarqube if custom plugins are installed, custom `sonar.properties` files doesn't work as well. 

*   Public chart version: 0.10.3
    Sonarqube version: 7.4-community (latest right now)
    
    Starts sonarqube with custom plugins, but do no install default plugins then. Custom `sonar.properties` files doesn't work as well. 

*   Public chart version: 0.10.3
    Sonarqube version: 7.1
    
    Starts sonarqube with custom plugins, and installs default plugins correctly too. Custom `sonar.properties` files still doesn't work.


# Further findings

I finally managed to install plugins, and pass properties file in the version `0.7.2` of sonarqube public chart. The issue is all the passed properties will work fine, but when viewed from dashboard, default properties will be shown. This seems to be a bug in the old version. For using this version following command must be passed in the values file 

```
  command: [
    "sh",
    "-ce",
    "mkdir /scripts &&
    cp /tmp-script/startup.sh /scripts/startup.sh &&
    chmod 0755 /scripts/startup.sh &&
    /scripts/startup.sh
    "
    ]
```