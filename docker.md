### Docker

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

## Run Dockerfile (name image starting)
docker build --tag=starting .

## Run App 
docker run -p 4000:80 tag
docker run -d -p 4000:80 tag (-d detached mode = run in background)
docker run -d -p 4000:80 username/repository:tag (remote image)

## Stop Container
docker container stop [c id]

## Push to Docker Hub
docker tag image username/repo:tag
docker push username/repo:tag

## Remove All Containers & Images
docker container rm $(docker container ls -a -q)
docker image rm $(docker image ls -a -q)
```
