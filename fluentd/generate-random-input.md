# Generate random logs from source

You can generate random logs by using in_exec input plugin. Simply put this as your source and update the exec command and interval and you're good to go.

```xml
<source>
  @type exec
  command "echo \"test\""
  keys k1
  run_interval 2s
  tag test
</source>
```