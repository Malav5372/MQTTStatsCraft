# Using Docker to Run `StatMQTT`

Verify your docker installation by:

```sh
sudo docker run hello-world
```

## Build `StatMQTT` image

```
StatMQTT/ > sudo docker build -t "StatMQTT:latest" .
```
Verify that the image is built:
```
 > sudo docker image ls
REPOSITORY                             TAG           IMAGE ID       CREATED              SIZE
StatMQTT                                 latest        de06300844f9   About a minute ago   63.1MB
public.ecr.aws/docker/library/python   3.11-alpine   5f9e8f452a5c   5 days ago           51.1MB
```

## Execute the `StatMQTT` Image in a Container

[Run the image](https://docs.docker.com/engine/reference/commandline/run/) we
created in the above step passing to it your configuration file
(`/opt/StatMQTT/StatMQTT.conf`)

```
docker run --rm -v /opt/StatMQTT/StatMQTT.conf:/opt/StatMQTT/conf/StatMQTT.conf:ro StatMQTT
```
