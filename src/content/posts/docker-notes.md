---
title: "Docker Notes"
published: 2025-09-12
description: "Comprehensive Docker commands, Dockerfile examples, and practical tips for DevOps and container management."
image: "/assets/docker-notes/cover.png"  
tags: [Docker, Container, DevOps]
category: "DevOps"
draft: false
lang: "en"
---
# Docker

### Docker Syntax Categories

1. Running a container
2. Managing & Inspecting containers
3. Managing Docker images
4. Docker daemon stats & info

---

## Managing Docker Images

### Pulling Images

Download an image with `docker pull IMAGE[:TAG]`.

If no tag is specified → defaults to `latest`.

```bash
docker pull nginx
docker pull ubuntu
docker pull ubuntu:22.04
docker pull ubuntu:20.04
docker pull ubuntu:18.04
```

**Tags:**

- `ubuntu:latest` → latest release (could change)
- `ubuntu:22.04` → Jammy
- `ubuntu:20.04` → Focal
- `ubuntu:18.04` → Bionic

---

### Docker Image Management

```bash
shohan@kali:~$ docker image

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
shohan@kali:~$
```

Example:

- pull (we have done this above!)
- ls (list images)
- rm (remove an image)
- build (we will come onto this in the “Building your First Container” task)

## **Docker Image ls**

This command allows us to list all images stored on the local system. We can use this command to verify if an image has been downloaded correctly and to view a little bit more information about it (such as the tag, when the image was created and the size of the image).

```jsx
shohan@kali:~$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       22.04     2dc39ba059dc   10 days ago   77.8MB
nginx        latest    2b7d6430f78d   2 weeks ago   142MB
shohan@kali:~$
```

| **Repository** | **Tag** | **Image ID** | **Size** |
| --- | --- | --- | --- |
| ubuntu | 22.04 | 2dc39ba059dc | 77.8MB |
| nginx | latest | 2b7d6430f78d | 142MB |

## **Docker Image rm**

If we want to remove an image from the system, we can use `docker image rm` along with the name (or Image ID). In the following example, I will remove the "*ubuntu*" image with the tag "*22.04*". My command will be `docker image rm ubuntu:22.04`:

It is important to remember to include the *tag* with the image name.

```jsx
shohan@kali:~$ docker image rm ubuntu:22.04
Untagged: ubuntu:22.04
Untagged: ubuntu@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1
Deleted: sha256:2dc39ba059dcd42ade30aae30147b5692777ba9ff0779a62ad93a74de02e3e1f
Deleted: sha256:7f5cbd8cc787c8d628630756bcc7240e6c96b876c2882e6fc980a8b60cdfa274
shohan@kali:~$
```

---

## Running Containers

### Syntax

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARGS...]
```

## RUN Container

```jsx
docker run -it helloworld /bin/bash
```

### Common Options

| Option | Description | Example |
| --- | --- | --- |
| `-it` | Interactive shell | `docker run -it helloworld /bin/bash` |
| `-d` | Detached mode | `docker run -d helloworld` |
| `-v` | Mount volume | `docker run -v /host:/container helloworld` |
| `-p` | Port mapping | `docker run -p 80:80 webserver` |
| `--rm` | Remove container after exit | `docker run --rm helloworld` |
| `--name` | Assign name | `docker run --name mycontainer helloworld` |

---

## Listing Containers

```bash
docker ps      # list running
docker ps -a   # list all (including stopped)

```

---

# Intro to Dockerfiles

Dockerfiles are instruction manuals for building Docker images.
They contain commands that define what a container should do.

Syntax:
INSTRUCTION argument

---
🔑 Core Instructions

```bash
| Instruction | Description | Example |
|-------------|------------|---------|
| FROM        | Base image | FROM ubuntu:22.04 |
| RUN         | Execute cmd| RUN whoami |
| COPY        | Copy files | COPY ./app/ /app/ |
| WORKDIR     | Set dir    | WORKDIR / |
| CMD         | Default cmd| CMD ["./script.sh"] |
```

---

🚀 Example 1: Simple Dockerfiles

Goal:

- Use Ubuntu 22.04
- Set working directory to root (/)
- Create a helloworld.txt file

Dockerfiles:

```yaml
# THIS IS A COMMENT
# Use Ubuntu 22.04 as the base operating system of the container
FROM ubuntu:22.04

# Set the working directory to the root of the container
WORKDIR /

# Create helloworld.txt
RUN touch helloworld.txt
```

Build the image:

```yaml
docker build -t helloworld .
```

```yaml
shohan@kali:~$ docker build -t helloworld .
Sending build context to Docker daemon  4.778MB
Step 1/3 : FROM ubuntu:22.04
22.04: Pulling from library/ubuntu
2b55860d4c66: Pull complete
Digest: sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1
Status: Downloaded newer image for ubuntu:22.04
 ---> 2dc39ba059dc
Step 2/3 : WORKDIR /
 ---> Running in 64d497097f8a
Removing intermediate container 64d497097f8a
 ---> d6bd1253fd4e
Step 3/3 : RUN touch helloworld.txt
 ---> Running in 54e94c9774be
Removing intermediate container 54e94c9774be
 ---> 4b11fc80fdd5
Successfully built 4b11fc80fdd5
Successfully tagged helloworld:latest
shohan@kali:~$
```

Check built images:

```
docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
helloworld   latest    4b11fc80fdd5   2 minutes ago   77.8MB
ubuntu       22.04     2dc39ba059dc   10 days ago     77.8MB
shohan@kali:~$
```

---

🌐 Example 2: Apache Web Server

Goal:

- Use Ubuntu 22.04
- Install Apache2
- Expose port 80
- Start Apache at container launch

Dockerfile:

```
# THIS IS A COMMENT
FROM ubuntu:22.04

# Update the APT repository to ensure we get the latest version of apache2
RUN apt-get update -y 

# Install apache2
RUN apt-get install apache2 -y

# Tell the container to expose port 80 to allow us to connect to the web server
EXPOSE 80 

# Tell the container to run the apache2 service
CMD ["apache2ctl", "-D","FOREGROUND"]
```

Build & Run:

```
docker build -t webserver .
docker run -d --name webserver -p 80:80  webserver
```

Open in browser: [http://localhost](http://localhost/)

![image.png](\public\assets\docker-notes\image.png)

---

⚡ Optimizing Dockerfiles

Why optimize?

- Smaller images
- Faster builds
- Easier to maintain

Techniques:

1. Install only what you need
2. Clean caches & temp files
3. Use minimal base images (e.g., alpine, ubuntu-minimal)
4. Reduce layers by chaining commands

❌ Before (many layers):

```
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install apache2 -y
RUN apt-get install net-tools -y
```

✅ After (fewer layers):

```
FROM ubuntu:latest
RUN apt-get update -y && \\
    apt-get upgrade -y && \\
    apt-get install -y apache2 net-tools
```

![image.png](\public\assets/docker-notes/image%201.png)

---

## Docker Compose

![image.png](\public\assets/docker-notes/image%202.png)

### Common Commands

```bash
docker-compose up
docker-compose start
docker-compose stop
docker-compose down
docker-compose build
```

### Example `docker-compose.yml`

```yaml
version: '3.3'
services:
  web:
    build: ./web
    networks:
      - ecommerce
    ports:
      - '80:80'

  database:
    image: mysql:latest
    networks:
      - ecommerce
    environment:
      - MYSQL_DATABASE=ecommerce
      - MYSQL_USERNAME=root
      - MYSQL_ROOT_PASSWORD=helloworld

networks:
  ecommerce:

```

---

# END
