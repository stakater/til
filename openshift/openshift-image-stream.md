# What are OpenShift image stream?

This is good read [Image Streams FAQ](https://blog.openshift.com/image-streams-faq/)

9. What are the three biggest advantages of using Image Streams?

- You can tag, rollback a tag, and quickly deal with images, without having to re-push using a docker command line.
- You can trigger Builds and Deployments when new image is pushed to the registry, or schedule the import to do it later.
- You can share images using fine-grained access and quickly distribute images across your teams.

2. What is the benefit of using an Image Stream over a regular image?

An Image Stream provides a stable pointer to an image using various identifying qualities. This means that even if the source image changes, the Image Stream will still point to a known-good version of the image, ensuring that your application will not break unexpectedly.

How does it track a known-good version? Users tend to refer to images by one of their tags because these are easily human-readable. However, every image can also be identified by its ID, which is not really human-readable (it is a SHA256 string and image:latest reads much better than image@sha256:8698dcd8eb…). If the author of an image pushes a newer version without properly tagging the image, they can accidentally create a new distinct image (with newer version information). While there are use cases when this is desired and perfectly reasonable (like security updates), often times it can impact our application in unexpected ways (eg. configuration change causing our application not to start).
