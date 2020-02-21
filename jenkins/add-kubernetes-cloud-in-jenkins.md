# Add Kubernetes as Cloud on Jenkins

1. Goto `Manage Jenkins` -> `Configure System`

2. Goto `Cloud` Section and click `Add a new Cloud` -> `Kubernetes`

3. `Kubernetes Cloud Details` ->
    - Set `Kubernetes URL` to `https://kubernetes.default`
    - Set `Kubernetes Namespace` to `delivery`
    - Set `Jenkins URL` to `http://stakater-delivery-jenkins:8080`
    - Set `Jenkins Tunnel` to `stakater-delivery-jenkins-agent:50000`
    - Set `Connection Timeout` to `5`
    - Set `Read Timeout` to `15`
    - Set `Concurrency Limit` to `10`
    - Set `Max connections to Kubernetes API` to `32`
    - Set `Seconds to wait for pod to be running` to `600`

4. `Pod Templates` ->

    - Set `Name` to `base`
    - Set `labels` to `stakater-delivery-jenkins-jenkins-slave` 
    - Set `Usage` to `Use this node as much as possible`
    - `Add Container` -> `Container Template` ->
        - Set `name` to `jnlp`
        - Set `Docker image` to `jenkins/jnlp-slave:3.27-1`
        - Set `Working directory` to `/home/jenkins`
        - Set `Arguments to pass to command` to `${computer.jnlpmac} ${computer.name}`
        - `Add Environment Variable` -> `Add Environment Variable`
            - Set `Key` to `JENKINS_URL`
            - Set `Value` to `http://stakater-delivery-jenkins.delivery.svc.cluster.local:8080` 

5. Set `Timeout in seconds for Jenkins connection` to `1000`
6. Check `Show raw yaml in console`