https://docs.docker.com/get-started/ 
https://www.sysdig.com/learn-cloud-native/what-is-docker-architecture 

Containerisation strategies used to exists before the docker but they called it something as syscalls, cgroups and namespaces. 

| Feature        | What it does                                                  | Who uses it                        | Analogy               |
| -------------- | ------------------------------------------------------------- | ---------------------------------- | --------------------- |
| **Namespaces** | Isolates _what_ a process can see (PIDs, network, filesystem) | `clone()` syscall                  | Blinders on a horse   |
| **Cgroups**    | Limits _how much_ a process can use (CPU, RAM, I/O)           | `/sys/fs/cgroup` virtual files     | Prepaid utility meter |
| **Syscalls**   | The API for processes to request kernel work                  | `read`, `write`, `clone`, `socket` | Door to the kernel    |
| **Seccomp**    | Blocks _dangerous_ syscalls                                   | `seccomp` syscall                  | Bouncer at the door   |

---



*Containarzation wala scene. instead of VM =>*
- **Container** — A lightweight, isolated box that runs your app. Think of it like a mini-computer inside your computer, but much faster and leaner than a full virtual machine. runnable instance of the images. 
- **Image** — The blueprint/template used to create a container. You build an image once and can spin up many containers from it. Read only. usually docker images are based on  other images with additional customisation.  exec file that helps us to create containers. 
- **Dockerfile** — A simple text file with instructions for how to build your image (what OS, what dependencies to install, what code to include).

*VM verses DOCKER* 
1. primary difference between the VM And a Container is that the container only virtaulises the software layer over the hardware layer in the OS. thus you can run multiple containers over the operating system.   
2. Docker is a wrapper over the LXC. Docker keeps alot of stuff isolated and is easier to use. Linux Containers aim to offer a vender neutral open-source container runtime. And is meant neiche applications. 
3. Containers are usually static definitions of the expected dependencies and configuration needed to run the container. Virtual machines are more dynamic and can be interactively developed. Once the basic hardware definition is specified for a virtual machine the virtual machine can then be treated as a bare bones computer. Software can manually be installed to the virtual machine and the virtual machine can be snapshotted to capture the current configuration state. The virtual machine snapshots can be used to restore the virtual machine to that point in time or spin up additional virtual machines with that configuration.
4. Also docker is a process level application and does not require Full hardware level control over like the OS. 

![A container showing the differences between virtual machines and containers.](https://dam-cdn.atl.orangelogic.com/AssetLink/8rey324iqvflta48it3i685ys31yv4wj.png)

Popular containarization tools : Docker, RKT, Linux Containers(LXC).
Popular CM providers : VirtaulBOX, VMWare, QEMU.

1.  Docker allows running applications in a Isolated enviorments. The isolation and security let you run many containers simultaneously on a given host. Containers are lightweight and contain everything needed to run the application, so you don't need to rely on what's installed on the host. 
2. docker streamlines the development lifecycle by allowing developers to work in standardized environments using local containers which provide your applications and services. Containers are great for continuous integration and continuous delivery (CI/CD) workflows. 
3. Docker is lightweight and fast. It provides a viable, cost-effective alternative to hypervisor-based virtual machines, so you can use more of your server capacity to achieve your business goals. Docker is perfect for high density environments and for small and medium deployments where you need to do more with fewer resources.

What does DOCKER SOLVES? ISSUES WITHOUT DOCKER? 
1. Works on my machine fails on diffrent machine? due tp OS versions, Library dependencies and enviorment configrations. 
2. Dependency HELL : Diffrent apps need diffrent versions and might need diffrent libraries of dependencies. this docker needed.
3. Slow & Manual Deployments : Deploying software takes alot of coordination, manualy SSHing into the servers. running scripts by hand. No wasy rollbacks and transaction handelling. 
4. Inconsistent enviorment: Dev, testing, staging, and production environments were all set up differently by different people. A bug that only appeared in production was a nightmare to reproduce locally.
5. Monolithic Architecture :Everything is deployed in a single application style. 
6. Poor Colaboration and long realese cycles. 


.*Docker Architecture* :- 
Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface. Another Docker client is Docker Compose, that lets you work with applications consisting of a set of containers. ![[docker-architecture.webp]]


HIGH LEVEL OVERVIEW OF THE ARCHITECTURE OF DOCKER :  and the terminolgy in the docker. 
```
Developer (You)
     |
  Docker CLI (Client)
     |
     | docker cli sends commands to the dockerd 
     | 
  Docker Daemon (Server/Engine)
     |
     | directly has the control over the images , containers and volumes and network.
     | 
  ┌──────────────────────────────------------------ |
  │  Images  │  Containers  │  Volumes  │  Networks |
  └──────────────────────────────------------------ |
     |
     | 
     | 
  Docker Registry (Docker Hub)
```
---
 


*Docker Client* -> side of docker the dev interact with and type in commands like docker run and docker build. client sends these instructions to the Docker Daemon via a REST API. The client and daemon can run on the same machine or remotely to a server. 

*Docker Daemon* dockerd -> run in the backround and does all the heavylifiting. It runs in the background and does all the heavy lifting — building images, running containers, managing networks and volumes. It listens for API requests from the client and acts on them.  the CLI and other CLI tools interact to with the dockerD daemin for the docker features. 


*Docker Images* : Used to create containers and isolated enviorments. Images are built in **layers** — each instruction in a Dockerfile adds a new layer on top of the previous one. Layers are cached, making builds faster. BLUEPRINT OF CONTAINERS. 

*Docker Containers* : A running instance of an image. It's isolated, lightweight, and has its own filesystem, networking, and process space. You can run many containers from the same image. RUNNING INSTANCES of images after creation. 

*Docker Registry* : is a distribution system for docker images. **Docker Hub** is the default public registry. You can also host private registries. When you do `docker pull`, you're downloading an image from a registry.
*Docker Hub* : is a public registry that holds the all the public available repos. You can even run your private registry.  docker pull, docker run etc. 


*Volumes* :  are static storages for the docker containers. Persistent storage for containers. Containers are ephemeral — when they stop, data is gone. Volumes solve this by storing data outside the container on the host machine.

*Docker Networks* : Allow containers to communicate with each other and the outside world. Docker creates isolated virtual networks so containers can talk securely.  
![[Pasted image 20260223154030.png]]

**docker is written in GO!**


## *# Hypervisor* ->  
A hypervisor (or Virtual Machine Monitor, VMM) is software that lets multiple operating systems run on a single physical machine. It manages hardware resources (CPU, memory, storage) and allocates them to virtual machines (VMs) without interference. This improves hardware utilization, reduces costs, and provides flexibility in cloud and server environments. 

*Type 1* : Bare metal hypervisor : Runs **directly on the physical hardware** — no host operating system needed. best performance and speed due to the no intermediate OS and overhead. It's the standard for enterprise-level data centers and cloud providers like Amazon Web Services (AWS) and Microsoft Azure. eg. Xen, VMWare ESXi etc. 
-> fast, relailable and very secure. but requires dedicated hardware and setup and management is difficult. 

*Type 2* : Has the Host OS. runs on top of conventional OS. like windows and linux. Ususlly used where user want to run multiple OS  on top of each others. eg. using  VM Ware, oracle VM VirtualBox etc. Performance is slightly lower than Type 1 due to the overhead of the host OS.
-> easy to install and setup, fast deployments, host-guest integration. 
but slow, security might be lower. 
![[hypervisor.webp]] 


A Docker iceberg video can definitely be an engaging way to break down the complexity of Docker's features and components. We can layer the content from basics to more advanced topics, with each level uncovering deeper and more intricate details. Here’s a suggested breakdown for your Docker iceberg, starting with the surface-level concepts and diving down into the most intricate or specialized features of Docker.

---

**Docker engine working and High-Level Architecture**
-> Docker Client  - user perception of docker. 
-> Docker Daemon - backround service that does the actual working. 
-> Container Runtime - containerD manages the containers lifecycle.  

request cycles : 
Client ----sends a rest(request)----> docker daemon -------> 
	variety of tasks here  : 
	-----> run command or pull command -> pulls from local system. 
	-----> creates a writable layer on top of an image. 
	-----> uses kernel features ontop of already configured docker container. 
	-----> runc - starts the container main process and creates the isolated enviorment. 
		


## **DOCKER ROADMAP**
####  **Layer 1 (Surface) – Basic Concepts**

- **What is Docker?**
    - Containerization and Virtualization
    - Differences between VMs and containers
    - Advantages of Docker (portability, consistency, and efficiency)
- **Docker Images**
    - What is a Docker image?
    - Image layers and how they work
    - Docker Hub (public registry) and private registries
- **Docker Containers**
    - What is a container?
    - How containers are isolated from each other
    - Running and stopping containers (e.g., `docker run`, `docker stop`)
- **Docker CLI (Command Line Interface)**
    - Basic commands (e.g., `docker ps`, `docker run`, `docker build`)

---

### **Layer 2 – Intermediate Concepts**

- **Dockerfile**
    - What is a Dockerfile?
    - Common instructions (`FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`)
    - Building custom images with Dockerfile
    - Multi-stage builds
        
- **Docker Volumes**
		    
    - What are volumes?
    - Persistent storage in containers
    - Volume types (host volumes, named volumes, anonymous volumes)
        
- **Docker Networks**
		    
    - Networking modes: bridge, host, and none
    - Docker network commands and concepts
    - Communication between containers
        
- **Docker Compose**
		    
    - Introduction to Docker Compose
    - `docker-compose.yml` structure
    - Defining multi-container applications (e.g., connecting databases to web apps)
        
- **Docker Registry**
		    
    - What is a Docker registry?
    - Pushing and pulling images (e.g., `docker push`, `docker pull`)
        

---

### **Layer 3 – Advanced Concepts**

- **Docker Swarm** 
    - Introduction to Swarm mode
    - Managing clusters with Swarm
    - Services, nodes, and stacks in Swarm
    - Swarm’s orchestration features
        
- **Docker Networking in Detail**
	    
    - Custom networks and advanced bridging
    - Overlay networks for multi-host communication
    - Host networking vs. bridge networking
        
- **Docker Security**
    - User namespaces
    - Seccomp profiles and AppArmor
    - Securing Docker containers with resource limits
		
- **Docker Logging and Monitoring**
    
    - Managing logs with `docker logs`
    - External logging systems (e.g., ELK Stack, Fluentd)
    - Monitoring Docker containers with tools like Prometheus and Grafana
        
- **Docker with CI/CD**
    
    - Integrating Docker into Continuous Integration (CI) pipelines
    - Automating builds and deployments with Docker in CI tools like Jenkins, GitLab CI, or GitHub Actions
        

---

### **Layer 4 – Expert Concepts**

- **Docker in Production**
		
    - Scaling containers in production
    - Best practices for deploying Docker containers
    - Orchestrating with Kubernetes vs. Docker Swarm
        
- **Kubernetes (as a container orchestration system)**
    
    - Introduction to Kubernetes and its components (Pods, Nodes, Services, etc.)
    - How Kubernetes integrates with Docker (though Kubernetes often uses containerd now)
        
- **Docker Container Runtimes**
    
    - Container runtimes beyond Docker (e.g., containerd, CRI-O)
    - Differences between Docker and containerd
    - Docker Engine vs. other runtimes in orchestration systems like Kubernetes
        
- **Docker Buildx**
    
    - Advanced image building with Buildx
    - Multi-platform builds (e.g., building ARM64 and x86 images)
        
- **Docker in the Cloud**
    
    - Running Docker in AWS, GCP, Azure 
    - Using Docker with cloud-native technologies (e.g., ECS, EKS, Google Kubernetes Engine)
        

---

### **Layer 5 – Hidden Depths (Deep Dive)**

- **Docker Internals**
    
    - How Docker Engine interacts with the OS kernel
    - The union filesystem (OverlayFS)
    - cgroups, namespaces, and their role in containers
        
- **Containerized OS**
    
    - Introduction to lightweight OS like Alpine or CoreOS in Docker containers
    - Benefits and challenges of running a minimal OS inside a container
        
- **Container Security at a Low Level**
    
    - Linux capabilities and container security
    - Container escape vulnerabilities and mitigations
    - Runtime security tools (e.g., gVisor, Kata Containers)
        
- **Docker Networking in Kubernetes**
    
    - How Docker handles networking when Kubernetes is managing containers
    - CNI (Container Networking Interface) in Kubernetes
        
- **Docker Layer Caching and Performance**
    
    - Understanding Docker image caching
    - Optimizing Dockerfile for faster builds and smaller image
    - Docker layer squashing for reducing image size
        

---

### **Layer 6 – Extreme Depth (Theoretical or Specialized Knowledge)**

- **Docker on ARM Architecture**
    
    - Building and running Docker on ARM-based processors
    - Cross-platform builds for ARM and x86 architectures
    - Docker’s role in IoT (Internet of Things) with ARM-based devices
        
- **Docker in Edge Computing**
    
    - How Docker works for edge applications
        
    - Lightweight containerization for low-resource devices
        
- **Deep Dive into Docker's Networking Stack**
    
    - Docker’s low-level networking internals
        
    - Custom network drivers for specialized applications
        
- **Immutable Infrastructure with Docker**
    
    - The concept of immutable infrastructure in modern DevOps
        
    - How Docker supports immutability in CI/CD pipelines
        
- **Docker vs. Podman**
    
    - A comparison of Docker vs. Podman (rootless containers, daemonless architecture)
        
    - Why some prefer Podman over Docker
        

---