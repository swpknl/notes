### Introduction
- Solves for matrix hell in development environments
- Containers are completely isolated environments like VMs except they share the underlying OS kernel
- Docker can run software which share the same underlying kernel
- So Windows cannot be run in Linux OS. 
- When we run a Linux container in Windows we use WSL, which runs Linux on Windows
- Docker has less isolation; while VMs have complete isolation
- Container are running instances of Docker images
- Docker images are templates with packages

### Commands
```bash
docker run # will run an image. If not present, it'll pull and then run
docker ps # list all the containers running locally. 
docker ps -a # shows all exited and running containers
docker stop # stop a container command. Provide an id or container name
docker rm # remove a stopped or exited container permanently
docker images # list all the docker images downloaded locally
docker rmi # Remove an image. The image to be removed must be stopped and deleted'
docker pull # docker pull will download an image
#Containers are not meant to be run OSes. They are designed to run a specific task. Once the task is complete, it'll exit. This is why if we run ubuntu, it exits immediately
docker exec # Execute a command on a running container
docker run # runs in the foreground and is attached to the container. It'll not listen to any other commands
docker run -d # Runs in detached mode and runs the container in background mode
docker attach # Run the background container as a foreground from background
```

docker run tag:
```bash
 docker run redis:4.0 # 4.0 is the tag. If tag is not given, then latest is use, which is the latest version
```

docker run stdin: 
- docker run -i : runs in interactive mode
- Need to attach to the prompts terminal
- So command will become: docker run -it - attaches to the current terminal

Map ports:
- use -p to map port in host to that of docker
```bash
docker run -p 80:5000 kodekloud/webapp
```
- It is of the format localPort:containerPort

Persistence of data in docker:
- Each docker container has its own isolated file system
- If we delete the container, the data inside the container is also thrown away
- If we want to persist the data, we need to map to the host volume to that of the container:
	- Use -v
```bash
docker run -v /opt/datadir:/var/lib/mysql mysql
```

Docker inspect:
- returns the details of the container

Container logs:
```bash
docker logs container_id
```

Detached and attached mode:
- -d to run as detached
```
docker attach id # to attach to a container
```


Creating your own image
- Create your own Dockerfile
- Build your image using the docker build command
```bash
docker build Dockerfile -t tag
```
- Push the image to the registry
```bash
docker push
```

Dockerfile:
- Text file written in a specific format with the format:
	Instruction Argument
- Everything on the left is an instruction
	- Example: FROM, RUN, COPY, ENTRYPOINT, EXPOSE, VOLUME, WORKDIR, CMD
	- Every Dockerfile is based on a base image of an OS, or another image created before based on an OS
	- RUN runs a command in the docker image
- Docker builds a instruction using a layered architecture
	- Each layer stores the changes from the previous layer and this reflects in the size of the image
	- All layers built are cached, docker build will reuse the layer from the cache and continue
	- This way rebuilding an image is faster
	- Example with tag:
		-docker build . -t <account_name>/<application_name>
- docker login first to push.
	- Use the command: docker login

Pass env variable: 
- docker run -e APP_COLOR=blue simple-webapp-color
- Inspect env variable of a container:
	- use docker inspect
		- Check the config section

Entrypoint in Dockerfile:
- CMD ["command", "param"]
- ENTRYPOINT is like command which will run when the container starts
	- ENTRYPOINT ["sleep"]
- To override the ENTRYPOINT use --entrypoint argument 

Docker compose:
- docker run --links - it is used to link separate containers
```bash
docker run -d --name=bote -p 5000:80 --link redis:redis voting-app
```
- This adds an entry to the etc/hosts file that the 2 separate containers are to be run together
- links has been deprecated and there are advanced and better ways are present, like Docker swarm and networking
- Running separate docker images can be done using docker-compose.yml
- in the file, we specify the names of the images
	- Under the image name, we have key-value pair, where the key is image and the value is the name of the image
- We also have ports
- And finally we have links in the file as well
- Run the docker compose up command to run the entire application stack
- We can specify the version of the docker compose file at the top using:
	- version: 2
	- It also specifies the depends_on, where we can specify the start order
- Version 3: 
	- It is similar to version 2 and specifies the services section similar to version 2 where we can specify the services
	- It also specifies the docker swarm
- Networks in docker compose:
	- create a new property called networks in the root of the file next to the services section
	- Then specify the networks in each image in the services section

Docker engine
- When we install Docker engine, we install the docker daemon, docker REST API server, and the docker CLI
```bash
docker -h hostname run ngnix # to run docker container on a remote host
```
- The docker container shares the same CPU and memory but they use namespaces to differentiate between host and containers.
- In Linux, the root process has a PID of 1. The child container also runs in its own namespace with the root PID of 1. When docker run is executed, the PID of the container process maps to that in a different PID of the host as it is another process running in the host system
- docker run --cpus=0.5 --memory=100m - to specify cap in memory and CPUs. This is done using CGROUPs or control groups

Docker file system:
- All data is stored in /var/lib/docker
- Dockerfile image layers are read only
- When we execute docker run, a read write layer called container layer is created, for storing logs etc, which is alive till the container is alive
- The image layer is shared with all containers using the image
- Docker uses "Copy-on-write" for source code changes. The source code file is copied to the read write layer from the image and all changes in the source code happen in the read write layer
- docker uses volume mounting to map data from the container to a location in the host file system
- It is under the folder volumes under /var/lib/docker
- It also uses bind mounting to mount any directory on the docker host
- -v is the old way, the new way is --mount
```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```
- Docker uses storage drivers to map volumes/directories
	- AUFS
	- ZFS
	- BTRFS
	- Device Mapper
	- Overlay
	- Overlay2

Docker networking
- When we create a container we create 3 networks:
	- Bridge
	- None
	- Host
- We can specify the network type using --network
- Bridge = Is the default network and is private when a container is created. They are typically in the range of 172.17 series. It creates an internal IP. To access the container externally, using port binding
- Host - It binds to the host network without network isolation. We will now not be able to run multiple containers with the same port in the same host individually
- None - They run in an isolated network and are not attached to any network and do not have any access to any network
- We can create our own network in a docker container
	- We use the docker network command
- Containers reach other using the names, or by using IP, but IP changes when container is rebooted
- Docker has a built in DNS resolver, which resolves using the container name. It runs by default on 127.0.0.11

Docker registry
- We can host our own docker registry using the official docker registry image called registry
- When we push, if we only specify the name of the image, it'll push to library, which is the official dockerhub registry
- We can run the dockerhub registry locally using 
```bash
docker run --name registry -p 5000:5000 registry:2
```
- We can push any image to our local registry via the following:
```bash
docker tag nginx:latest localhost:5000/nginx
docker push localhost:5000/nginx
docker pull localhost:5000/nginx
```
- We can use docker login to authenticate against a registry