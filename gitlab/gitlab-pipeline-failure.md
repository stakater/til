If the pipeline fails with something similar to "container probably didn't start properly" followed by an error saying that the container name is already in use then do the following.

- Go to the aws dev environment, go to systems manager -> sessions manager and start a new session with the gitlab runner instance.
- Run following command to clear any left over containers that are stuck in exited / removal in progress. (They sometimes, probably after manual cancelation of pipeline, get stuck and will not go away).

If you want to run this for other projects, change XXXX to that project id.

for c in $(sudo docker ps -a | grep project-XXXX | cut -d ' ' -f1); do sudo docker rm -f $c; done
