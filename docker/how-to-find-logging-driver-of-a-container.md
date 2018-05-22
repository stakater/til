# How to find logging driver of a container?

To find the current logging driver for a running container, if the daemon is using the json-file logging driver, run the following docker inspect command, substituting the container name or ID for <CONTAINER>:

```
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

journald
```

When you start a container, you can configure it to use a different logging driver than the Docker daemon’s default, using the --log-driver flag.

The following example starts an Alpine container with the none logging driver.

```
$ docker run -it --log-driver none alpine ash
```