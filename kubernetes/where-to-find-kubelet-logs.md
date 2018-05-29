# kubelet logs

It depends how it was installed

If you run kubelet using systemd, then you could use the following method to see kubelet's logs:

```
# journalctl -u kubelet
```