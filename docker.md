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

## Initialize Docker Swarm
docker swarm init

## Run Docker Service
docker stack deploy -c docker-compose.yml [name]

## View Services Running
docker service ls

docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager

## Create VM
docker-machine create --driver virtualbox myvm1
docker-machine ls

## Register Swarm
docker-machine ssh myvm1 "docker swarm init --advertise-addr (ip given, ex. 192.168.99.100)"


## Quick Start Ubuntu
docker run -i -t ubuntu /bin/bash

## SSH into Container
docker exec -it <container name> /bin/bash

## Add to /etc/hosts for speed
127.0.0.1	localunixsocket.local

## COMMON
### Build image
docker build -t postgres:eno .

### Build container / mount host
docker run -v "$(pwd)"/data:/var/lib/postgresql <container>
docker run  --mount type=bind,source="$(pwd)"/data,/var/lib/postgresql <container>
```
