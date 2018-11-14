- Rate limiting
- Suppressed messages

```
Suppressed 618 messages from /system.slice/docker.service
```

The limits are controlled in the /etc/systemd/journald.conf file.

```
RateLimitInterval=30s
RateLimitBurst=1000
```

If more messages than the amount specified in RateLimitBurst are received within the time defined by RateLimitInterval, all further messages within the interval are dropped until the interval is over.

You can modify these values as you see fit, you can completely disable systemd journal logging rate limiting by setting both to 0.

If you make any changes to /etc/systemd/journald.conf you will need to restart the systemd-journald service to apply the changes.

```
systemctl restart systemd-journald
```

https://www.rootusers.com/how-to-change-log-rate-limiting-in-linux/

Add ansible playbook to change the rate limiting on systemd journald and restart the service:

```
TODO
```
