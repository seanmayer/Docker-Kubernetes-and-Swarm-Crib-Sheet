# Docker, Kubernetes and Swarm Crib Sheet

## Docker
Play with Docker: https://labs.play-with-docker.com/
Docker hub: https://hub.docker.com/


### Docker commands
`docker run -d -p 8800:80 httpd` # Run httpd container in background and map port 80 to 8800

Output: (downloads layers of image and runs container, creates networking, empty file system, etc)
``` Unable to find image 'httpd:latest' locally
``` Unable to find image 'httpd:latest' locally
latest: Pulling from library/httpd
d121f8d1c412: Pull complete
a3ed95caeb02: Pull complete
a3ed95caeb02: Pulling fs layer
a3ed95caeb02: Verifying Checksum
Digest: sha256:3c7d1b7a266e7a6d6b2c5b5bdc4c0a0d5b6a1a6c0f1e9a3e8c1e1a4b7b6b7b6b
Status: Downloaded newer image for httpd:latest
a3ed95caeb02: Pull complete
```

- `docker container run --publish 80:80 nginx` - run nginx container in foreground and map port 80 to 80

- `docker container run --publish 80:80 --detach nginx` - run nginx container in background and map port 80 to 80

- `docker container run --publish 80:80 --detach --name webhost nginx` - run nginx container in background and map port 80 to 80 and name it webhost

- `docker container ls` - list running containers

- `docker container ls -a` - list all containers

- `docker container stop <container id>` - stop container   

- `docker container rm <container id>` - remove container

- `docker container rm -f <container id>` - remove container even if it is running

- `docker logs <container id>` - show logs of container

- `docker container top <container id>` - show running processes in container

### Docker Ubuntu vs Alpine vs Centos vs Debian vs Fedora vs Oracle Linux vs Red Hat Enterprise Linux vs SUSE Linux Enterprise Server

Ubuntu: https://hub.docker.com/_/ubuntu
Alpine: https://hub.docker.com/_/alpine
Centos: https://hub.docker.com/_/centos
Debian: https://hub.docker.com/_/debian
Fedora: https://hub.docker.com/_/fedora
Oracle Linux: https://hub.docker.com/_/oraclelinux
Red Hat Enterprise Linux: https://hub.docker.com/_/rhel
SUSE Linux Enterprise Server: https://hub.docker.com/_/suse

- Ubuntu is the most popular Linux distribution for running Docker containers. It is the most similar to Debian.
- Alpine is the most lightweight Linux distribution for running Docker containers. It is the most secure Linux distribution.
- Centos is the most popular Linux distribution for running Docker containers. It is the most similar to Red Hat Enterprise Linux.
    - Centos tends to lag behind Red Hat Enterprise Linux in terms of updates
- Debian is the most popular Linux distribution for running Docker containers. It is the most similar to Ubuntu.
- Fedora is the most popular Linux distribution for running Docker containers. It is the most similar to Red Hat Enterprise Linux.
- Oracle Linux is the most popular Linux distribution for running Docker containers. It is the most similar to Red Hat Enterprise Linux.
- Red Hat Enterprise Linux is the most popular Linux distribution for running Docker containers. It is the most similar to Centos.
- SUSE Linux Enterprise Server is the most popular Linux distribution for running Docker containers. It is the most similar to Red Hat Enterprise Linux.


### Docker Proces Monitoring

#### Commands for process monitoring
- `docker container top <container id>` - show running processes in container
- `docker container inspect <container id>` - show metadata about container (including process ID)
- `docker container stats` - show live performance stats for all containers

#### Commands for process management
- `docker system df` - show docker disk usage
- `docker system events` - show docker system events
- `docker system info` - show docker system info
- `docker system prune` - remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes

### Getting inside a container

- `docker container run -it <container id> <command>` - run additional command in new container
- `docker container exec -it <container id> <command>` - run additional command in existing container

- `docker container run -it --name proxy nginx bash` - run nginx container in background and map port 80 to 80 and name it webhost
- `docker container exec -it <container id> bash` - run bash in existing container

- `docker container run -it --name ubuntu ubuntu` - run ubuntu container in background and map port 80 to 80 and name it webhost
    - `apt-get update` - update ubuntu
    - `apt-get install curl` - install curl

- `docker container exec -it <container id> bash` - run bash in existing container
- `docker container run -it alpine sh` - run bash in new container (alpine is a lightweight Linux distribution does not come with bash by default, so use sh instead of bash or install bash with `apk add bash`)

### Docker Networking

- `docker container port <container id>` - show port mapping of container
- `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <container id>` - show IP address of container
- `docker network ls` - list networks
- `docker network inspect <network id>` - show metadata about network
- `docker network create <network name>` - create network
- `docker network connect <network name> <container id>` - connect container to network
- `docker network disconnect <network name> <container id>` - disconnect container from network

Tips for networking in Docker containers (best practices):
- Each container should have only one concern
- Create your apps so frontend/backend sit on same Docker network
- Their inter-communication never leaves host
- All externally exposed ports closed by default
- You must manually expose via -p, which is better default security
- This gets even better with Docker Swarm and Overlay networks

#### Docker Networking: DNS

- You cannot rely on IP addresses to communicate between containers
- Use Docker DNS to allow containers to resolve each other by name
- Docker daemon has a built-in DNS server that containers use by default
- Try this: `docker container run -d --name my_nginx --network my_app_net nginx`
    - Ping the DNS: `docker container exec -it my_nginx ping new_nginx`
- Linking containers is a legacy feature
    - DNS is now the preferred method of inter-communication
    - Linking still works, but is not recommended
    - Linking may be removed in future versions of Docker

#### Docker Networking: DNS Round Robin Test

- Round Robin DNS is a technique of load distribution, load balancing, or fault-tolerance provisioning multiple, redundant Internet Protocol service hosts, e.g., Web server, FTP servers, by managing the Domain Name System's (DNS) responses to address requests from client computers according to an appropriate statistical model.
- Multiple containers can resolve to the same DNS name, and Docker will load balance requests between them

- Create a network: `docker network create dude`
- Run 2 containers: `docker container run -d --net dude --net-alias search elasticsearch:2`
- Run 2 containers: `docker container run -d --net dude --net-alias search elasticsearch:2`
- Run 1 container: `docker container run --rm --net dude alpine nslookup search` # Should return 2 IP addresses
- Run 1 container: `docker container run --rm --net dude centos curl -s search:9200` # Should return html

Example:
- ` docker container run -p 80:80 --name webhost -d nginx` - run nginx container in background and map port 80 to 80 and name it webhost
- `docker container port webhost` - show port mapping of container
- `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost` - show IP address of container

### Docker Images

- `docker image ls` - list images
- `docker image history <image id>` - show history of image
- `docker image inspect <image id>` - show metadata about image
- `docker image tag <image id> <new image name>` - tag image
- `docker image build -t <image name> .` - build image from Dockerfile
- `docker image push <image name>` - push image to Docker hub
- `docker image pull <image name>` - pull image from Docker hub

### Dockerfile

- `docker image build -t <image name> .` - build image from Dockerfile

Example:
```
FROM nginx:latest
WORKDIR /usr/share/nginx/html
COPY index.html index.html
```

Dockerfile commands:
- FROM - base image
- LABEL - metadata
- RUN - run command in container
- COPY - copy files from host to container
- ADD - copy files from host to container (can also download files from internet)
- CMD - run command when container starts
- EXPOSE - expose port when container starts
- ENV - set environment variable
- ENTRYPOINT - run command when container starts (preferred over CMD)
- VOLUME - create mount point and mark as holding externally mounted volumes from native host or other containers
- USER - set user name or UID
- WORKDIR - set working directory
- ARG - define build-time variable
- ONBUILD - adds trigger instruction when image is used as the base for another build
- STOPSIGNAL - sets the system call signal that will be sent to the container to exit
- HEALTHCHECK - tells Docker how to test a container to check that it is still working
- SHELL - override default shell

### Docker Compose

- Why use Docker Compose?
    - configure relationships between containers
    - save our docker container run settings in easy-to-read file
    - create one-liner developer environment startups
    - Compose is great for local development, test and CI workflows
    - Staging and production deployments
    - Swarm and Kubernetes

- Difference between Docker Compose and Dockerfile:
    - Dockerfile is used to build images
    - Docker Compose is used to run containers

- Difference between Docker Compose and Docker Swarm:
    - Docker Compose is for local development
    - Docker Swarm is for production deployments

- `docker compose up` - start container
- `docker compose up -d` - start container in background
- `docker compose down` - stop container
- `docker compose down --rmi local` - stop container and remove image

- `docker compose ps` - list containers
- `docker compose top` - show running processes in container
- `docker compose logs` - show logs of container

- `docker compose build` - build image from Dockerfile (e.g. /docker-compose.yml)
- `docker compose push` - push image to Docker hub

- `docker compose config` - validate and view the compose file

- `docker compose up -d --scale <service name>=<number of containers>` - start container in background and scale number of containers

- docker-compose.yml:
```
version: '3.1'

services:
  web:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - webnet
  redis:
    image: redis:latest
    networks:
      - webnet
networks:
    webnet:
``` 
    - compose yaml syntax: https://docs.docker.com/compose/compose-file/compose-file-v3/

### Docker persistent data

- Two ways to deal with persistent data:
    - Volumes
    - Bind mounts

- Volumes:
    - Make special location outside of container UFS
    - Used for databases
    - When the container is removed, the volume still exists
    - Volumes can be shared and reused among containers
    - Changes to a volume are made directly
    - Changes to a volume will not be included when you update an image
    - Volumes persist until no containers use them
    - Volumes are the best way to persist data in Docker

    - In dockerfile:
        - `VOLUME /var/lib/mysql` - create volume in container, where the volume is stored on the host machine (in this case, the volume is stored in /var/lib/docker/volumes/ on the host machine)

    - Docker volume commands:
        - `docker volume ls` - list volumes
        - `docker volume inspect <volume name>` - show metadata about volume
        - `docker volume create <volume name>` - create volume
        - `docker volume rm <volume name>` - remove volume
        - `docker volume prune` - remove all unused volumes
        - `docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql` - create mysql container with named volume mysql-db

- Bind mounts:
    - Link container path to host path
    - Used for development
    - When the container is removed, the bind mount is removed
    - Changes to a bind mount are immediately visible on the host
    - Changes to a bind mount will not be included when you update an image
    - Bind mounts persist until you delete them

### Bind mounts - local files and running apps on containers

- `docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx` - create nginx container with bind mount (pwd is the current directory) (this command will not work on Windows)

### How to upgrade a database and keep data persistent

- Create a named volume: `docker volume create postgres-db`
- `docker container run -d --name postgres -v postgres-db:/var/lib/postgresql/data postgres:13-bullseye` - create postgres container with named volume postgres-db
- `docker logs postgres` - show logs of postgres container
- `docker stop postgres` - stop postgres container
- `docker container run -d --name postgres-new -v postgres-db:/var/lib/postgresql/data postgres:13.11-bullseye` - create postgres-new container with named volume postgres-db
- `docker logs postgres-new` - show logs of postgres-new container
- `docker stop postgres-new` - stop postgres-new container

### File permissions across multiple containers

- use ps aux to find out the user id of the process running in the container
- find uid/gid of user in container: `etc/passwd` (uid:gid) and `etc/group` (gid) files to find out the uid/gid of the user running the process in the container
- find uid/gid if both containers are running with the matching user ID (uid) and group ID (gid)
- if the uid/gid of the user running the process in the container does not match the uid/gid of the user running the process in the container, then you will need to change the uid/gid of the user running the process in the container to match the uid/gid of the user running the process in the container
- dockerfile:
```
FROM nginx:latest
RUN groupadd --gid 1000 node \\
    && useradd --uid 1000 --gid node --shell /bin/bash --create-home node
USER 1000:1000
```
- Tip setting up Dockerfiles USER with numbers works better than names in Kubernetes

### Keeping Docker system clean

- `docker container prune` - remove all stopped containers
- `docker image prune` - remove unused images
- `docker system prune` - remove all unused data
- `docker system prune -a` - remove all unused data (including unused images)

### Docker logging

stdout and stderr are the output streams from a Linux process. By default, stdout and stderr are output to the console (your terminal window) where you started the process. You can redirect stdout and stderr to a file by using the > and 2> redirect symbols respectively.

### Docker install/config

#### Commands

- `docker version` - check docker version (client and server)
- `docker info` - check docker info (including storage driver)
- `docker` - list docker commands (including swarm commands)
   - Management Commands:
      - `docker container` - manage containers
      - `docker image` - manage images
      - `docker network` - manage networks
      - `docker node` - manage swarm nodes
      - `docker plugin` - manage plugins
      - `docker secret` - manage secrets
      - `docker service` - manage services
      - `docker stack` - manage Docker stacks
      - `docker swarm` - manage swarm
      - `docker system` - manage Docker
      - `docker volume` - manage volumes


### What is a container?

A container is a process or group of processes that are isolated from the rest of the system. It has its own file system, networking, etc. It is a process that is running on the host machine.

- Containers are not VMs
- Containers are restricted processes (restricted by Linux kernel)
   - This is why Docker Desktop uses lightweight Linux VM to run containers on Windows and Mac

- `ps aux` - list all processes running on host machine

- Nginx (web server) is a container that runs on the host machine
    - https://www.nginx.com/resources/glossary/nginx/ (open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more)


### What is image?

An image is a file that contains all the dependencies and configuration required to run a container. It is a read-only template with instructions for creating a Docker container.

- Images are not containers
- Images are not VMs
- Images are not processes
- Images are not files
- Images are not directories

- Nginx (web server) is an image that can be used to create a container
    - https://hub.docker.com/_/nginx

### What happens when you run a container?

1. Docker client sends a request to Docker daemon
2. Docker daemon checks if image exists locally
3. If image does not exist locally, Docker daemon downloads image from Docker hub
4. Docker daemon creates container based on image
5. Gives container a virtual IP on a private network inside docker engine
6. Opens up port on host and forwards to port in container
7. Starts container by using the CMD in the image Dockerfile

### Docker Swarm - orchestration

#### Managing multiple containers the problems:

- How do we automate container lifecycle?
- How do we scale containers horizontally?
- How do we ensure containers are re-created if they fail?
- How do we ensure containers are running on all nodes?
- How do we ensure the right containers are running on the right nodes?
- How can we replae containers without downtime (blue/green deployment)?
- How can we control/track where containers get started?
- How can we create cross-node virtual networks?
- How can we ensure only trusted servers run our containers?
- How can we store secrets, keys, passwords and get them to the right container when it needs it?

#### What is Docker Swarm?

- Swarm is Docker's orchestration tool
- Swarm is built inside Docker
- Swarm is a clustering and scheduling tool for Docker containers
- Swarm is the native clustering engine for Docker
- Swarm is a group of Docker engines running in swarm mode
- Not enabled by default, new commands when enabled
    - `docker swarm` - manage swarm
    - `docker node` - manage swarm nodes
    - `docker service` - manage services
    - `docker stack` - manage Docker stacks
    - `docker secret` - manage secrets
    - `docker config` - manage Docker configs

#### Docker Swarm - manager nodes:

- Manager nodes are responsible for maintaining the desired state of the swarm
- Manager nodes elect a leader to conduct orchestration tasks
- Manager nodes maintain the cluster state
- Manager nodes schedule services
- Manager nodes serve as the swarm HTTP API endpoints
- Manager nodes enforce policies
- Manager nodes encrypt/decrypt secrets
- Manager nodes are the only nodes where you can run `docker swarm` commands

#### Docker Swarm - worker nodes:

- Worker nodes receive and execute tasks dispatched from manager nodes
- Worker nodes run containers and services
- Worker nodes report back to manager nodes

Docker swarm commands:
- `docker info` - check if swarm is active
- `docker swarm init` - initialize swarm
- `docker node ls` - list nodes in swarm
- `docker service create alpine ping 8.8.8.8` - create service
- `docker service ls` - list services
- `docker service ps <service name>` - list tasks in service
- `docker service update <service name> --replicas <number of replicas>` - update service
- `docker service ls` - list services (replicas column shows number of replicas)
- `docker service ps <service name>` - list tasks in service (desired state column shows number of replicas)
- `docker update --help` - show help for update command (e.g. `docker service update --help`, this command shows help for service update command)

#### Docker Swarm - Creating 3 Node Swarm Cluster

- Create 3 VMs (e.g. using VirtualBox)
- Install Docker on each VM (e.g. `curl -fsSL https://get.docker.com -o get-docker.sh` and `sudo sh get-docker.sh`)
- Initialize swarm on one VM (e.g. `docker swarm init --advertise-addr` and `docker swarm join-token`)
- Join other VMs to swarm as worker nodes e.g. `docker swarm join --token SWMTKN-xxxx `
- Run `docker node ls` on manager node to verify swarm is active
- Run `docker node update --role manager <node id>` on manager node to promote worker node to manager node
- Run `docker swarm join-token manager` on manager node to get join token for manager node

Test: `curl localhost:8800` # Should return html

#### Docker Swarm - Overlay Networking

- Overlay networking is a built-in docker network driver
- Overlay networking enables multi-host communication
- Overlay networking requires a key-value store
- Overlay networking requires a routing mesh
- Overlay networking is the default network driver for swarm services
- Overlay networking is the best way to network containers across multiple hosts

- `docker network create --driver overlay <network name>` - create overlay network
- `docker service create --name <service name> --network <network name> <image name>` - create service with overlay network
- `docker service ls` - list services
- `docker service ps <service name>` - list tasks in service
- `docker container logs <container id>` - show logs of container

#### Docker Swarm - if we had 3 replicas and it created 3 tasks with 3 containers on 3 nodes

- Inside that overlay network, it creates a virtual IP that is mapped to the dns name of the service
- The "web" will load balance between the 3 containers
- This is not DNS round robin, this is a virtual IP that is mapped to the dns name of the service
- The benefits over DNS round robin is that it is faster

![Screenshot](images/vip-mapped.png)

#### Docker Swarm - Routing Mesh

- Routing mesh routes ingress (incoming) packets for a Service to proper Task
- Routing mesh routes packets on a Service's published ports
- Routing mesh routes packets for a Service to proper Node
- Routing mesh uses IPVS from Linux kernel

### Recommended VS Code extensions

- Docker (allows you to run docker commands from VS Code)
- Kubernetes (allows you to run kubectl commands from VS Code)
- Remote Development (allows you to run VS Code in a container)
    - use cases:
        - you don't want to install all the tools on your local machine
        - you want to use a different OS than your local machine
        - you want to use a different version of a tool than your local machine
- Remote SSH (allows you to run VS Code on a remote machine)
- Live Share (allows you to share your VS Code session with others)