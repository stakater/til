# Skopeo
Skopeo is a command line utility that performs various operations on container images and image repositories. Skopeo can work with OCI images as well as the original Docker v2 images.

It does not require Docker Engine to perform its operations

## Operations
Skopeo can perform:
- Inspection of Remote Images without actually pulling the image
- Copy of Container Images across local to remote repositories and vice versa
- Pass credentials for Source and Destination repositories if needed
- Deleting an image in the repository


## Inspecting image without actually pulling
    # show properties of fedora:latest
    $ skopeo inspect docker://docker.io/fedora
    {
        "Name": "docker.io/library/fedora",
        "Tag": "latest",
        "Digest": "sha256:cfd8f071bf8da7a466748f522406f7ae5908d002af1b1a1c0dcf893e183e5b32",
        "RepoTags": [
            "20",
            "21",
            "22",
            "23",
            "heisenbug",
            "latest",
            "rawhide"
        ],
        "Created": "2016-03-04T18:40:02.92155334Z",
        "DockerVersion": "1.9.1",
        "Labels": {},
        "Architecture": "amd64",
        "Os": "linux",
        "Layers": [
            "sha256:236608c7b546e2f4e7223526c74fc71470ba06d46ec82aeb402e704bfdee02a2",
            "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
        ]
    }

## Copying images across local/remote registries/directories
    $ skopeo copy docker://busybox:1-glibc atomic:myns/unsigned:streaming
    $ skopeo copy docker://busybox:latest dir:existingemptydirectory
    $ skopeo copy docker://busybox:latest oci:busybox_ocilayout:latest

## Delete an image from a local repository
    $ skopeo delete docker://localhost:5000/imagename:latest

## Passing Source/Destination credentials
    $ skopeo copy --src-creds=testuser:testpassword docker://myregistrydomain.com:5000/private oci:local_oci_image
