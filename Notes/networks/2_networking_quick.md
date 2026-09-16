## Day 1 — Networking essentials for DevOps

Forget trying to memorize the entire OSI model. Our question is:

> **“What actually happens when I access my deployed FastAPI API?”**

Start with this:

```text
              YOUR COMPUTER
                   │
                   │
             ┌─────▼─────┐
             │  Browser  │
             └─────┬─────┘
                   │
                HTTPS
                   │
                   ▼
              INTERNET
                   │
                   ▼
          ┌────────────────┐
          │ Cloud Server   │
          │                │
          │ Public IP      │
          └───────┬────────┘
                  │
                :443
                  │
                  ▼
             ┌─────────┐
             │  Nginx  │
             └────┬────┘
                  │
                :8000
                  │
                  ▼
             ┌─────────┐
             │ FastAPI │
             └────┬────┘
                  │
                  ▼
             Database
```

We're going to understand **every single piece** of this.

---

# 1. IP Address — "Where?"

Suppose your server has:

```text
203.0.113.50
```

An **IP address** identifies a network endpoint so packets can be routed toward it.

Think:

```text
IP = destination address
```

For example:

```text
Browser
   |
   | "I need to reach 203.0.113.50"
   |
   ▼
Internet routers
   |
   ▼
203.0.113.50
```

This is Layer 3 territory.

But there's an immediate problem.

### What if that server is running 20 different services?

For example:

```text
203.0.113.50

SSH       → ?
HTTP      → ?
FastAPI   → ?
Database  → ?
```

The IP tells us **which machine/network endpoint**.

It doesn't tell us **which service/process**.

That's where the next concept comes in.

---

# 2. Port — "Which service?"

A **port** identifies a transport-layer endpoint associated with a service/process.

Think:

```text
IP   = which machine
Port = which service
```

For example:

```text
203.0.113.50:22
```

means:

```text
Server: 203.0.113.50
Port:   22
         ↓
        SSH
```

Common ports you'll encounter constantly in DevOps:

| Port | Typical use                     |
| ---: | ------------------------------- |
|   22 | SSH                             |
|   53 | DNS                             |
|   80 | HTTP                            |
|  443 | HTTPS                           |
| 5432 | PostgreSQL                      |
| 6379 | Redis                           |
| 8000 | FastAPI/Uvicorn commonly        |
| 3000 | Node/React development commonly |

Important:

**Ports aren't inherently tied to a specific application.**

For example, FastAPI can technically listen on `9000` instead of `8000`.

So:

```text
8000 ≠ FastAPI by definition
```

It's simply a port number that FastAPI/Uvicorn commonly uses.

---

# 3. Now combine IP + Port

Suppose your FastAPI server is:

```text
203.0.113.50
```

and FastAPI listens on:

```text
8000
```

You have:

```text
203.0.113.50:8000
```

Now networking knows:

> **Where?** → 203.0.113.50
> **Which endpoint?** → 8000

This combination is commonly called a **socket endpoint**.

---

# 4. But how does data actually travel?

This is where **TCP** enters.

Suppose your browser wants:

```text
GET /users
```

Your application generates HTTP data:

```text
GET /users
```

But that data needs reliable transport between the two endpoints.

TCP provides mechanisms such as:

* connection establishment
* sequencing
* acknowledgements
* retransmission
* flow control
* ordered delivery

So conceptually:

```text
HTTP
 │
 │ "GET /users"
 ▼
TCP
 │
 │ reliable transport
 ▼
IP
 │
 │ routing
 ▼
Network
```

This is the important relationship:

```text
HTTP
 ↓
TCP
 ↓
IP
```

---

# 5. TCP connection

Before TCP normally sends application data, the endpoints establish a connection.

The famous:

```text
Client                         Server

  SYN ───────────────────────►

      ◄──────────────── SYN-ACK

  ACK ───────────────────────►
```

This is the **TCP three-way handshake**.

You don't need to memorize packets yet.

Understand the purpose:

> **Both sides establish that they are ready to communicate using TCP.**

Then:

```text
HTTP request
     ↓
TCP
     ↓
IP
     ↓
network
```

---

# 6. TCP vs UDP

There are two transport protocols you'll constantly encounter:

```text
             Transport Layer
                    │
            ┌───────┴───────┐
            │               │
           TCP             UDP
```

### TCP

Designed for reliable, ordered communication.

Useful when correctness matters:

```text
HTTP
SSH
database connections
Git over HTTPS/SSH
```

### UDP

Connectionless and lightweight.

It doesn't provide TCP's built-in reliability/ordering mechanisms.

Useful for things such as:

```text
DNS queries
real-time media
gaming
QUIC/HTTP/3
```

The key idea:

```text
TCP → reliability/features
UDP → simplicity/low overhead
```

Don't reduce it to:

> TCP = fast, UDP = slow

That's incorrect.

---

# 7. Now DNS

Here's something you've used thousands of times without thinking about it.

You type:

```text
api.example.com
```

But networking ultimately needs an IP address.

DNS provides the name → address lookup mechanism.

Conceptually:

```text
api.example.com
        │
        │ DNS query
        ▼
    DNS server
        │
        │
        ▼
203.0.113.50
```

So:

```text
Domain name = human-friendly name
IP address  = network address
```

Then the client can communicate with:

```text
203.0.113.50
```

---

# 8. HTTP

Now we're back at the application layer.

Suppose you request:

```text
https://api.example.com/users
```

The application-level protocol is:

```text
HTTP
```

The request might conceptually contain:

```text
GET /users HTTP/1.1
Host: api.example.com
```

HTTP defines things such as:

* methods (`GET`, `POST`, `PUT`, `DELETE`)
* URLs/paths
* headers
* status codes
* request/response structure

So HTTP answers:

> **“What are the client and server saying to each other?”**

---

# 9. HTTPS

HTTPS is essentially:

```text
HTTP
 +
TLS security
```

TLS provides protections such as:

* encryption
* authentication of the server
* integrity protection

That's why you see:

```text
https://
```

instead of:

```text
http://
```

And HTTPS commonly uses:

```text
443
```

So:

```text
HTTPS
  ↓
TCP (traditionally for HTTP/1.1 and HTTP/2)
  ↓
IP
```

There's an important modern exception:

**HTTP/3 uses QUIC over UDP**, so don't build the mental model that HTTPS must always mean TCP.

For our initial DevOps deployment work, however, you'll very commonly encounter:

```text
HTTPS → TCP → IP
```

---

# 10. Now put EVERYTHING together

Suppose you deploy your FastAPI application.

Your domain:

```text
api.example.com
```

DNS:

```text
api.example.com
       ↓
203.0.113.50
```

User requests:

```text
https://api.example.com/users
```

Conceptually:

```text
        Browser
           │
           │ HTTPS
           ▼
        DNS lookup
           │
           ▼
    203.0.113.50
           │
           │ TCP :443
           ▼
       Cloud Server
           │
           ▼
         Nginx
           │
           │ HTTP :8000
           ▼
        FastAPI
           │
           ▼
       PostgreSQL
```

**THIS is the networking knowledge I want you to internalize.**

Not 500 networking definitions.

---

# 11. And now OSI finally becomes useful

Look at the same request:

```text
HTTPS / HTTP
     ↓
TCP + port
     ↓
IP
     ↓
Ethernet/Wi-Fi
     ↓
physical signals
```

Map it:

```text
L7 ─ HTTP/HTTPS
        ↓
L4 ─ TCP + ports
        ↓
L3 ─ IP + routing
        ↓
L2 ─ Ethernet/Wi-Fi + MAC
        ↓
L1 ─ electrical/optical/radio signals
```

Now OSI isn't something we're memorizing.

It's describing **different responsibilities involved in getting your request from A → B.**

---

# And this is where we're going next

We now have the minimum networking foundation.

Next I want to teach you:

## **"What happens when I type `ssh user@server-ip`?"**

Because SSH is one of the most important DevOps skills.

We'll use it to understand:

```text
IP
 ↓
port 22
 ↓
TCP
 ↓
SSH
 ↓
authentication
 ↓
remote shell
```

Then we'll move directly into:

```text
SSH
 ↓
Linux services
 ↓
Docker
 ↓
Docker networking
 ↓
Nginx
 ↓
AWS EC2
 ↓
Deployment
```


