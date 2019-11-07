# Removing Images from Nexus

You can install Nexus CLI and remove images from Nexus. 

Check details from [here](https://www.blog.labouardy.com/cleanup-old-docker-images-from-nexus-repository/)

Run 

```sh
wget https://s3.eu-west-2.amazonaws.com/nexus-cli/1.0.0-beta/linux/nexus-cli
chmod +x nexus-cli
nexus-cli configure
nexus-cli image ls
nexus-cli image tags -name IMAGE_NAME
nexus-cli image info -name IMAGE_NAME -tag TAG
nexus-cli image delete -name IMAGE_NAME -tag TAG
```