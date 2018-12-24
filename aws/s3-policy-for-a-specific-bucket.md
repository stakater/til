# S3 role policy for a specific bucket

To grant s3 rights lets say on a specific bucket we can use the following policy: 

```
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::carbook-dev-images"
        }
    ]
}
```

Here `Resource": "arn:aws:s3:::carbook-dev-images` specifies that only the certain ARN will be given the permission