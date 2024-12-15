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
- `p` (Port mapping): Maps a port on the host to a port on the container. `-p 8080:80` maps port 80 in the container to port 8080 on the host. 
- `-e` (Environment variables): Sets environment variables inside the container. 
- `--name`: Assigns a name to the container.
- `it` or `-i` (interactive) and `-t` (psuedo-terminal): Allows us to interact with the container's terminal. Useful for containers that need user input or have a shell interface.

### Creating a Dockerfule