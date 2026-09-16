# 🐳 My First Docker Full-Stack Architecture Guide

Welcome to my personal Docker learning journal! This document outlines exactly how I configured, debugged, and deployed an isolated, secure full-stack database environment on my ThinkPad T480s running Linux.

---

## 🗺️ System Architecture Overview

Instead of installing database engines directly onto my clean host Operating System, everything is sandboxed inside Docker using a custom virtual bridge network.

```text
 ┌────────────────────────────────────────────────────────────────────────┐
 │                      YOUR THINKPAD HOST OPERATING SYSTEM               │
 │                                                                        │
 │   ┌────────────────────────────────────────────────────────────────┐   │
 │   │               🔒 VIRTUAL NETWORK: mongo-network                 │   │
 │   │                                                                │   │
 │   │   ┌─────────────────────┐            ┌─────────────────────┐   │   │
 │   │   │ CONTAINER: mongo    │            │ CONTAINER:          │   │   │
 │   │   │                     │            │ mongo-express       │   │   │
 │   │   │ Database Server     │◄───────────┤                     │   │   │
 │   │   │ (Port 27017)        │  Connected │ Web Dashboard UI    │   │   │
 │   │   └──────────┬──────────┘            └──────────┬──────────┘   │   │
 │   └──────────────┼──────────────────────────────────┼──────────────┘   │
 │                  │                                  │                  │
 │    Tunnel Gateway│                    Tunnel Gateway│                  │
 │                  ▼                                  ▼                  │
 │          Port 27017                         Port 8081                  │
 │   (For internal apps/code)            (For your Web Browser)           │
 └──────────────────┴──────────────────────────────────┴──────────────────┘
```

---

## 🎛️ How the Network Ports Connect

When running this system, my Node.js server sits in the middle, managing two separate lines of communication simultaneously:

1. **The Incoming Door (Port 5050):** My Node/Express application listens here. When a browser visits `http://localhost:5050/getUsers`, it talks directly to my Node code.
2. **The Outgoing Door (Port 27017):** When Node receives a request, it makes a client connection *outward* to `localhost:27017`. Docker intercepts this and tunnels the connection straight into the `mongo` container.

---

## 🛠️ Step-by-Step Deployment Commands

### 1. Set Up the Virtual Switchboard
First, create an isolated virtual network so the containers can see and talk to each other safely:
```bash
docker network create mongo-network
```

### 2. Launch the MongoDB Database Server
Run the MongoDB engine inside the network. 
*(Note: We explicitly use version `7.0` to avoid a modern tcmalloc memory conflict present with v8.0+ on newer Linux kernels).*

```bash
docker run -d \
  -p 27017:27017 \
  --name mongo \
  --network mongo-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo:7.0
```

### 3. Launch the Mongo Express UI Dashboard
Deploy the optional, browser-based management console. Notice that it links to the database container using its plain text container name `mongo` over the shared network bridge:

```bash
docker run -d \
  -p 8081:8081 \
  --name mongo-express \
  --network mongo-network \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=qwerty \
  -e ME_CONFIG_MONGODB_URL="mongodb://admin:qwerty@mongo:27017" \
  mongo-express
```

---

## 🔍 Key Docker Concepts I Mastered

### 💡 Core Command Flags
* `docker run`: Creates a **brand-new** container from an image asset and shoots it online.
* `docker start`: Wakes up an **existing, stopped** container while maintaining its internal adjustments (cannot change port or environment configurations here).
* `-d` (Detached): Runs the container silently in the background, keeping my terminal console active.
* `-p <Host>:<Container>` (Port Mapping): Binds a physical port on my laptop to a private port inside the sandbox container.
* `-e` (Environment Variable): Injecting global settings into the container application context on initialization.
* `-t` (Tag): Assigning a readable string identifier (e.g., `myserver:1.0`) during an image build process.

### 🗂️ What are Layers?
Docker images are constructed like a layered cake or clear overhead sheets. Each instruction (`RUN`, `COPY`, `ENV`) adds a read-only filesystem modification layer.
* **Layer Caching:** If I edit code files, Docker reuses unchanged base operating system layers from memory, resulting in instant rebuild speeds.
* **Disk Optimization:** Multiple individual containers using the same base operating system share the identical underlying layers on my storage, saving gigabytes of memory space.

---

## 🚑 Troubleshooting & Disaster Recovery

### Clean Out Ghost Container States
If a setup crashes due to mismatched port assignments, Docker retains a "Created" ghost configuration. Clear out old definitions using:
```bash
docker rm -f <container_name>
```

### Debugging Application Logs
If a container disappears from `docker ps`, check its exit state inside the universal roster (`docker ps -a`) and extract its logs to diagnose application exceptions:
```bash
docker logs mongo
```

### Recovering a Broken Docker Daemon Socket
If manual execution leaves behind stale `.sock` system lock files that keep the Docker API offline, fix the service configuration entirely using:
```bash
sudo killall dockerd
sudo rm -f /var/run/docker.sock /var/run/docker.pid
sudo systemctl restart docker.socket
sudo systemctl restart docker
```

---

## 📈 Next Milestones
* [ ] Move the Node.js server itself inside a Dockerfile.
* [ ] Replace individual CLI launch setups with a single, structured `docker-compose.yml` file.
* [ ] Map a local disk directory to a Docker Volume (`-v`) so database tables persist forever even if a container is wiped away.
