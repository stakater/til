# BUFFER_QUEUE_LIMIT

How to properly set buffer queue limit:

```BUFFER_QUEUE_LIMIT = FILE_BUFFER_LIMIT / (number_of_outputs * BUFFER_SIZE_LIMIT)```

we have had

```FILE_BUFFER_LIMIT=1Gi
BUFFER_QUEUE_LIMIT=1024
BUFFER_SIZE_LIMIT=1m```

and

```1024 ≠ 1024 / (2 * 1)```

the defaults are

```FILE_BUFFER_LIMIT=256Mi
BUFFER_SIZE_LIMIT=8Mi
BUFFER_QUEUE_LIMIT=32```

I'm thinking of setting

```FILE_BUFFER_LIMIT=1024Mi
BUFFER_SIZE_LIMIT=16Mi
BUFFER_QUEUE_LIMIT=32```

since

```32 = 1024 / (2 * 16)```
