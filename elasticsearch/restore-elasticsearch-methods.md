# Restore Elasticsearch data

## Overview
This guide provides guideline on how to back elasticsearch data.


## Steps

There are currently two method:

### **Method 1**
* Creating backup repo, follow this [guide](https://z0z0.me/how-to-create-snapshot-and-restore-snapshot-with-elasticsearch/) for this method/


### **Method 2**
This method is specific to the `stakater/elasticsearch:2.3.1` image used by team:

The problem with this image is that once image is started we can move the data from outside the container to the `/usr/share/elasticsearch/data` folder path because logs show that this folder is busy which neither allow to modify this folder not delete it.

If container is started and data is copied after the start the elasticsearch will not load the data.

Workaround for this issue is that the data is copied inside the image during image build process. I have tested it using the steps given below:

* Create a dockerfile for new image that uses this image and copy the data at build time:

```
FROM stakater/elasticsearch:2.3.1

# place the dockerfile where the data exists
COPY ./ /usr/share/elasticsearch/data

ENTRYPOINT ["service", "elasticsearch", "start"]
```

* Build the image using the command given below:
```bash

sudo docker build -t <imagename:imagetag>
```

* Start the container using the command given below:
```bash
sudo docker run -it <imageName:imageTag>
```
It will start the elasticsearch server at container start time, which loads the data.

* Run the command given below for verification:

```bash

curl http://localhost:9200/_aliases [use this command to get the indices]

curl -X GET "localhost:9200/<indice-name>" [to get the indice data]
```



