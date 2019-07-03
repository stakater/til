# Build go Project

## Overview

This guide provide guideline on how to build a go project.

## Build Steps

* Clone the golang repository.

* Run the image container

* Update package repository using the command given below:
```bash
$ apt update
```

* Install `glide` using the command given below:
```bash
$ apt install golang-glide
```

* Clone the Project.

* Install all the packages using the command given below:
```bash
$ glide install
```

* Once all the packages are installed run the command given below to build the project:
```bash
$ go build
```

* Project will be built in the root directory.

* To run the command run the command given below:
```bash
$ sudo ./ProjectBinary
```