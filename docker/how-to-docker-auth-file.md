# How to create docker auth file?

The easiest and simplest solution is to do docker login i.e.

```
docker login -u username -p password docker.abc.com
```

This will create an entry in ~/.docker/config.json:

```
{
	"auths": {
		"your-repo:8082": {
			"auth": "YWRtaW46YWRtaW4xMjM="
		}
}
```