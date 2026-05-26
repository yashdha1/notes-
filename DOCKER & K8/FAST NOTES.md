
Open container initiative -> open governence structure for the exoress purpose of creating open statndard around container formars and runtime. 

---

1. image is a bluprint for container 
	1. docker images -> list all the images in the system currently. 
	2. docker build -> create containers with reference to the current image. 
---
1.  docker is fast yo, any changes can be directly appilied to the running of the application. 
2. docker Hub has 1lakh+ public images.  
3. If you run image without pulling it first : docker automatically pulls the image from the docker registry and runs it. 
4. All of the docker containers have a port binded to them. called as port binding. the port given in shown in the configuration of the thingy. is the port number in the container of lighter images. eg. so a windows machine having a container running will have all the ports and in those ports have a container running inside the container. Done using the port binding.
5. docker images are Images are **read-only** layered snapshots (built from a Dockerfile). When you run a container, Docker adds a thin **writable layer** on top — that's your "live object." The image itself never changes, which is why you can spin up 10 containers from the same image simultaneously. The command? `docker run <image-name>` that's what flips blueprint → live container. 
6. chain multiple run commands into a single line with the && syntax. 
7. EXPOSE : does not actually publish on port thats the work of the -p in docker run. 
8. No chache in the docker build : Using `--no-cache` with `docker build` The simplest way: docker build --no-cache -t myimage:latest .
		- `--no-cache` → tells Docker **not to reuse any previous layers**.
		- `-t myimage:latest` → tags the image. 
		- `.` → build context (current directory).
		This ensures every step in your `Dockerfile` runs fresh.

9. *docker registry -mirrors* if you want to pull some images from the private docker images, use docker registry for the the name of the repository. 
10. 