### 1. First: what is a Docker network?

A Docker network allows containers to communicate.

For example, your setup is roughly:

```text
                 Docker network
              "docker-testapp-main"
                       │
          ┌────────────┴────────────┐
          │                         │
     MongoDB                    Mongo Express
       mongo                  mongo-express
          │                         │
          └───────────┬─────────────┘
                      │
                 Node.js app
```

The network determines **how these containers connect to each other and to the outside world**.

---

# 2. So what's a "driver"?

A **network driver tells Docker how that network should work internally.**

You can see available drivers with:

```bash
docker network ls
```

You'll typically see:

```text
NETWORK ID     NAME       DRIVER    SCOPE
xxxx           bridge     bridge    local
xxxx           host       host      local
xxxx           none       null      local
```

Here:

```text
DRIVER
  ↓
How Docker implements the network
```

---

# 3. The important drivers

For your DevOps learning, focus primarily on these:

### `bridge`

The most important one for normal Docker applications.

```bash
docker network create --driver bridge my-network
```

Containers connected to this network can communicate with each other.

For example:

```text
Node container
      │
      │ mongodb://mongo:27017
      ↓
Mongo container
```

Docker provides internal networking and DNS, so:

```text
mongo
```

can resolve to the Mongo container's IP.

**This is what you've already been using.**

Your earlier:

```bash
--network mongo-network
```

was connecting containers to a bridge network.

---

# 4. `host`

With the host driver, the container shares the host's network namespace rather than getting a normal isolated Docker network interface.

Conceptually:

```text
Normal bridge:

Host
 │
 ├── Docker network
 │      ├── Container A
 │      └── Container B
 │
 └── Host network


Host driver:

Host network
      │
      └── Container
          (uses host networking)
```

Example:

```bash
docker run --network host nginx
```

The container doesn't get the same kind of separate network isolation as a bridge-networked container.

This can be useful for specific performance/networking cases, but **don't use it as your default application networking model**.

---

# 5. `none`

This essentially gives the container **no normal network connectivity**.

```bash
docker run --network none nginx
```

Conceptually:

```text
Container
   │
   ✕
No network
```

Useful when a container genuinely shouldn't communicate over a network.

---

# 6. The one you'll encounter later: `overlay`

`overlay` is important when you're dealing with **multiple Docker hosts**, particularly Docker Swarm.

Imagine:

```text
Server A                         Server B

Node container                  Node container
     │                                │
     └──────── Overlay network ───────┘
```

The network can span multiple Docker hosts.

For your current learning, you don't need to go deep into it yet.

---

# 7. Your Mongo example

You previously had:

```bash
docker network create mongo-network
```

Docker creates a network.

You can inspect it:

```bash
docker network inspect mongo-network
```

You'll see something indicating its driver:

```json
"Driver": "bridge"
```

So:

```text
mongo-network
      │
      └── Driver = bridge
```

That means Docker is implementing this as a **bridge network**.

Then:

```bash
docker run --network mongo-network mongo
```

connects Mongo to that network.

And:

```bash
docker run --network mongo-network mongo-express
```

connects Mongo Express to the same network.

Therefore Mongo Express can communicate with Mongo using:

```text
mongo:27017
```

rather than:

```text
localhost:27017
```

That's because **`mongo` is the Docker DNS/service/container name on that network**.

---

# The mental model I want you to remember

Don't think:

> "Driver = a network."

Think:

> **Network = the actual Docker network.**
> **Driver = the technology/mechanism Docker uses to implement that network.**

Like:

```text
Docker Network
      │
      ├── Name: mongo-network
      │
      ├── Driver: bridge
      │
      └── Containers:
            ├── mongo
            └── mongo-express
```

And the main drivers you'll encounter are:

```text
bridge   → normal container networking ⭐
host     → use host networking
none     → no normal networking
overlay  → networking across Docker hosts
```

