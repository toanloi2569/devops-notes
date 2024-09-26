# Docker Cheat-sheet for beginners 🐳
Source: [Docker Cheat-sheet for beginners](https://dev.to/keshav___dev/docker-cheat-sheet-for-beginners-18mo?ref=dailydev)
  

### 🔧 Common Docker Commands
**Start Docker:**
```
systemctl start docker  # Linux
open -a Docker  # macOS
```
**Check Docker Version:**
```
docker --version
```
### 📦 Working with Containers
**List Running Containers:**
```
docker ps
```
**List All Containers (Running + Stopped):**
```
docker ps -a
```
**Run a Container (starts and attaches):**
```
docker run <image_name>
```
**Run in Detached Mode:**
```
docker run -d <image_name>
```
**Run with Port Mapping:**
```
docker run -p <host_port>:<container_port> <image_name>
```
**Stop a Running Container:**
```
docker stop <container_id>
```
**Start a Stopped Container:**
```
docker start <container_id>
```
**Remove a Stopped Container:**
```
docker rm <container_id>
```
### 📜 Images
**List Docker Images:**
```
docker images
```
**Pull an Image from Docker Hub:**
```
docker pull <image_name>
```
**Build an Image from Dockerfile:**
```
docker build -t <image_name> .
```
**Tag an Image:**
```
docker tag <image_id> <new_image_name>:<tag>
```
**Remove an Image:**
```
docker rmi <image_id>
```
### 🔄 Container Management
**View Logs of a Container:**
```
docker logs <container_id>
```
**Access a Running Container (Interactive Shell):**
```
docker exec -it <container_id> /bin/bash
```
**Copy Files from Container to Host:**
```
docker cp <container_id>:<path_inside_container> <host_path>
```
### 🏗 Docker Networks
**List Networks:**
```
docker network ls
```
**Create a Network:**
```
docker network create <network_name>
```
**Connect a Running Container to a Network:**
```
docker network connect <network_name> <container_id>
```
### 🐳 Docker Compose
**Start Services in Detached Mode:**
```
docker-compose up -d
```
**Stop Services:**
```
docker-compose down
```
**Build and Start Containers:**
```
docker-compose up --build
```
### 📊 Inspecting and Monitoring
**Inspect Container Details:**
```
docker inspect <container_id>
```
**Display Resource Usage (CPU, Memory):**
```
docker stats
```
### 🛠 Volumes
**List Volumes:**
```
docker volume ls
```
**Create a Volume:**
```
docker volume create <volume_name>
```
**Mount a Volume (during docker run):**
```
docker run -v <volume_name>:<path_inside_container> <image_name>
```

---
💡 Pro Tip: Use `docker system prune` to remove unused containers, networks, and images.



