# SECURITY IMPLICATIONS

Below is an example of what is referred to as a super-privileged container. It is designed to have almost complete access to the host system as root user. The following docker command options open selected privileges to the host:

`-d` : Runs continuously as a daemon process in the background

`--privileged` : Turns off security separation, so a process running as root in the container would have the same access to the host as it would if it were run directly on the host.

`--net=host` : Allows processes run inside the container to directly access host network interfaces

`--pid=host` : Allows processes run inside the container to see and work with all processes in the host process table

`--restart=always` : If the container should fail or otherwise stop, it would be restarted
