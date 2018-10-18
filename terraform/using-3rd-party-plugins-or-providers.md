# Using 3rd Party plugins or providers

 Terraform supports use of 3rd party providers .These providers must be manually installed, since `terraform init` cannot automatically download them.

 Install third-party providers by placing their plugin executables in the user plugins directory. The user plugins directory is in one of the following locations, depending on the host operating system:

| Operating system  | User plugins directory |
|-------------------|------------------------|
| Windows           | `terraform.d\plugins` in your user's "Application Data" directory |
| All other systems | `.terraform.d/plugins` in your user's home directory |

Terraform plugins are written in go and must be compiled before adding it to the plugins directory.

Once a plugin is installed, terraform init can initialize it normally.

Providers distributed by HashiCorp can also go in the user plugins directory. If a manually installed version meets the configuration's version constraints, Terraform will use it instead of downloading that provider. This is useful in airgapped environments and when testing pre-release provider builds.

## Plugin Names and Versions

The naming scheme for provider plugins is terraform-provider-<NAME>_vX.Y.Z, and Terraform uses the name to understand the name and version of a particular provider binary.

If multiple versions of a plugin are installed, Terraform will use the newest version that meets the configuration's version constraints.

Third-party plugins are often distributed with an appropriate filename already set in the distribution archive, so that they can be extracted directly into the user plugins directory.