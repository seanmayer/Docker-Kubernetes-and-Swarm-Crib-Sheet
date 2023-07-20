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

### Docker Proces Monitoring

- `docker container top <container id>` - show running processes in container
- `docker container inspect <container id>` - show metadata about container (including process ID)
- `docker container stats` - show live performance stats for all containers

### Docker Networking

- `docker container port <container id>` - show port mapping of container
- `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <container id>` - show IP address of container
- `docker network ls` - list networks
- `docker network inspect <network id>` - show metadata about network
- `docker network create <network name>` - create network
- `docker network connect <network name> <container id>` - connect container to network
- `docker network disconnect <network name> <container id>` - disconnect container from network

### Docker Images

- `docker image ls` - list images
- `docker image history <image id>` - show history of image
- `docker image inspect <image id>` - show metadata about image
- `docker image tag <image id> <new image name>` - tag image
- `docker image build -t <image name> .` - build image from Dockerfile
- `docker image push <image name>` - push image to Docker hub
- `docker image pull <image name>` - pull image from Docker hub


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






Test: `curl localhost:8800` # Should return html

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