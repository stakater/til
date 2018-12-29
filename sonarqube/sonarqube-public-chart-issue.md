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