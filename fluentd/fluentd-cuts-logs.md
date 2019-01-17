# fluentd cuts/truncates logs above 16k

Log message is split into chunks about ~16374 characters when using fluentd driver with docker. Log message is split into 
chunks which makes it more complicated to investigate through this log.

Basically its docker and not fluentd which breaks the log messages; lets look why docker does it

## Docker

Docker chunks log messages at 16K.
It does mark the log as a partial message but really depends on the driver/service to re-assemble.
We have to chunk log messages otherwise we can end up with a container that can crash dockerd by simply logging to stdout and 
not sending a `\n`.

THe message struct has a `Partial` field which is true for each message until a `\n` is found.
Up to the driver to handle it. The log driver definitely should not buffer it, so it would need to be handled on the logging service as well.

## Solution

A possible worked around this issue by utilizing this fluentd plugin: https://github.com/fluent-plugins-nursery/fluent-plugin-concat
The downside to this plugin is that it only happens to work based on the fact that a single newline character is on the end of 
each complete log message. I don't feel particularly comfortable depending on that character being there forever, so maybe we 
could come up with a better indicator of the end of a message.

```yaml
  # Concatenate multi-line logs (>=16KB)
  <filter kubernetes.var.log.containers.**>
    @type concat
    key log
    multiline_end_regexp /\n$/
    separator ""
  </filter>
```

Just after the kubernetes_metadata injection.

## References

- https://github.com/moby/moby/issues/34620
- https://github.com/moby/moby/issues/34855
- https://github.com/kubernetes/kubernetes/issues/52444
