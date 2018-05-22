# How to get values of environment vars?

## Problem

Sometimes we need to find what exact values of environment variable are being passed to the container?

## Solution

Probably the most valuable use of `inspect` for me in the past has been getting the values of environment vars.

```
$ docker inspect logtest
[
  {
    “Id”: “fdb3008e70892e14d183f8 ... 020cc34fec9703c821”,
    “Created”: “2016–03–23T17:45:01.876121835Z”,
    “Path”: “/bin/sh”,
    “Args”: [
      “-c”,
      “while true; do sleep 2; echo ‘Testing log’; done”
    ],
    … and lots, lots more.
  }
]
```

I’ve skipped the bulk of the output because there’s a lot of it. Some of the more valuable bits of intelligence you can get are:

- Current state of the container. (In the “State” property.)
- Path to the log history file. (In the “LogPath” field.)
- Values of set environment vars. (In the “Config.Env” field.)
- Mapped ports. (In the “NetworkSettings.Ports” field.)
