# Assuming an IAM role in AWS

To assume an IAM role we need to do the following things

- Update the trust relationships of the role which we want to be assumed like this:

```
      "Principal": {
        "AWS": "ARN_OF_MAIN_ROLE_LINKED_TO_EC2_INSTANCE"
      },
      "Action": "sts:AssumeRole"
```

Here Principal uses the ARN of main role whici can assume this particular role, and Action is a flag which tells that this role can be assumed

- Update the Permession Policy of the main role which is linked to the EC2 instance. We need to create a policy rule for "STS" service for action "AssumeRole" and add the ARN of the role be assumed in the "resources" of this newly created STS rule. In this way our main role will know that this specific ARN can be assumed. 

## References

- https://docs.aws.amazon.com/cli/latest/userguide/cli-roles.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_roles.html#troubleshoot_roles_cant-assume-role
