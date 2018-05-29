# How to create a container which logs/print every second?

```
name=systemd-logs; docker rm -f $name; docker run -d --name $name --log-driver journald busybox sh -c 'i=0;while true; do echo ["$i"]: $(date); sleep 1; let i+=1; done'; docker logs -f $name
```

```
core@ip-10-4-145-37 /var/log $ name=systemd-logs; docker rm -f $name; docker run -d --name $name --log-driver journald busybox sh -c 'i=0;while true; do echo ["$i"]: $(date); sleep 1; let i+=1; done'; docker logs -f $name
Error response from daemon: No such container: systemd-logs
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
07a152489297: Pulling fs layer
```

```
core@ip-10-4-145-37 /var/log $ journalctl --unit docker
-- Logs begin at Mon 2018-05-28 20:59:39 UTC, end at Mon 2018-05-28 22:39:06 UTC. --
May 28 21:00:05 ip-10-4-145-37.eu-west-1.compute.internal systemd[1]: Starting Docker Application Container Engine...
May 28 21:00:05 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:05.087373001Z" level=warning msg="failed to rename /var/lib/docker/tmp for background
May 28 21:00:05 ip-10-4-145-37.eu-west-1.compute.internal systemd[1]: Started Docker Application Container Engine.
May 28 21:00:25 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:25.342894755Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:00:30 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:30.441729974Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:00:34 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:34.971907709Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:00:44 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:44.608526767Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:00:47 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:47.130237354Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:00:52 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:00:52.085483658Z" level=warning msg="Unknown healthcheck type 'NONE' (expected 'CMD') in
May 28 21:21:57 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:21:57.621855794Z" level=error msg="Download failed, retrying: unexpected EOF"
May 28 21:22:02 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:22:02.923326619Z" level=error msg="Download failed, retrying: could not parse Content-Ran
May 28 21:24:15 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:24:15.720427147Z" level=error msg="Download failed, retrying: unexpected EOF"
May 28 21:24:20 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:24:20.769939758Z" level=error msg="Download failed, retrying: could not parse Content-Ran
May 28 21:25:09 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:25:09.423814354Z" level=error msg="Download failed, retrying: unexpected EOF"
May 28 21:25:14 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:25:14.469093124Z" level=error msg="Download failed, retrying: could not parse Content-Ran
May 28 21:25:21 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:25:21.717927413Z" level=error msg="Download failed, retrying: unexpected EOF"
May 28 21:25:32 ip-10-4-145-37.eu-west-1.compute.internal env[1005]: time="2018-05-28T21:25:32.420446282Z" level=error msg="Download failed, retrying: unexpected EOF"
May 28 21:25:37 ip-10-4-1
```

