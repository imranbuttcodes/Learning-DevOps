# Docker Volumes & Bind Mounts — Practical Guide

> A hands-on guide to the Docker storage concepts we learned, including **named volumes**, **bind mounts**, and a complete persistence experiment with MongoDB.

---

## Table of Contents

1. [Why Docker Storage Matters](#1-why-docker-storage-matters)
2. [The Container Filesystem Problem](#2-the-container-filesystem-problem)
3. [What Is a Docker Volume?](#3-what-is-a-docker-volume)
4. [Named Volumes](#4-named-volumes)
5. [Creating a Named Volume](#5-creating-a-named-volume)
6. [Where Does the Volume Live?](#6-where-does-the-volume-live)
7. [Practical Tutorial: MongoDB + Named Volume](#7-practical-tutorial-mongodb--named-volume)
8. [Proving Data Persistence](#8-proving-data-persistence)
9. [How the Persistence Experiment Works](#9-how-the-persistence-experiment-works)
10. [Bind Mounts](#10-bind-mounts)
11. [Practical Tutorial: Bind Mount](#11-practical-tutorial-bind-mount)
12. [Named Volume vs Bind Mount](#12-named-volume-vs-bind-mount)
13. [Important Docker Storage Concepts](#13-important-docker-storage-concepts)
14. [Useful Commands](#14-useful-commands)
15. [Common Mistakes](#15-common-mistakes)
16. [Production Mental Model](#16-production-mental-model)
17. [Final Cheat Sheet](#17-final-cheat-sheet)

---

# 1. Why Docker Storage Matters

Containers are designed to be **disposable**.

You can create a container:

```bash
docker run ...
```

stop it:

```bash
docker stop ...
```

and delete it:

```bash
docker rm ...
```

But applications often need data to survive the container.

For example:

- MongoDB needs to store database records.
- PostgreSQL needs to store database files.
- Redis may need persistent data depending on the architecture.
- An application may generate uploaded files.
- Logs or other application data may need persistence.

This creates an important question:

> **What happens to data when the container is deleted?**

Docker provides storage mechanisms such as **volumes** and **bind mounts** to solve this problem.

---

# 2. The Container Filesystem Problem

A container has its own filesystem.

For example:

```text
Container
│
├── /app
├── /etc
├── /usr
├── /var
└── /data
```

If an application writes data into the container's writable filesystem, that data belongs to that container.

Conceptually:

```text
Container
└── Writable layer
      └── Application data
```

If the container is deleted:

```text
Container
└── Writable layer
      └── Data
```

becomes:

```text
Container ❌
Data ❌
```

The Docker image itself still exists, but changes made only to the container's writable layer are not preserved by deleting that container.

Therefore, important application data should normally be stored outside the container's disposable writable layer.

---

# 3. What Is a Docker Volume?

A **Docker volume** is persistent storage managed by Docker.

The basic idea is:

```text
Container
   │
   │ mount
   ▼
Docker Volume
   │
   ▼
Physical storage
```

For example:

```text
MongoDB container
       │
       │ /data/db
       ▼
   mongo-data
       │
       ▼
   Local disk
```

The container can be deleted while the volume remains.

That means a new container can mount the same volume and access the existing data.

---

# 4. Named Volumes

A **named volume** is a Docker volume that has a name.

Example:

```bash
docker volume create mongo-data
```

This creates a volume called:

```text
mongo-data
```

It does **not**:

- create a MongoDB container
- install MongoDB
- reserve a fixed amount of disk space
- automatically store database data

It simply creates the persistent storage object.

Think:

```text
docker volume create mongo-data
            │
            ▼
Create storage object
            │
            ▼
Initially empty
```

Later, a container can mount it.

---

# 5. Creating a Named Volume

Run:

```bash
docker volume create mongo-data
```

Docker returns:

```text
mongo-data
```

Verify it:

```bash
docker volume ls
```

You should see something similar to:

```text
DRIVER    VOLUME NAME
local     mongo-data
```

## What does this actually mean?

Docker now knows about a volume named:

```text
mongo-data
```

But nothing is using it yet.

---

# 6. Where Does the Volume Live?

Inspect it:

```bash
docker volume inspect mongo-data
```

Example output:

```json
[
    {
        "CreatedAt": "2026-09-17T18:07:10+05:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mongo-data/_data",
        "Name": "mongo-data",
        "Options": null,
        "Scope": "local"
    }
]
```

The important field is:

```text
Mountpoint:
/var/lib/docker/volumes/mongo-data/_data
```

This tells us where the local Docker Engine stores the volume.

Conceptually:

```text
Your physical SSD
        │
        ▼
/var/lib/docker/volumes/
        │
        ▼
mongo-data/
        │
        ▼
_data/
```

## Does Docker reserve a fixed amount of storage?

**No.**

Creating a volume does not mean:

> "Reserve 10 GB for this volume."

Instead, the volume grows as data is written to it.

For example:

```text
Create volume
     ↓
Almost empty
     ↓
MongoDB writes 500 MB
     ↓
Volume uses roughly 500 MB of data
     ↓
MongoDB writes another 2 GB
     ↓
Volume grows accordingly
```

It is limited by the available storage and relevant filesystem/storage constraints.

> Avoid manually editing files inside Docker's managed volume directory. Use Docker or the application to manage the data.

---

# 7. Practical Tutorial: MongoDB + Named Volume

Now we will reproduce the complete experiment we performed.

Our goal:

> Create a MongoDB container → store real data → delete the container → create a new MongoDB container → attach the same volume → recover the data.

---

## Step 1 — Create the volume

```bash
docker volume create mongo-data
```

Verify:

```bash
docker volume ls
```

---

## Step 2 — Run MongoDB using the volume

```bash
docker run -d \
  --name mongo-volume-test \
  -v mongo-data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo:7.0
```

### The important part

```bash
-v mongo-data:/data/db
```

This follows:

```text
-v VOLUME_NAME:CONTAINER_PATH
```

Therefore:

```text
mongo-data:/data/db
```

means:

```text
Docker volume             Container
mongo-data      ────────► /data/db
```

MongoDB uses `/data/db` for its database files, so its database data is being stored in the volume.

---

## Step 3 — Check the container

```bash
docker ps
```

You should see:

```text
mongo-volume-test
```

You may see:

```text
27017/tcp
```

without a host mapping such as:

```text
0.0.0.0:27017->27017/tcp
```

That's okay for this experiment.

We don't need to connect to this MongoDB from the host. We can enter the container directly.

---

## Step 4 — Enter MongoDB

Run:

```bash
docker exec -it mongo-volume-test mongosh -u admin -p qwerty
```

You should enter the MongoDB shell:

```text
test>
```

Notice the distinction:

```text
Terminal
   │
   ▼
docker exec
   │
   ▼
MongoDB container
   │
   ▼
mongosh
   │
   ▼
test>
```

---

## Step 5 — Create a database

Inside MongoDB:

```javascript
use volume-demo
```

You should get:

```text
switched to db volume-demo
```

---

## Step 6 — Insert real data

Run:

```javascript
db.users.insertOne({
  name: "Imran",
  role: "AI Engineer"
})
```

You should get something similar to:

```text
{
  acknowledged: true,
  insertedId: ObjectId(...)
}
```

---

## Step 7 — Verify the data

Run:

```javascript
db.users.find()
```

You should see something like:

```text
[
  {
    _id: ObjectId('...'),
    name: 'Imran',
    role: 'AI Engineer'
  }
]
```

At this point, we have real MongoDB data.

The storage chain is:

```text
MongoDB
   │
   ▼
/data/db
   │
   │ mounted to
   ▼
mongo-data
   │
   ▼
Docker-managed storage
   │
   ▼
Local disk
```

---

## Step 8 — Exit MongoDB

Run:

```javascript
exit
```

You return to your normal Ubuntu terminal.

---

# 8. Proving Data Persistence

Now comes the most important part.

We are going to destroy the MongoDB container.

## Step 9 — Stop the container

```bash
docker stop mongo-volume-test
```

Then delete it:

```bash
docker rm mongo-volume-test
```

The container is now gone.

But what about the volume?

Check:

```bash
docker volume ls
```

You should still see:

```text
mongo-data
```

This proves:

```text
Container ❌ deleted

Volume ✅ still exists
```

---

# 9. How the Persistence Experiment Works

Now create a **completely new MongoDB container**.

## Step 10 — Create the new container

```bash
docker run -d \
  --name mongo-volume-test-2 \
  -v mongo-data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo:7.0
```

Notice:

```text
Old container:
mongo-volume-test
       │
       ▼
mongo-data
       │
       ✕
Container deleted


New container:
mongo-volume-test-2
       │
       ▼
mongo-data
```

We attached the **same volume** to a **new container**.

---

## Step 11 — Enter the new MongoDB

```bash
docker exec -it mongo-volume-test-2 mongosh -u admin -p qwerty
```

Then:

```javascript
use volume-demo
```

And:

```javascript
db.users.find()
```

You should see:

```text
[
  {
    _id: ObjectId('...'),
    name: 'Imran',
    role: 'AI Engineer'
  }
]
```

🔥 The data survived!

---

## What did we prove?

```text
OLD CONTAINER
mongo-volume-test
       │
       │ writes data
       ▼
mongo-data
       │
       │
       ▼
Database data


Delete old container
       ↓
Container ❌
Volume    ✅
Data      ✅


NEW CONTAINER
mongo-volume-test-2
       │
       │ mounts same volume
       ▼
mongo-data
       │
       ▼
Original database data
       │
       ▼
"Imran" still exists ✅
```

This is the fundamental purpose of a persistent Docker volume.

---

# 10. Bind Mounts

A **bind mount** is different from a named volume.

Instead of asking Docker to manage the storage location, you explicitly tell Docker:

> "Use this directory from my machine inside the container."

Example:

```bash
-v ~/bind-demo:/app
```

This means:

```text
Ubuntu host                     Container

~/bind-demo  ────────────────►  /app
```

The host directory and container directory refer to the same underlying files through the mount.

---

# 11. Practical Tutorial: Bind Mount

We performed this exact experiment.

## Step 1 — Create a directory

```bash
mkdir ~/bind-demo
```

Enter it:

```bash
cd ~/bind-demo
```

---

## Step 2 — Create a file on Ubuntu

```bash
echo "Hello from Ubuntu" > message.txt
```

Verify:

```bash
cat message.txt
```

Output:

```text
Hello from Ubuntu
```

---

## Step 3 — Start a container with a bind mount

We used Alpine because it is a small Linux image with a shell.

```bash
docker run --rm -it \
  -v ~/bind-demo:/app \
  alpine sh
```

You enter the container:

```text
/app #
```

Now run:

```bash
cat /app/message.txt
```

You see:

```text
Hello from Ubuntu
```

But we created that file on the host.

Why can the container see it?

Because:

```text
Ubuntu
~/bind-demo/
     │
     │ bind mount
     ▼
Container
/app/
```

---

## Step 4 — Test Host → Container

Open another Ubuntu terminal.

Change the file:

```bash
echo "Changed from Ubuntu" > ~/bind-demo/message.txt
```

Now go back to the container and run:

```bash
cat /app/message.txt
```

You should immediately see:

```text
Changed from Ubuntu
```

---

## Step 5 — Test Container → Host

Inside the container:

```bash
echo "Changed from container" > /app/message.txt
```

Now, in your normal Ubuntu terminal:

```bash
cat ~/bind-demo/message.txt
```

You should see:

```text
Changed from container
```

🔥 Both directions work.

That demonstrates that a bind mount is not simply copying a file into the container.

It is mounting the host directory into the container.

---

# 12. Named Volume vs Bind Mount

| Feature | Named Volume | Bind Mount |
|---|---|---|
| Example | `mongo-data:/data/db` | `~/bind-demo:/app` |
| Storage managed by | Docker | Host/user |
| Host path explicitly chosen | No | Yes |
| Good for persistent DB data | Yes | Possible, but not the typical Docker-managed-volume approach |
| Good for source-code development | Less convenient | **Yes** |
| Host changes immediately visible | Not the main purpose | **Yes** |
| Docker manages storage lifecycle | Yes | No |
| Docker creates storage object | Yes | No |

## Mental model

### Named volume

```text
Docker
  │
  ▼
mongo-data
  │
  ▼
Container /data/db
```

### Bind mount

```text
Your host directory
        │
        │ mount
        ▼
Container directory
```

---

# 13. Important Docker Storage Concepts

## Container writable layer

A running container has a writable layer on top of its image.

Conceptually:

```text
Image
├── Layer 1
├── Layer 2
└── Layer 3
       │
       ▼
Container writable layer
```

Changes made in the container's writable layer belong to that container.

Deleting the container removes that writable layer.

---

## Volume

A volume exists independently of the container:

```text
Container
    │
    ▼
Volume
```

Delete container:

```text
Container ❌
Volume    ✅
```

---

## Bind mount

A bind mount connects a host directory directly to a container directory:

```text
Host directory
      │
      ▼
Container directory
```

---

# 14. Useful Commands

## List volumes

```bash
docker volume ls
```

---

## Inspect a volume

```bash
docker volume inspect mongo-data
```

Useful information includes:

```text
Name
Driver
Mountpoint
Scope
```

---

## Create a volume

```bash
docker volume create mongo-data
```

---

## Remove a volume

```bash
docker volume rm mongo-data
```

⚠️ Be careful: removing a volume can permanently remove the persistent data stored in it.

Make sure no important data is stored there before removing it.

---

## Run a container with a named volume

```bash
docker run -v mongo-data:/data/db mongo:7.0
```

---

## Run a container with a bind mount

```bash
docker run -v ~/bind-demo:/app alpine sh
```

---

## See Docker storage usage

```bash
docker system df -v
```

This can help you understand how much space Docker resources are consuming.

---

# 15. Common Mistakes

## Mistake 1 — Thinking `docker volume create` reserves a fixed amount of disk

Wrong idea:

```text
docker volume create mongo-data
       ↓
Reserve 10 GB
```

Correct:

```text
docker volume create mongo-data
       ↓
Create Docker-managed storage object
       ↓
Data grows as needed
```

---

## Mistake 2 — Thinking deleting a container deletes its named volume

Normally:

```bash
docker rm mongo-volume-test
```

does **not** remove:

```text
mongo-data
```

That's why our MongoDB data survived.

---

## Mistake 3 — Confusing volume removal with container removal

These are different:

```bash
docker rm mongo-volume-test
```

removes a container.

Whereas:

```bash
docker volume rm mongo-data
```

removes the volume.

The second operation can destroy your persistent data.

---

## Mistake 4 — Using the wrong container path

For MongoDB, we used:

```bash
-v mongo-data:/data/db
```

The second part must correspond to the directory where MongoDB stores its database data.

---

## Mistake 5 — Thinking bind mounts copy files

A bind mount such as:

```bash
-v ~/bind-demo:/app
```

doesn't mean:

```text
copy ~/bind-demo → /app
```

Instead, it makes the host directory available at `/app` through a mount.

That's why changes are immediately visible in both places.

---

# 16. Production Mental Model

For an application such as an AI backend, you might eventually have:

```text
                    Production Server
                           │
             ┌─────────────┴─────────────┐
             │                           │
        FastAPI container           PostgreSQL container
             │                           │
             │                           ▼
             │                      DB volume
             │
             ▼
       Application code
```

The application container can be recreated:

```text
Old FastAPI container
        ↓
      deleted
        ↓
New FastAPI container
```

while persistent database storage remains:

```text
PostgreSQL
    │
    ▼
Persistent volume
    │
    ▼
Database data survives
```

This is one of the key ideas behind containerized deployments:

> **Containers can be disposable; important data should not be.**

---

# 17. Final Cheat Sheet

## Named Volume

```bash
docker volume create mongo-data
```

Mount it:

```bash
docker run -v mongo-data:/data/db mongo:7.0
```

Mental model:

```text
Docker-managed storage
        │
        ▼
     Volume
        │
        ▼
Container
```

Best mental association:

> **Named volume = persistent application data managed by Docker**

---

## Bind Mount

```bash
docker run -v ~/bind-demo:/app alpine sh
```

Mental model:

```text
Host directory
      │
      ▼
Container directory
```

Best mental association:

> **Bind mount = directly share a host directory with a container**

---

## The Persistence Experiment

```text
Create volume
     ↓
Run MongoDB
     ↓
Mount volume at /data/db
     ↓
Insert database data
     ↓
Delete container
     ↓
Volume remains
     ↓
Create new MongoDB container
     ↓
Mount same volume
     ↓
Original data appears
```

---

## The Most Important Distinction

```text
IMAGE
  │
  │ creates
  ▼
CONTAINER
  │
  │ disposable runtime
  │
  ├──────────────► NAMED VOLUME
  │                  │
  │                  └── persistent data
  │
  └──────────────► BIND MOUNT
                     │
                     └── host directory
```

### Remember:

**Image** → packaged application environment

**Container** → running instance

**Named volume** → Docker-managed persistent storage

**Bind mount** → host directory mounted into container

**Container can be deleted. Persistent storage can survive.**
