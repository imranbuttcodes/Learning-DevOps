## 1. The problem

Suppose your FastAPI application runs inside a container:

```text
┌──────────────────────────────┐
│       Docker Container       │
│                              │
│       FastAPI                │
│          │                   │
│       listens on             │
│          ↓                   │
│         :8000                │
└──────────────────────────────┘
```

FastAPI is listening on port `8000` **inside the container**.

But your laptop/browser is outside:

```text
Your laptop
     │
     │ ??? 
     ↓
Docker container
     │
   :8000
```

How does traffic get from your laptop to the container?

That's where **port binding** comes in.

---

# 2. Port binding

You can bind a host port to a container port:

```bash
docker run -p 8000:8000 my-api
```

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

So:

```text
-p 8000:8000
   │     │
   │     └── container port
   └──────── host port
```

Meaning:

> "Docker, forward traffic arriving at port 8000 on my machine to port 8000 inside this container."

---

# 3. Visualize it

```text
        YOUR LAPTOP
        localhost:8000
              │
              │
              ↓
      ┌─────────────────┐
      │      Docker     │
      │      Engine     │
      └────────┬────────┘
               │
               │ port mapping
               ↓
      ┌─────────────────┐
      │    Container    │
      │                 │
      │    FastAPI      │
      │      :8000      │
      └─────────────────┘
```

So when you open:

```text
http://localhost:8000
```

traffic can reach FastAPI inside the container.

---

# 4. Host port and container port don't have to be the same

For example:

```bash
docker run -p 9000:8000 my-api
```

means:

```text
Host                         Container

localhost:9000  ───────────→  :8000
```

FastAPI still listens on:

```text
8000
```

inside the container.

But users access it through:

```text
localhost:9000
```

from the host.

---

# 5. Why would we do this?

Imagine you have:

```text
Container 1 → FastAPI → 8000
Container 2 → React   → 3000
Container 3 → PostgreSQL → 5432
```

You could expose:

```bash
-p 8000:8000
-p 3000:3000
```

But PostgreSQL doesn't necessarily need to be exposed publicly.

This gives you an important distinction:

```text
Container port
      ↓
Where the application listens

Host port
      ↓
Where traffic enters from outside
```

---

# 6. Port binding vs exposing a port

This distinction is **very important**.

In a Dockerfile you might see:

```dockerfile
EXPOSE 8000
```

`EXPOSE` does **not** publish the port to your host.

It mainly documents:

> "This application expects to listen on port 8000."

You still need:

```bash
docker run -p 8000:8000 my-api
```

to publish/bind it to the host.

Think:

```text
EXPOSE 8000
      ↓
Documentation / image metadata

-p 8000:8000
      ↓
Actually publish host port → container port
```

---

# 7. Real FastAPI example

Your FastAPI application might run:

```text
Container
FastAPI
  ↓
0.0.0.0:8000
```

Then:

```bash
docker run -d -p 8000:8000 my-api
```

gives:

```text
Browser
   │
   │ http://localhost:8000
   ↓
Host :8000
   │
   │ Docker port binding
   ↓
Container :8000
   │
   ↓
FastAPI
```

---

# 🧠 The one sentence to remember

> **Port binding maps a port on the host machine to a port inside a Docker container so external traffic can reach the service running in the container.**

And remember the syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

