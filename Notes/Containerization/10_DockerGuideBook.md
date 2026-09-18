# Docker GuideBook --- Summer 2026 DevOps Journey

> **Personal revision guide:** This follows the Docker learning journey
> we actually covered, in learning order, using the real commands,
> project, mistakes, and fixes from the sessions. It is intentionally
> not a generic Docker handbook.

## 🧭 Quick Navigation

-   [1. Setup & Docker Engine](#1-setup--docker-engine)
-   [2. CLI, Daemon & Context](#2-cli-daemon--context)
-   [3. Images](#3-images)
-   [4. Tags & Digests](#4-tags--digests)
-   [5. Image Layers & Cache](#5-image-layers--cache)
-   [6. Containers & Lifecycle](#6-containers--lifecycle)
-   [7. Docker vs venv vs VM](#7-docker-vs-venv-vs-vm)
-   [8. Dockerfile](#8-dockerfile)
-   [9. Build Context & `.dockerignore`](#9-build-context--dockerignore)
-   [10. Running the Node App](#10-running-the-node-app)
-   [11. Ports](#11-ports)
-   [12. Logs & Exec](#12-logs--exec)
-   [13. MongoDB + Mongo Express](#13-mongodb--mongo-express)
-   [14. Docker Networking](#14-docker-networking)
-   [15. Network Drivers](#15-network-drivers)
-   [16. Docker Compose](#16-docker-compose)
-   [17. Volumes](#17-volumes)
-   [18. Named Volumes vs Bind Mounts](#18-named-volumes-vs-bind-mounts)
-   [19. `docker volume prune`](#19-docker-volume-prune)
-   [20. Docker Hub / Registries](#20-docker-hub--registries)
-   [21. Real Errors & Fixes](#21-real-errors--fixes)
-   [22. Complete Command Cheat Sheet](#22-complete-command-cheat-sheet)
-   [23. Current Project Checkpoint](#23-current-project-checkpoint)
-   [24. One-Minute Mental Model](#24-one-minute-mental-model)
-   [25. Next Topics](#25-next-topics)

------------------------------------------------------------------------

# 1. Setup & Docker Engine

## Our environment

-   Ubuntu 24.04.x LTS
-   Docker Engine 29.7.2
-   Docker Compose 5.x
-   Linux laptop
-   Main practice project:

``` text
/media/imranbuttcodes/Data/DevOps_Learning/docker-test-app/docker-testapp-main
```

## Docker's problem

Docker addresses the classic:

> "It works on my machine."

An application may depend on a particular runtime version, packages,
system libraries, filesystem layout, configuration, and other services.

The core pipeline:

``` text
Dockerfile
    │
    │ docker build
    ▼
Image
    │
    │ docker run
    ▼
Container
    │
    ▼
Application
```

**Remember:**

> Dockerfile → Image → Container

------------------------------------------------------------------------

# 2. CLI, Daemon & Context

## Docker CLI

The command:

``` bash
docker
```

is the command-line interface.

Example:

``` bash
docker run nginx
```

The CLI sends a request to Docker Engine.

## Docker daemon

A **daemon** is a background process/service that waits for requests and
performs work.

Docker's Engine/daemon manages:

-   images
-   containers
-   networks
-   volumes
-   builds
-   container lifecycle

Mental model:

``` text
Terminal
   │
   ▼
Docker CLI
   │ request
   ▼
Docker Engine / daemon
   │
   ├── images
   ├── containers
   ├── networks
   ├── volumes
   └── builds
```

> **CLI asks. Daemon does.**

## Commands

``` bash
docker version
docker info
docker context ls
```

`docker version` showed Client and Server, confirming the CLI was
communicating with the Engine.

Our active context was:

``` text
default *
```

and used:

``` text
unix:///var/run/docker.sock
```

## Docker socket permissions

We previously encountered Docker socket permission problems.

Fix:

``` bash
sudo usermod -aG docker $USER
newgrp docker
```

Logging out/in or rebooting also applies the new group membership.

After this, routine commands can be run without:

``` bash
sudo docker ...
```

------------------------------------------------------------------------

# 3. Images

## What is an image?

An image is a packaged, mostly read-only template used to create
containers.

``` text
Image = blueprint/template
Container = instance
```

One image can create multiple containers:

``` text
             testapp:1.0
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
    Container  Container  Container
```

## List images

``` bash
docker images
```

Newer Docker output may combine repository and tag into the `IMAGE`
field and show columns such as:

``` text
IMAGE
ID
DISK USAGE
CONTENT SIZE
EXTRA
```

Do not assume every tutorial has the same output layout.

## Images encountered

Our machine contained images such as:

``` text
docker.n8n.io/n8nio/n8n:latest
ghcr.io/n8n-io/n8n-sandbox-service-api:latest
ghcr.io/n8n-io/n8n-sandbox-service-runner-dind:latest
ghcr.io/searxng/searxng:latest
hello-world:latest
mongo-express:latest
mongo:7.0
mongo:latest
myserver:latest
mysql:latest
nginx:latest
postgres:16
testapp:1.0
```

------------------------------------------------------------------------

# 4. Tags & Digests

## Image reference

Normal form:

``` text
IMAGE:TAG
```

Examples:

``` text
nginx:latest
postgres:16
python:3.12
python:3.12-slim
python:3.12-bookworm
testapp:1.0
```

Without a tag:

``` bash
docker pull nginx
```

normally means:

``` text
nginx:latest
```

## `latest`

`latest` is a tag maintained by the publisher.

It is **not an immutable version** and can move.

## Our tags

``` bash
docker build -t testapp:1.0 .
docker build -t testapp:1.1 .
```

## `docker tag`

``` bash
docker tag testapp:1.0 imranbuttcodes/testapp:1.0
```

This creates another reference/name for the image.

## Digest

An image digest looks like:

``` text
sha256:...
```

Mental model:

``` text
Tag
testapp:1.0
    ↓
human-friendly reference; can move

Digest
sha256:...
    ↓
exact image-content identity
```

------------------------------------------------------------------------

# 5. Image Layers & Cache

Images consist of layers.

``` text
┌─────────────────────────┐
│ Application files       │
├─────────────────────────┤
│ Dependencies            │
├─────────────────────────┤
│ Base image              │
└─────────────────────────┘
```

Layers can be shared/reused, reducing storage and download/build work.

## Build-cache ordering

We learned the useful pattern:

``` dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

Why?

If source code changes but dependencies do not, Docker can often reuse
the dependency-installation cache.

``` text
requirements unchanged
        │
        ▼
dependency layer reused
        │
        ▼
only later application layer rebuilds
```

## Container writable layer

Conceptually:

``` text
Container
┌─────────────────────────┐
│ Writable layer          │
├─────────────────────────┤
│ Image layer             │
├─────────────────────────┤
│ Image layer             │
└─────────────────────────┘
```

Deleting the container removes its writable layer, while the image
remains.

Important data should therefore be placed in persistent storage such as
volumes.

------------------------------------------------------------------------

# 6. Containers & Lifecycle

## Container

A container is an instance created from an image.

``` text
Image
  │
  │ docker run
  ▼
Container
```

## List running containers

``` bash
docker ps
```

## List all containers

``` bash
docker ps -a
```

## Lifecycle

``` text
docker run
    │
    ▼
created
    │
    ▼
running
    │
    ├── exits
    └── crashes
    ▼
stopped
    │
    │ docker rm
    ▼
deleted
```

> **Stopped ≠ deleted**

## Remove

``` bash
docker rm <container>
```

## Force remove

We used:

``` bash
docker rm -f running-app
```

This was used to remove the old Node container before recreating it with
correct port publishing.

------------------------------------------------------------------------

# 7. Docker vs venv vs VM

## Python venv

`venv` isolates Python-level packages.

`requirements.txt` describes Python dependencies.

This solves part of dependency isolation.

## Docker

Docker provides a broader application environment, including things such
as:

-   runtime
-   application dependencies
-   system packages/libraries
-   filesystem environment
-   networking
-   startup configuration

Example problem:

``` text
Python 3.12
FastAPI
FFmpeg
libpq/system libraries
```

A `requirements.txt` file does not automatically install every
system-level dependency.

## Docker vs VM

VM:

``` text
Host OS
   │
Hypervisor
   │
Guest OS
   │
Guest kernel
   │
Application
```

Linux container:

``` text
Host Linux
   │
Host Linux kernel
   │
Container runtime
   │
Container user space
   │
Application
```

Linux containers normally share the host Linux kernel while maintaining
isolated user-space views.

## Kernel

The kernel is the core OS component managing resources such as:

-   CPU/processes
-   memory
-   filesystems
-   networking
-   devices
-   permissions/security

Applications communicate with it through system calls.

------------------------------------------------------------------------

# 8. Dockerfile

A Dockerfile contains instructions for building an image.

Our cleaner Node pattern:

``` dockerfile
FROM node:24

WORKDIR /testapp

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5050

CMD ["node", "server.js"]
```

## `FROM`

``` dockerfile
FROM node:24
```

Selects the base image.

## `ENV`

Our original Dockerfile contained:

``` dockerfile
ENV MONGO_DB_USERNAME=admin    MONGO_DB_PWD=qwerty
```

`ENV` defines environment variables.

It does **not** create MongoDB.

Do not bake real production credentials into an image.

## `RUN`

``` dockerfile
RUN npm install
```

Runs at **image build time**.

## `COPY`

``` dockerfile
COPY . .
```

Copies files from the build context into the image.

## `WORKDIR`

``` dockerfile
WORKDIR /testapp
```

Sets the working directory for subsequent Dockerfile instructions and
the container process.

## `EXPOSE`

``` dockerfile
EXPOSE 5050
```

Documents the intended application port.

**It does not publish the port to the host.**

## `CMD`

``` dockerfile
CMD ["node", "server.js"]
```

Defines the default startup command.

Remember:

``` text
RUN → build time
CMD → container startup
```

## Original Dockerfile we analyzed

``` dockerfile
FROM node

ENV MONGO_DB_USERNAME=admin    MONGO_DB_PWD=qwerty

RUN mkdir -p testapp

COPY . /testapp

CMD ["node", "/testapp/server.js"]
```

We then understood why `WORKDIR` and dependency-first copying make the
Dockerfile cleaner.

------------------------------------------------------------------------

# 9. Build Context & `.dockerignore`

## Build

We used:

``` bash
docker build -t testapp:1.0 .
```

The final:

``` text
.
```

means:

> Use the current directory as the build context.

## Build context

Our project:

``` text
docker-testapp-main/
├── Dockerfile
├── package.json
├── package-lock.json
├── server.js
└── public/
```

The context is the set of files Docker can access for the build.

## `.dockerignore`

Recommended for our Node project:

``` text
node_modules
.git
.env
npm-debug.log
```

This avoids sending unnecessary files/secrets as build context.

## Linux filename case

We encountered:

``` text
DockerFile
```

but Docker expects the conventional:

``` text
Dockerfile
```

On Linux, filenames are case-sensitive.

Fix:

``` bash
mv DockerFile Dockerfile
```

------------------------------------------------------------------------

# 10. Running the Node App

Our project contained:

``` text
Dockerfile
mongo.yaml
node_modules
package.json
package-lock.json
public
README.md
server.js
```

We built:

``` bash
docker build -t testapp:1.0 .
```

## Interactive shell

This worked:

``` bash
docker run -it testapp:1.0 sh
```

We reached:

``` text
/testapp #
```

This lets us inspect the container interactively.

## Why `bash` failed

We tried:

``` bash
docker run -it testapp:1.0 bash
```

and got a Node error similar to:

``` text
Cannot find module '/testapp/bash'
```

The Node image has an entrypoint that can cause the supplied `bash`
argument to be interpreted by Node.

Also, minimal images may not contain Bash.

`sh` worked:

``` bash
docker run -it testapp:1.0 sh
```

This introduced the next important topic:

> `CMD` and `ENTRYPOINT` are different and should be learned explicitly.

------------------------------------------------------------------------

# 11. Ports

## The fundamental distinction

Suppose Node listens inside the container on:

``` text
5050
```

That does not automatically publish the port to the host.

## `-p`

Syntax:

``` bash
-p HOST_PORT:CONTAINER_PORT
```

Example:

``` bash
docker run -p 5050:5050 testapp:1.0
```

Meaning:

``` text
Host:5050
    │
    ▼
Container:5050
    │
    ▼
Node
```

Different ports are possible:

``` bash
docker run -p 9000:8000 my-api
```

means:

``` text
localhost:9000 → container:8000
```

## Our real error

We initially ran:

``` bash
docker run --name running-app testapp:1.0
```

Node printed:

``` text
server running on port 5050
```

but the browser could not reach:

``` text
http://localhost:5050
```

`docker ps` showed no published port for `running-app`.

### Fix

``` bash
docker rm -f running-app
```

then:

``` bash
docker run -d   --name running-app   -p 5050:5050   testapp:1.0
```

Then:

``` text
0.0.0.0:5050->5050/tcp
```

appeared in `docker ps`.

### Core rule

> Container port ≠ automatically published host port.

------------------------------------------------------------------------

# 12. Logs & Exec

## Logs

``` bash
docker logs <container>
```

Example:

``` bash
docker logs running-app
```

Follow logs:

``` bash
docker logs -f running-app
```

Last 50 lines:

``` bash
docker logs --tail 50 running-app
```

Timestamps:

``` bash
docker logs -t running-app
```

## Exec

Run a command inside a running container:

``` bash
docker exec <container> <command>
```

Interactive shell:

``` bash
docker exec -it running-app sh
```

## Difference

``` text
docker logs
    ↓
see main-process output

docker exec
    ↓
run a command inside the running container
```

Debug workflow:

``` text
docker ps
   ↓
docker logs
   ↓
docker exec
   ↓
inspect
   ↓
fix Dockerfile/config
   ↓
rebuild
   ↓
recreate
```

------------------------------------------------------------------------

# 13. MongoDB + Mongo Express

We used MongoDB and Mongo Express to learn realistic multi-container
networking.

Architecture:

``` text
┌────────────── Docker network ──────────────┐
│                                            │
│  MongoDB  ◄──────────────► Mongo Express   │
│  mongo:27017                                │
│                                            │
└────────────────────────────────────────────┘
```

## Manual Mongo Express command

Our first attempt used the wrong hostname:

``` bash
docker run -d   -p 8081:8081   --name mongo-express   --network mongo-network   -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin   -e ME_CONFIG_MONGODB_ADMINPASSWORD=qwerty   -e ME_CONFIG_MONGODB_URL="mongodb://admin:qwerty@mongo-27017"   mongo-express
```

### Problem

This:

``` text
mongo-27017
```

was incorrectly treating hostname and port as one name.

## Correct command

``` bash
docker run -d   -p 8081:8081   --name mongo-express   --network mongo-network   -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin   -e ME_CONFIG_MONGODB_ADMINPASSWORD=qwerty   -e ME_CONFIG_MONGODB_URL="mongodb://admin:qwerty@mongo:27017"   mongo-express
```

Correct:

``` text
hostname = mongo
port     = 27017
```

------------------------------------------------------------------------

# 14. Docker Networking

## Why networking?

Containers need to communicate.

Example:

``` text
Node
 │
 │ MongoDB protocol
 ▼
MongoDB
```

## Create network

``` bash
docker network create mongo-network
```

## List networks

``` bash
docker network ls
```

## Inspect network

``` bash
docker network inspect mongo-network
```

## Connect a container

``` bash
docker run --network mongo-network ...
```

## Container-to-container communication

When containers share a Docker network, they can communicate using
Docker-provided DNS/service/container names.

Example:

``` text
Node container
      │
      │ Docker network
      ▼
Mongo container
```

Use:

``` text
mongo:27017
```

not:

``` text
localhost:27017
```

## Why `localhost` is wrong

Inside a container:

``` text
localhost
127.0.0.1
```

means:

> **this container itself**

Therefore:

``` text
Node container
localhost:27017
       ↓
Node container itself ❌
```

Whereas:

``` text
Node container
mongo:27017
       ↓
Mongo container ✅
```

## Host port vs internal port

If Mongo has:

``` text
-p 27017:27017
```

the host can use:

``` text
localhost:27017
```

But another container on the same Docker network normally uses:

``` text
mongo:27017
```

It does not need to go through the host-published port.

------------------------------------------------------------------------

# 15. Network Drivers

## What is a driver?

A network **driver tells Docker how a network is implemented/behaves**.

Do not confuse:

``` text
Network = actual Docker network
Driver  = mechanism used to implement it
```

Example:

``` text
mongo-network
    │
    ├── Driver: bridge
    └── Containers
```

## `bridge`

The important driver for our current work.

Create explicitly:

``` bash
docker network create --driver bridge my-network
```

Normal bridge networking lets containers on the same network
communicate.

## `host`

Example:

``` bash
docker run --network host nginx
```

The container uses host networking more directly and has less normal
network isolation.

## `none`

Example:

``` bash
docker run --network none nginx
```

Provides no normal network connectivity.

## `overlay`

Important for networking across multiple Docker hosts, such as Swarm
scenarios.

Conceptually:

``` text
Docker Host A              Docker Host B
     │                           │
     └────── overlay ───────────┘
```

### Driver table

  Driver      Main idea
  ----------- ------------------------------------
  `bridge`    Normal Docker container networking
  `host`      Host networking
  `none`      No normal networking
  `overlay`   Network across Docker hosts

------------------------------------------------------------------------

# 16. Docker Compose

## Why Compose?

Manually running multiple containers requires many commands.

Compose lets us define the application in YAML.

Example:

``` text
Node
MongoDB
Mongo Express
```

## Modern command

Use:

``` bash
docker compose
```

rather than relying on the old:

``` bash
docker-compose
```

## Our `mongo.yaml`

``` yaml
services:
  mongo:
    image: mongo:7.0
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty
    volumes:
      - mongo-data:/data/db

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: qwerty
      ME_CONFIG_MONGODB_URL: "mongodb://admin:qwerty@mongo:27017/"

volumes:
  mongo-data:
```

## Start

``` bash
docker compose -f mongo.yaml up
```

Detached:

``` bash
docker compose -f mongo.yaml up -d
```

## Check services

``` bash
docker compose -f mongo.yaml ps
```

## Logs

``` bash
docker compose -f mongo.yaml logs
```

## Exec

``` bash
docker compose -f mongo.yaml exec mongo bash
```

## Service

A Compose **service** is a Compose-level application component
definition.

A service commonly results in managed container(s), but:

> **Service ≠ container**

## Compose networking

Compose normally creates an application network automatically.

Services can communicate using service names.

Example:

``` text
mongo-express
      │
      │ mongodb://mongo:27017
      ▼
mongo
```

## `version`

Older tutorials may contain:

``` yaml
version: '3.8'
```

Modern Compose considers the top-level `version` field obsolete/ignored,
so it can normally be omitted.

## Stop vs down

``` bash
docker compose stop
```

stops services.

``` bash
docker compose down
```

removes the Compose containers and network.

Named volumes can remain unless explicitly removed.

------------------------------------------------------------------------

# 17. Volumes

## The problem

Container writable storage is disposable.

If Mongo stores data only in its container:

``` text
Mongo container
    │
    └── database data
```

and the container is deleted:

``` bash
docker rm -f mongo
```

that writable layer disappears.

## Volume

A Docker volume provides persistent storage outside the container
writable layer.

``` text
Mongo container
      │
      ▼
mongo-data
      │
      ▼
Docker-managed disk storage
```

## Create a volume

``` bash
docker volume create mongo-data
```

This creates **only the volume object**.

It does not:

-   create MongoDB
-   create a container
-   start MongoDB

## List

``` bash
docker volume ls
```

## Inspect

``` bash
docker volume inspect mongo-data
```

Our local Docker Engine showed a mountpoint similar to:

``` text
/var/lib/docker/volumes/mongo-data/_data
```

## Does creating a volume reserve fixed capacity?

No.

It does not reserve a fixed amount such as 10 GB.

It consumes actual disk space as data is written, subject to the
available filesystem/storage.

## Mount a named volume

``` bash
docker run -d   --name mongo-volume-test   -v mongo-data:/data/db   -e MONGO_INITDB_ROOT_USERNAME=admin   -e MONGO_INITDB_ROOT_PASSWORD=qwerty   mongo:7.0
```

Syntax:

``` text
-v VOLUME_NAME:CONTAINER_PATH
```

For Mongo:

``` text
mongo-data:/data/db
```

------------------------------------------------------------------------

# 18. Named Volumes vs Bind Mounts

## Named volume

``` bash
-v mongo-data:/data/db
```

Docker manages the storage location.

Good fit:

``` text
database/application persistent data
```

## Bind mount

Example:

``` bash
-v /home/imran/project:/app
```

This explicitly maps a host path to a container path.

Good fit:

``` text
development/source-code access
```

## Mental model

``` text
Named volume
Docker-managed storage
        │
        ▼
mongo-data:/data/db
```

``` text
Bind mount
Explicit host path
        │
        ▼
/home/imran/project:/app
```

## Compose creates named volumes automatically

``` yaml
services:
  mongo:
    image: mongo:7.0
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

Then:

``` bash
docker compose up -d
```

creates the named volume if needed.

------------------------------------------------------------------------

# 19. `docker volume prune`

Command:

``` bash
docker volume prune
```

This removes volumes that are currently unused by containers.

### Critical distinction

``` text
Unused
   ≠
Unimportant
```

A volume may contain valuable database data even when no current
container is using it.

So:

``` bash
docker volume prune
```

can delete important data if that data is sitting in an unused volume.

Before cleanup:

``` bash
docker volume ls
```

For an important volume:

``` bash
docker volume inspect mongo-data
```

------------------------------------------------------------------------

# 20. Docker Hub / Registries

## Registry mental model

``` text
GitHub
   ↓
source code

Docker Registry
   ↓
built Docker images
```

## Login

``` bash
docker login
```

## Tag for Docker Hub

Pattern:

``` text
USERNAME/REPOSITORY:TAG
```

Our image:

``` bash
docker tag testapp:1.0 imranbuttcodes/testapp:1.0
```

## Push

``` bash
docker push imranbuttcodes/testapp:1.0
```

We successfully pushed the image.

## Pull

``` bash
docker pull imranbuttcodes/testapp:1.0
```

Then another machine can run it.

## What gets pushed?

The image contains what was built into it:

-   runtime
-   installed dependencies
-   application files
-   image filesystem

It does not automatically include:

-   a separately running MongoDB container
-   Docker volumes
-   external databases
-   runtime secrets
-   external cloud services

------------------------------------------------------------------------

# 21. Real Errors & Fixes

## Error 1 --- Wrong Mongo hostname

Wrong:

``` text
mongodb://admin:qwerty@mongo-27017
```

Correct:

``` text
mongodb://admin:qwerty@mongo:27017
```

Reason:

``` text
hostname = mongo
port     = 27017
```

------------------------------------------------------------------------

## Error 2 --- `localhost` inside container

Wrong for Node → Mongo in separate containers:

``` text
mongodb://admin:qwerty@localhost:27017
```

Correct when both are on the same Docker network:

``` text
mongodb://admin:qwerty@mongo:27017
```

Reason:

``` text
localhost = current container
```

------------------------------------------------------------------------

## Error 3 --- Port 5050 connection refused

We ran:

``` bash
docker run --name running-app testapp:1.0
```

Node listened on:

``` text
5050
```

but host port 5050 was not published.

Fix:

``` bash
docker rm -f running-app
```

``` bash
docker run -d   --name running-app   -p 5050:5050   testapp:1.0
```

------------------------------------------------------------------------

## Error 4 --- `DockerFile`

Wrong:

``` text
DockerFile
```

Correct:

``` text
Dockerfile
```

Fix:

``` bash
mv DockerFile Dockerfile
```

------------------------------------------------------------------------

## Error 5 --- `bash` became a Node argument

We ran:

``` bash
docker run -it testapp:1.0 bash
```

and received a Node module error.

Reason: the image's entrypoint affected how `bash` was interpreted, and
the image may not contain Bash.

Working command:

``` bash
docker run -it testapp:1.0 sh
```

------------------------------------------------------------------------

## Error 6 --- Compose volume indentation

The volume mount belongs under the Mongo service:

``` yaml
services:
  mongo:
    ...
    volumes:
      - mongo-data:/data/db
```

The named volume declaration belongs at the top level:

``` yaml
volumes:
  mongo-data:
```

Correct structure:

``` yaml
services:
  mongo:
    ...
    volumes:
      - mongo-data:/data/db

  mongo-express:
    ...

volumes:
  mongo-data:
```

------------------------------------------------------------------------

# 22. Complete Command Cheat Sheet

## Engine / information

``` bash
docker version
docker info
docker context ls
```

## Images

``` bash
docker images
docker pull nginx
docker build -t testapp:1.0 .
docker build -t testapp:1.1 .
docker tag testapp:1.0 imranbuttcodes/testapp:1.0
docker push imranbuttcodes/testapp:1.0
docker pull imranbuttcodes/testapp:1.0
```

## Containers

``` bash
docker run nginx
docker run -d nginx
docker run -it testapp:1.0 sh

docker ps
docker ps -a

docker rm <container>
docker rm -f <container>

docker logs <container>
docker logs -f <container>
docker logs --tail 50 <container>
docker logs -t <container>

docker exec <container> <command>
docker exec -it <container> sh
```

## Ports

``` bash
docker run -p 5050:5050 testapp:1.0
docker run -p 9000:8000 my-api
```

Remember:

``` text
-p HOST_PORT:CONTAINER_PORT
```

## Networks

``` bash
docker network ls
docker network create mongo-network
docker network create --driver bridge my-network
docker network inspect mongo-network
docker run --network mongo-network ...
```

## Volumes

``` bash
docker volume create mongo-data
docker volume ls
docker volume inspect mongo-data
docker volume prune
```

Mount:

``` bash
docker run -v mongo-data:/data/db ...
```

Bind mount:

``` bash
docker run -v /home/imran/project:/app ...
```

## Compose

``` bash
docker compose up
docker compose up -d

docker compose -f mongo.yaml up
docker compose -f mongo.yaml up -d

docker compose -f mongo.yaml ps
docker compose -f mongo.yaml logs
docker compose -f mongo.yaml exec mongo bash

docker compose stop
docker compose down
```

## Docker permissions

``` bash
sudo usermod -aG docker $USER
newgrp docker
```

------------------------------------------------------------------------

# 23. Current Project Checkpoint

## Project path

``` text
/media/imranbuttcodes/Data/DevOps_Learning/docker-test-app/docker-testapp-main
```

## Files

``` text
Dockerfile
mongo.yaml
node_modules
package.json
package-lock.json
public
README.md
server.js
```

## Image

``` text
testapp:1.0
```

## Latest running setup

``` text
Node application
    image: testapp:1.0
    host:5050 → container:5050

MongoDB
    image: mongo:7.0
    host:27017 → container:27017

Mongo Express
    image: mongo-express
    host:8081 → container:8081
```

The current project was still a mixed setup:

``` text
Compose
 ├── Mongo
 └── Mongo Express

docker run
 └── Node application
```

The natural next step is to put the Node application into the same
Compose project.

------------------------------------------------------------------------

# 24. One-Minute Mental Model

``` text
                         Docker Engine
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
      Images              Containers             Networks
        │                     │                     │
        │                     │                     │
 Dockerfile                  App              Container ↔ Container
        │                     │                     │
        ▼                     ▼                     ▼
 docker build            docker run            bridge network
        │                     │
        ▼                     │
      Image                   │
        │                     │
        └──────────────┬──────┘
                       ▼
                   Container
                       │
             ┌─────────┼─────────┐
             │         │         │
           Ports     Volumes   Logs/exec
             │         │
             ▼         ▼
          Host ↔    Persistent
          Container   data
```

## The five core relationships

### 1. Dockerfile → Image

``` bash
docker build -t testapp:1.0 .
```

### 2. Image → Container

``` bash
docker run testapp:1.0
```

### 3. Host → Container

``` bash
docker run -p 5050:5050 testapp:1.0
```

### 4. Container → Container

``` text
mongo:27017
```

when both containers share a Docker network.

### 5. Container → Persistent data

``` bash
-v mongo-data:/data/db
```

------------------------------------------------------------------------

# 25. Next Topics

These are the next logical Docker lessons based on where we stopped.

## 1. Environment variables & secrets

Learn:

``` text
ENV
-e
.env
Compose environment
secrets
```

and why real credentials should not be baked into images.

## 2. CMD vs ENTRYPOINT

Directly connected to:

``` bash
docker run -it testapp:1.0 bash
```

Learn exactly how:

``` text
ENTRYPOINT
CMD
docker run IMAGE command
```

interact.

## 3. Complete Docker networking

Connect:

``` text
Node
Mongo
Mongo Express
```

inside one Compose network.

## 4. Complete Compose application

Target:

``` text
Browser
   │
   ▼
Node / Express
   │
   ▼
MongoDB
   │
   ▼
mongo-data
```

with Mongo Express available for administration.

## 5. Production Docker

Eventually connect Docker to deployment:

``` text
FastAPI / AI backend
        │
      Docker
        │
   Docker Compose
        │
      Nginx
        │
      HTTPS
        │
     Cloud/VPS
```

------------------------------------------------------------------------

# 🏁 Docker Learning Checkpoint

## Concepts covered

-   [x] Docker problem / "works on my machine"
-   [x] Docker Engine
-   [x] Docker CLI
-   [x] Docker daemon
-   [x] Docker context
-   [x] Docker socket
-   [x] Docker permissions
-   [x] Images
-   [x] Containers
-   [x] Container lifecycle
-   [x] Image tags
-   [x] `latest`
-   [x] Digests
-   [x] Image layers
-   [x] Build cache
-   [x] Docker vs `venv`
-   [x] Docker vs VM
-   [x] Linux kernel sharing
-   [x] Dockerfile
-   [x] `FROM`
-   [x] `ENV`
-   [x] `RUN`
-   [x] `COPY`
-   [x] `WORKDIR`
-   [x] `EXPOSE`
-   [x] `CMD`
-   [x] Build context
-   [x] `.dockerignore`
-   [x] Port publishing
-   [x] Logs
-   [x] `exec`
-   [x] MongoDB
-   [x] Mongo Express
-   [x] Docker networks
-   [x] Docker DNS / service names
-   [x] `localhost` inside containers
-   [x] Network drivers
-   [x] `bridge`
-   [x] `host`
-   [x] `none`
-   [x] `overlay`
-   [x] Docker Compose
-   [x] Compose services
-   [x] Compose networking
-   [x] Compose volumes
-   [x] Named volumes
-   [x] Bind mounts
-   [x] Volume persistence
-   [x] `docker volume prune`
-   [x] Docker Hub publishing
-   [x] Real Docker debugging

## Practical milestones

-   [x] Built `testapp:1.0`
-   [x] Entered the container with `sh`
-   [x] Ran the Node application
-   [x] Diagnosed missing host-port publishing
-   [x] Published `5050:5050`
-   [x] Ran MongoDB
-   [x] Ran Mongo Express
-   [x] Connected containers through networking
-   [x] Learned `mongo:27017`
-   [x] Learned why `localhost` means the current container
-   [x] Created and inspected `mongo-data`
-   [x] Verified Mongo data survives container deletion/recreation
-   [x] Moved volume configuration into Compose YAML
-   [x] Ran Mongo + Mongo Express with Compose
-   [x] Tagged and pushed `testapp:1.0`

------------------------------------------------------------------------

# 🧠 Final Vocabulary

``` text
Docker Engine → manages Docker resources
CLI           → sends commands
Daemon        → performs Docker work

Dockerfile    → image build instructions
Image         → packaged template
Container     → image instance

Port          → network endpoint
-p            → host:container port publishing

Network       → container communication
Driver        → mechanism implementing the network
bridge        → normal Docker networking

Volume        → persistent storage
Bind mount    → explicit host ↔ container path

Compose       → multi-container application definition

Registry      → stores Docker images
Tag           → human-friendly image reference
Digest        → exact image-content identity
```

> **This guide is a living checkpoint. Add new Docker concepts to it in
> the same learning order instead of turning it into an unrelated
> generic Docker tutorial.**
