# Collecting and parsing multiline logs

For fluentd, a multiline log is just a single log containing new line delimiters e.g., `\n`. And mostly our logs are in docker format, so multiline logs are separated by docker. When fluentd reads them, they look just like single logs because they're already separated. In order to handle this, we have to perform 2 steps:

* Concatenate the logs
* Treat the concatenated logs as a single log and parse them

## Concatenating multiline logs

For concatenating logs, we need [fluent-plugin-concat](https://github.com/fluent-plugins-nursery/fluent-plugin-concat)

Once you have the plugin installed, you need to have a filter with a starting multiline regex, and a timeout label that will treat timeout logs as normal logs:

```xml
<filter tag>
    @type concat
    key log
    multiline_start_regexp /^\d+(?:-\d+){2}\s+\d+(?::\d+){2}\.\d+/
    flush_interval 5s
    timeout_label @LOGS
</filter>

# Relabel all logs to ensure timeout logs are treated as normal logs and not ignored
<match **>
    @type relabel
    @label @LOGS
</match>

<label @LOGS>
# Parser filters go here
</label>
```

Once this is done, all the multiline logs will be concatenated and ready for parsing.

## Parsing multiline logs

For parsing you need to use a parser filter like so inside the @LOGS label:

```xml
<label @LOGS>
    <filter tag>
        @type parser
        key_name log
        reserve_data true
        <parse>
            @type regexp
            expression full-multiline-regex
            time_format logs-time-format
        </parse>
    </filter>
</label>
```

And thats it. Any match filter after this will get the fully parsed multiline log.

## Example for java spring multiline logs

For java spring multiline logs, the multiline regex will be

```regex
/^(?<time>\d+(?:-\d+){2}\s+\d+(?::\d+){2}\.\d+)\s*(?<level>\S+) (?<pid>\d+) --- \[(?<thread>[\s\S]*?)\] (?<class>\S+)\s*:\s*(?<message>[\s\S]*?)(?=\g<time>|\Z)/
```

And the time_format will be `%Y-%m-%d %H:%M:%S.%L`

## Configuring apps for fluentd to handle its logs

In our cluster, we've setup fluentd in such a way that it accepts custom fluentd configuration from deployments, statefulsets, and daemonsets. In ordewr for fluentd to parse multiline logs of a certain app that is deployed, you need to add the following annotation to its resource and fluentd will pick it up and parse inner logs

```yaml
fluentdConfiguration: >
    [
        {
        "containers":
            [
                {
                "expressionFirstLine": "{{ .regexFirstLine }}",
                "expression": "{{ .regex }}",
                "timeFormat": "{{ .timeFormat }}",
                "containerName": "spring-boot"
                }
            ]
        }
    ]
```

Where `expressionFirstLine` is the regex used to concatenate logs that are split up, and `expression` is the regex used for parsing internal log of the application.
