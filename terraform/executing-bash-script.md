# Executing a Bash command in Terraform


`null_resource` can be used to execute a bash command in terraform. 

`Provisioners` are used to execute scripts on a local or remote machine as part of resource creation or destruction.
[Provisioners](https://www.terraform.io/docs/provisioners/index.html)

```
resource "null_resource" "echo_example" {
  provisioner "local-exec" {
  command = "echo My Name is Terraform Jr."
  }
}
```