# S3 file Upload

## Overview

This document provide guidelines on how to push files on s3 in Jenkins credentials.

## Guideline

* Install the plugin [`Pipeline: AWS Steps`](https://github.com/jenkinsci/pipeline-aws-plugin) using these steps `Jenkins Dashboard -> Manage Jenkins -> Manage Plugins`.

* Create a user on aws and assign limited access to the user to perform operations only on the AWS S3 resource.

* Create a Credentials using this steps `Credentials -> Add Credentials -> AWS Credentials`, add the appropriate `AWS KEY ID` and `AWS ACCESS KEY`. Once everything is configured, save the credentials.

* Using the snippet given below we can upload a file on the S3 bucket:

```groovy
withAWS(credentials: 'jenkins-aws-credentials-name', region: "aws-region") {
    s3Upload(file: "filename", bucket: "bucketName", path: "s3-bucket-path")
}
```

