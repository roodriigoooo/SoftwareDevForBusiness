# Docker 

## Table of Contents

 - [Why We Use Docker](#why-we-use-docker)
 - [Docker Structure and Architecture](#docker-structure-and-architecture)
   - [The Docker daemon](#the-docker-daemon)
   - [The Docker client](#the-docker-client)
   - [Docker registries](#docker-registries)
 - [Docker Objects](#docker-objects)
   - [Images](#images)
   - [Containers](#containers)
     - [Containers Sharing the Kernel](#containers-sharing-the-kernel)
     - [Sharing the Kernel vs. VMs Each with their Own Kernel](#sharing-the-kernel-vs-vms-each-with-their-own-kernel)
     - [Virtual Machines AND Docker](#virtual-machines-and-docker)
 - [Example docker run command](#example-docker-run-command)
 - [Dockerfile and Docker Image](#dockerfile-and-docker-image)
   - [Creating a Dockerfile](#creating-a-dockerfile)
     - [More comprehensive examples](#more-comprehensive-examples)
   - [Building the Dockerfile](#building-the-dockerfile)
 - [Docker Compose](#docker-compose)
   - [Example from class](#example-from-class)
 - [Docker System](#docker-system)
   - [Data Management in Containers](#data-management-in-containers)
     - [Creating New Data in Containers](#creating-new-data-in-containers)
     - [Modifying Files from the Image](#modifying-files-from-the-image)
     - [Data Persistence](#data-persistence)
 - [Docker Networking](#docker-networking)
   - [Bridge Network (Default)](#bridge-network-default)
   - [Host Network](#host-network)
   - [None Network](#none-network)
   - [Custom Docker Networks](#custom-docker-networks)
   - [Network Managements Commands](#network-managements-commands)
   - [Docker Hub and Private Repositories](#docker-hub-and-private-repositories)
 - [Docker Orchestration](#docker-orchestration)
   - [Why Use Docker Orchestration](#why-use-docker-orchestration)
   - [Key Concepts in Orchestration](#key-concepts-in-orchestration)
   - [Popular Orchestration Tools](#popular-orchestration-tools)

## Why We Use Docker
- Docker streamlines the development lifecycle by allowing developers to work in standardized environments. 
- It allows for the creation of local containers providing applications and services, which are great for continuous integration and delivery.
- Docker containers can run on a developer's local laptop, on physical or virtual machines, on cloud providers, or in a mix of environments. This portability and  lightweight nature make it easy to dynamically manage workloads in near real time. 

## Docker Structure and Architecture

- Docker uses a client-server architecture. The client talks to the Docker daemon, which is responsible for the heavy lifting: building, running, and distributing Docker containers. 
- The Docker client and daemon can run on the same system, or the client can be connected to a remote Docker daemon. 
- The client and daemon communicate using a REST API, over UNIX sockets or a network interface. 

### The Docker daemon

The docker daemon `dockerd` listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes. **Daemons can communicate with other daemons to manage Docker services.**

### The Docker client

The docker client `docker` is the primary way in which users interact with Docker. The `docker` command uses the Docker API to send commands, such as `docker run` or `docker compose` to `dockerd`, the daemon, which carries them out.

### Docker registries

Docker registries store Docker images. Docker Hub, for instance, is a public registry accessible to everyone. Docker looks for images on Docker hub by default. 

**Note**: You can also create and run your own private registry. When you use `docker pull` or `docker run`, Docker will pull the required images from the configured registry. Similarly, `docker push` pushes your image to the configured registry.

## Docker Objects

### Images
- **A read-only template with instructions for creating a Docker container.**
- Often, an image is based on another image, with some additional customization.

    - We can, for example, build an image based on the `ubuntu` image, but that additionally installs the Apache web server, our application, and the configuration details needed to make our application run.
- We can create our images or use those created and published in a registry, such as Docker Hub. 
- **To build our own images, we need to create a Dockerfile.**

    - Dockerfiles define the steps needed to create the image and run it. **Each instruction in a Dockerfile creates a layer in the image.**
    - If we change the Dockerfile and rebuild the image, **only those layers which have changed are rebuilt.** This is part of what makes images so lightweight, small and fast relative to other virtualization technologies. 

### Containers
- **A runnable instance of an image.** We can create, start, stop, move, or delete a container using the Docker API or CLI. 
- By default, containers are largely isolated, **and they all share the same kernel.**
- Containers can be connected to one or more networks.
- Storage can be attached to containers. 
- Images can be created based on a container's current, specific state. 
- **When a container is removed, any changes to its state that are not stored in persistent storage disappear**.

#### Containers Sharing the Kernel

Note that, when we say the containers share the kernel, we mean they all use the same core part of the OS (the kernel) from the host machine, while still running in isolated environents. 

- The kernel is the core part of the OS handling fundamental operations, as well as hardware resources (CPU, memory, devices, etc.)
- It also handles basic process management and scheduling, and controls low-level security and networking. 
- **Each container runs as an isolated process on the host**. They use namespace isolation and have their own view of the process tree, network interfaces, filesystem mounts, User IDs, etc. 

#### Sharing the Kernel vs. VMs Each with their Own Kernel
The shared kernel approach means:
1. Containers start faster (no need to boot additional OSs).
2. Containers use less memory (no duplicate kernel memory usage).
3. Containers are more lightweight.
4. The kernel must be compatible with the host kernel (A linux container will need a Linux kernel).

In many ways, it is like an apartment building:
- The building's foundation and core infrastructure (kernel) is shared. 
- All apartments (containers) are isolated and self-contained. 
- Residents (applications) can't see or interfere with each other. 
- Everyone still relies on the same building services. 

#### Virtual Machines AND Docker
There are many important reasons to use both VMs and Docker, as they serve different purposes and complement each other. 
1. Security and Isolation: VMs provide stronger isolation, due to having their own kernel. This is better for running untrusted, potentially malicious code. It also provides protection against kernel exploits. 
2. Different OS requirements: VMs allow us to run any type of containers on the same host machine, solving potential compatibility issues. 
3. Resource Management: VMs are more suitable for long-running, resource-intensive workloads. 

VMs are like separate buildings in a campus. Containers are like apartments in each building. 
Sometimes we need both levels of separation :)

## Example `docker run` command

The following command runs an `ubuntu` container, attaches interactively to our local command-line session, and runs `/bin/bash`.
```bash
$ docker run -i -t ubuntu /bin/bash
```
The following happens with this command:
1. If we do not have the `ubuntu` image locally, Docker pulls it automatically from the configured registry. It would therefore run `docker pull ubuntu` in the background. 
2. Docker creates a new container, as if we had run `docker container create`.
3. Docker allocates a read-write filesystem to the container as its final layer. This allows a running container to create/modify files and directories in it local filesystem.
4. Docker creates a network interface to connect the container to the default network, as we did not specify any networking options. **This includes assigning an IP address to the container**.

    - By default, containers connect to external networks using the host machine's internet network connection.
5. The container is started and `/bin/bash` is executed. The `-i` and `-t` flags cause the container to run interactively and attached to our terminal, respectively. This allows us to provide input while Docker logs the output to your terminal. 
6. When we run `exit` to terminate the `bin/bash` command, **the container stops but is not removed.** We can start it again or remove it. 

## Dockerfile and Docker Image
`docker run` is a command used to create and start a container from  specified image. 
```bash
# basic syntax
$ docker run [options] image [command][arg...]

# example
$ docker run -i -t ubuntu bin/bash
```
**Common `docker run` options**:
- `-d` (Detached mode): Runs the container in the background. 
```bash
# run nginx web server in the background
docker run -d nginx

# run detached container with name and ports
docker run -d --name webserver nginx -p 8080:80

# verify its running in the background
docker ps
```
- `p` (Port mapping): Maps a port on the host to a port on the container. `-p 8080:80` maps port 80 in the container to port 8080 on the host. 
```bash
# map single port
docker run -p 8080:80 nginx

# map multiple ports
docker run -p 8080:80 -p 443:443 nginx
```
- `-e` (Environment variables): Sets environment variables inside the container. 
```bash
# single environment variabke
docker run -e "DEBUG=true" my_app

# multiple env variables
docker run -e "DB_HOST=localhost" -e "DB_PORT=5432" postgres

# using env files
docker run --env-file ./env.conf mysql
```
- `--name`: Assigns a name to the container.
```bash
docker run --name myapp web-app 

#reference named container in other commands
docker exec -it myapp bash
docker logs myapp
```
- `it` or `-i` (interactive) and `-t` (psuedo-terminal): Allows us to interact with the container's terminal. Useful for containers that need user input or have a shell interface.
```bash
#basic interactive shell
docker run -it ubuntu bash 

#interactive with name and removal on exit
docker run -it --rm --name temp ubuntu bash
```

### Creating a Dockerfile

**Very basic example:**
```Dockerfile
FROM Ubuntu 

RUN apt-get update
RUN apt-get python

RUN pip install numpy

COPY ./opt/code
# we can copy the code from the current directory to a specific path inside the container.
.
.
.
```
- All Dockerfiles must start with a FROM instruction, which specifies the base OS or another image as the foundation (base image). 
- `apt-get` is a command in Linux used to install, update and remove software. 
- We can install dependencies: `RUN pip install numpy`, `RUN pip install -r requirements.txt`

#### More comprehensive examples

**1. FROM**:
```bash
# basic usage
FROM ubuntu:22.04

#multi-stage builds
FROM node:16 AS builder
FROM python:3.9-slim AS final
```
- FROM specifies the base image. 
- It can use multiple stages to reduce final image size.

    - Multi-stage builds allow a final image to include only the built applications, and not all the build tools and dependencies. This reduces the size of the image. 
    - By separating build and runtime environments, secrets and sensitive information such as API keys stay in the build stage, and source code and build tools are not in the production image.
```bash
# real-life example
# without multi-stage: 1GB image
FROM node:16
COPY . .
RUN npm install 
RUN npm run build 
CMD ["npm", "start"]

# with multi-stage: 100MB image
FROM node:16 AS builder
COPY . .
RUN npm install && npm run build

FROM node:alpine
COPY --from=builder /app/dist ./dist
CMD ["npm", "start"]
```

**2. ENV**:
```bash
# single environment variable
ENV NODE_ENV=production

# multiple variables
ENV APP_HOME=/app \
    PORT=8080 \
    DEBUG=false
#or
ENV APP_HOME=/app 
ENV PORT=8080
ENV DEBUG=false
```
- ENV sets environmental variables tht persist in the container. 
- It is used for configuration.

**3. RUN**:
```bash
# shell form
RUN apt-get update && \
    apt-get install -y python3
    
# exec form
RUN ["pip", "install", "numpy"]
```
- RUN executes commands during build. 
- It is best practice to combine commands to reduce layers. 

**4. COPY**:
```bash
COPY ./app /app/
COPY config.* /config/
```
- COPY simply copies from one path to the other one. 
- In this case, the first path, `./app` is the source directory on your host machine, relative to the Dockerfile's location. 
- The second path `/app/` is the destination directory inside the container (the absolute path).

Example directory structure
```bash
your_project/
├── Dockerfile
├── app/
│   ├── main.py
│   └── config.json
├── tests/
└── README.md
```
The current directory `.` is wherever your Dockerfile is located. If we run `docker build` from `your_project/`, this would become our build context, and all COPY paths are relative to that location.

**Note**: You can't COPY files from outside your build context. `COPY ../other_project/file.txt /app/` would fail because it tries to access files outside the build context. 

**5. WORKDIR**:
```bash
WORKDIR /app 
COPY . . # copies to /app now
```
- WORKDIR sets the working directory for subsequent commands. 
- If the directory does not exist, it creates it. 

**6. VOLUME**:
```bash
# single volume
VOLUME /data 

# multiple volumnes
VOLUME ["/data", "/logs", "/config"]
```
**7. CMD**:
```bash
# simple web server
CMD ['python', 'app.py']

# default command with arguments
CMD ['npm', 'start']
# could be overriden with: docker run myimage python other_script.py
```
- CMD specifies the default command to run when starting the container. 
- **It can be overriden from the command line when running the container.** For example:
```bash
# Case 1: Uses Dockerfile CMD
docker run myimage
# this will simply execute python app.py (our CMD instruction in the Dockerfile)

# Case 2: Overrides CMD completely
docker run myimage python other_script.py
# this will execute python other_script.py

docker run myimage echo "hello"
# this will echo: "hello"
```
### Building the Dockerfile
Once the Dockerfile is created, we need to build its corresponding Docker image. 
```bash
docker build -t image_name:version .

#example
docker build -t python_app:1.0 .
```
- Version is optional. 
- `.` specifies that Docker should look for the Dockerfile in the current directory.

**Recall that each line of the Dockerfile is a separate layer, or step**. If an error occurs at any given layer, Docker stops and shows the error message. One we solve the error, **Docker only rebuilds from the failed step onward**, saving time with cached layers. 
```bash
docker history image_name #shows the history of an image, listing all the layers that were created during its build process
```
## Docker Compose
Docker Compose is used to run multiple services.
- It uses a YAML file to configure application services: `docker-compose.yml`.
- It simplifies setup by specifying all dependencies and configurations in one file. 
```bash
docker-compose up #to start services
docker-compose down #to stop and remove services
docker-compose up --build #build and start
docker-compose up -d #start in detached mode
docker-compose logs -f #view logs
docker-compose ps #view running service
```

### Example from class
```bash
# Data Layers
docker run -d --name=redis redis #redis
docker run -d --name=db postgres #postgres

# Application Layers
docker run -d --name=vote -p 5000:80 voting-app
docker run -d --name=result -p 5001:80 result_app

#Worker
docker run d --name=worker worker #worker
```
The commands above create the necessary containers, defines the necessary names, and the necessary ports. 
It, however, **does not establish connections between containers**, which are needed for the application to work. 
```bash
# Linking commands
docker run -d --name=vote -p 5000:80 --link redis:redis voting_app
docker run -d --name=result -p 5001:80 --link db:db result_app 
docker run -d --name=worker --link redis:redis --link db:db worker
# link syntax: container name:name of the host inside the service
```
The networking done above can be simplified through a `docker-compose.yml` file:
```yaml
redis:
  image: redis
db:
  image: postgres:version
vote:
  image: voting-app
  ports:
    - 5000:80
  links:
    - redis
result: 
  image: result-app
  ports:
    - 5001:80
  links:
    - db
worker:
  image: worker
  links:
    - redis
    - db
```
There are 3 different version of Docker Compose:
1. Version 1: Basic, no advanced features. 
2. Version 2: Added network, volumes and service dependencies. Added `depends_on` instead of links. Version must be added at the beginning of the file. 
3. Version 3: Same config as version 2, but better for scaling and deployment. 

```yaml
#version 3 
version: 3
service:
  redis: 
    image: redis
    networks:
      - back-end
  db:
    image: postgres:version
    networks:
      - back-end
  vote:
    build: ./voting-app
    ports: 
      - 5000:80
    depends_on:
      - redis
    networks:
      - back-end
      - front-end
  result:
    build: ./result-app
    ports:
      - 5001:80
    depends_on:
      - db
    networks:
      - back-end
      - front-end
  worker:
    build: ./worker
    depends_on: 
      - redis
      - db
    networks:
      - back-end
networks:
  front-end
  back-end
```
## Docker System
Key components of Docker's layered system include:
- Images: Stored in `/var/lib/docker/images`. Represented by unique IDs. 
- Containers: Stored in `/var/lib/docker/containers`. Logs, metadata, and runtime configurations are stored here. 
- Volumes: Stored in `/var/lib/docker/volumes`. They are decoupled from containers for data persistence. 

**Recall that each line in a Dockerfile creates a layer in the Docker image**.

### Data Management in Containers

#### Creating New Data in Containers
- New data can be created inside the container. This data will only exist as long as the container is running. 
```bash
#start a container
docker run -it ubuntu bash

#inside container:
touch newfile.txt # create a new file
echo "data" > newfile.txt
mkdir newdir   # create a new directory
#these files/dirs only exist in the container's filesystem
```
#### Modifying Files from the Image
- Files inside images can be modified. First, we create a copy of the file inside the container:
```bash
# original container state from image
/app/
  ├── app.py (read-only from image)
  └── config.json (read-only from image)
  
# when you modify files:
docker run -it myapp bash

# inside container
cp /app/app.py /app/app.py.modified #create writable copy
vim /app/app.py.modifed #make changes
```
The original files in the image remain read-only; you work with copies in the container layer. 

#### Data Persistence
There are two main ways to persist data:
1. Volume

Create a Docker Volume, and mount the volume to the container. 
If the volume does not exist when using the `-v` flag, Docker will automatically create it. 
```bash
docker volume create new_volume
#creates a persistent volume named new_volume

docker run -v new_volume:/app/data myapp 
#docker run -v new_volume:path_inside_container image

# data in /app/data persists even after container is removed.
# this data can be reused by other containers.
```
2. Bind Mounts

Mount a host directory to the container. In this case, changes in the container will reflect in the host directory, and viceversa.
```bash
docker run -v /local/path:path_inside_container
```
**Example**:
```bash
# start container
docker run -it --name test ubuntu bash

# inside container:
echo "important data" > /data/myfile.txt
exit

# remove container
docker rm test

# data is gone! unless we used a volume:
docker run -it --name test -v mydata:/data ubuntu bash
# now /data/myfile.txt persists between container restarts/removals
```

## Docker Networking
- A network is a system of interconnected devices that can communicate and share data.
- Networks are composed of devices (computers, phones, servers, etc), Routers (direct traffic) and Protocols (rules for communication, e.g., TCP/IP).

An IP (Internet Protocol) address is a unique identifier for a device on a network. 
- IPv4: 192.168.1.1 (32-bit, most common).
- IPv6: 2001:0db8:85a3:0000:0000:8a2e:0370:7334 (128-bit, newer standard).

Devices use IP addresses to send and receive data.
Docker uses network modes to determine how containers connect to the outside world and to each other. 
- Bridge mode: Default mode, containers communicate through a private virtual network. 
- Host mode: Containers share the host machine's network. 
- None: No networking for the container. 

### Bridge Network (Default)
- Containers are connected to a private virtual network managed by Docker. 
- Best for containter-to-container communication, as each container gets its own IP address.
- Another key feature is internal DNS resolution between containers.

```bash
# list current networks
docker network ls 

# create custom bridge network
docker network create --driver bridge my_network

# run container with specific network and IP
docker run --network my_network --ip 172.18.0.10 nginx
```
**Note**: The resulting bridge network is isolated from other bridge networks. 

### Host Network 
- In host network mode, the container uses the same network as the host machine. It only works on Linux hosts. 
- Containers do not get their own IP address, they use the host's UP and ports. 
- Since there is port sharing between the host and the container, conflicts can occur.

```bash
docker run --network host nginx
# the nginx container would be accessible at the host's IP address.
```

### None Network
- In none mode, the container is isolated from all networks, and no network interfaces are assigned. 
- Use case: Highly secure containers don't need network connectivity. 
```bash
docker run --network none nginx
```

### Custom Docker Networks
Custom docker networks enhance the control over container communication. They also create better security and isolation. 

```bash
docker network create my_custom_network
docker network inspect <network_name>
```
Example: You have a frontend container and a backend container that need to communicate. 
```bash
# create a custom network:
docker network create app_network

# connect containers to the network
docker run --network app_network --name frontend frontend_image
docker run --network app_network --name backend backend_image

# verify the communication
docker exec -it frontend ping backend
```
### Network Managements Commands
```bash
# create network
docker network create my_network
## you can also pass --driver, --subnet, --gateway flags

# remove network
docker network rm my_network

# show network details and debug network issues
docker network inspect my_network

#check container networking
docker inspect container_name | grep IPAddress

# test container connectivity
docker exec container_name ping other_container
```
### Docker Hub and Private Repositories
Docker Hub allows creating private repositories to restrict access.
Steps to upload:
```bash
# login to Docker hub from the terminal
docker login 

# tag the image with your Docker Hub username
docker tag my_image username/repository:tag
docker push username/repository:tag

#example
docker tag my_app john123/my_app:v1
docker push john123/my_app:v1
```
Recall that private registries, self-hosted solutions for storing Docker images, can be used.
```bash
#start the registry container
docker run -d -p 5000:5000 --name registry registry:2

# tag and push an image
docker tag my_image localhost:5000/my_image
docker push localhost:5000/my_image

# pull the image from your registry
docker pull localhost:5000/my_image
```
## Docker Orchestration
### Why Use Docker Orchestration
- Manage multiple containers across multiple hosts. 
- Ensure availability and scalability. 
- Automate failover and recovery. 
- Streamline complex workflows in production.

### Key Concepts in Orchestration
- Cluster: A group of machines (physical or virtual) working together. 
- Nodes: Machines in the cluster (master and worker nodes).
- Services: Define how containers are deployed and accessed.
- Tasks: Individual instances of a container.

### Popular Orchestration Tools
- Docker Swarm: Built-in tool by Docker. Simple and tightly integrated with Docker CLI. 
```bash
# init a swarm
docker swarm init

# add worker nodes to the swarm
docker swarm join-token worker

# deploy a service
docker service create --replicas 3 nginx

# view the running services
docker service ls
```
- Kubernetes: Industry-standard. Supports advanced features like auto-scaling and self-healing.

    - Self-healing: Restarts failed containers. 
    - Rolling updates and rollbacks.
    - Auto-scaling and load balancing. 

- Basic Concepts:

    - Pods: Smallest deployable unit, can contain one or more containers. 
    - Deployments: Define the desired state of your application. 
    - Services: Expose your pods to the network
```bash
# create the deployment
kubectl create deployment my-app --image=nginx

#expose the deployment
kubectl expose deployment my-app --type=NodePort --port=80

#scale the application
kubectl scale deployment my-app --replicas=3

# view running pods
kubectl get pods
```
- Others: Apache Mesos, Nomad

