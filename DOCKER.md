# Docker 

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


