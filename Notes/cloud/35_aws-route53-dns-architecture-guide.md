> **EC2 public IP → domain registration → Route 53 Hosted Zone → nameservers → DNS records → ALB → health checks → failover**

Once you understand this flow, Route 53 will stop feeling like random DNS terminology.

---

# 1. Where we start: your EC2 public IP

Suppose you have an EC2 instance:

```text
EC2
 │
 └── Public IPv4
       54.197.28.1
```

You installed Nginx and your website is running on port 80.

So currently:

```text
Browser
   │
   │ http://54.197.28.1
   ▼
 EC2
   │
   ▼
 Nginx
   │
   ▼
Website
```

This works.

But there is a problem.

### IP addresses aren't good human addresses.

You don't want to tell someone:

```text
"Visit 54.197.28.1"
```

You want:

```text
"Visit example.com"
```

That's where the domain + DNS come in.

---

# 2. Step 1 — Get a domain

Suppose you want:

```text
imrancloud.com
```

You need to **register** it.

You can register a domain through a registrar such as:

* Amazon Route 53
* Namecheap
* GoDaddy
* Cloudflare Registrar
* etc.

For our AWS example:

```text
You
 │
 ▼
Route 53 Registrar
 │
 ▼
Register imrancloud.com
```

You pay a registration/renewal fee because the domain is a globally unique name that is being registered to you for a period.

---

# 3. Step 2 — Create a Hosted Zone

Now we get into the actual **Route 53 DNS** part.

You create:

```text
Hosted Zone
     │
     └── imrancloud.com
```

Think of the Hosted Zone as:

> **The DNS database/configuration for your domain.**

Inside it you'll have DNS records.

Initially, Route 53 will typically create records such as:

```text
imrancloud.com
     │
     ├── NS
     └── SOA
```

We'll come back to those.

---

# 4. Step 3 — Route 53 gives you nameservers

Route 53 assigns nameservers to your hosted zone.

For example:

```text
ns-123.awsdns-45.com
ns-456.awsdns-78.net
ns-789.awsdns-12.org
ns-012.awsdns-34.co.uk
```

These are your **authoritative nameservers**.

They basically say:

> "If you want DNS information about `imrancloud.com`, ask us."

---

# 5. Step 4 — Connect the domain to Route 53

This is where people often get confused.

### If you registered the domain through Route 53

AWS can handle the registration and DNS setup together, so much of this is automatic.

### If you registered it somewhere else

For example:

```text
Namecheap
```

You go to Namecheap and change the domain's **nameservers** to the Route 53 nameservers:

```text
Namecheap
    │
    │ Nameservers
    ▼
ns-123.awsdns-45.com
ns-456.awsdns-78.net
ns-789.awsdns-12.org
ns-012.awsdns-34.co.uk
```

Now the global DNS system knows:

> "Route 53 is authoritative for `imrancloud.com`."

---

# 6. Now we create our first DNS record

Suppose your EC2 IP is:

```text
54.197.28.1
```

We create:

```text
A Record
```

with:

```text
Name:
imrancloud.com

Type:
A

Value:
54.197.28.1
```

Conceptually:

```text
imrancloud.com
       │
       │ A record
       ▼
54.197.28.1
```

Now the whole flow becomes:

```text
User
 │
 │ imrancloud.com
 ▼
DNS
 │
 ▼
Route 53
 │
 │ A record
 ▼
54.197.28.1
 │
 ▼
EC2
 │
 ▼
Nginx
 │
 ▼
Website
```

🎉 **You now have a domain pointing to your EC2 server.**

---

# 7. What happens when someone types your domain?

Let's trace it properly.

User enters:

```text
https://imrancloud.com
```

The browser doesn't inherently know what IP that means.

It asks a DNS resolver.

Simplified:

```text
Browser
   │
   │ "What's imrancloud.com?"
   ▼
DNS Resolver
   │
   ▼
DNS hierarchy
   │
   ▼
Route 53 Nameserver
   │
   │ "A record = 54.197.28.1"
   ▼
54.197.28.1
   │
   ▼
EC2
```

Then the browser connects to that IP.

---

# 8. Now the problem with using EC2 IP directly

Here's an important AWS problem.

You might have:

```text
EC2
54.197.28.1
```

But EC2 public IPs can change when an instance is stopped and started.

You don't want:

```text
imrancloud.com
       ↓
54.197.28.1
```

and then suddenly:

```text
EC2 restarted
       ↓
54.197.55.73
```

Now your DNS record is wrong.

---

# 9. Elastic IP

One solution is an **Elastic IP**.

You allocate an Elastic IP:

```text
44.207.14.79
```

Associate it with your EC2:

```text
Elastic IP
44.207.14.79
     │
     ▼
EC2
```

Then:

```text
imrancloud.com
       │
       ▼
A Record
       │
       ▼
44.207.14.79
```

Now the public IP remains associated with your AWS account until you release/disassociate it.

But there's an even better architecture for production web applications.

---

# 10. Instead of EC2 → use ALB

Remember the **Application Load Balancer** we learned earlier.

Instead of:

```text
Route 53
    │
    ▼
EC2
```

we can do:

```text
Route 53
    │
    ▼
ALB
    │
 ┌──┴──┐
 ▼     ▼
EC2   EC2
```

Now the user doesn't need to know any EC2 IP.

The ALB already has a stable DNS name such as:

```text
mywebserver-792292551.us-east-1.elb.amazonaws.com
```

---

# 11. How do we connect our domain to ALB?

This is where **Alias records** become important.

In Route 53:

```text
Name:
imrancloud.com

Type:
A

Alias:
Yes

Target:
mywebserver-792292551.us-east-1.elb.amazonaws.com
```

Conceptually:

```text
imrancloud.com
       │
       ▼
Route 53 A / Alias
       │
       ▼
ALB
       │
 ┌─────┴─────┐
 ▼           ▼
EC2          EC2
```

This is a much more realistic AWS architecture.

---

# 12. What about `www`?

You may want:

```text
imrancloud.com
www.imrancloud.com
api.imrancloud.com
```

You can create different records.

For example:

```text
imrancloud.com
       │
       ▼
      ALB


www.imrancloud.com
       │
       ▼
imrancloud.com
```

For the second one, you could use:

```text
CNAME
```

So:

```text
www.imrancloud.com
       │
       │ CNAME
       ▼
imrancloud.com
```

---

# 13. Now let's understand DNS record types

You don't need to memorize 50 record types.

These are the important ones.

---

## A Record

**A = IPv4 address**

```text
example.com
      ↓
54.197.28.1
```

Use when you want a hostname to resolve to an IPv4 address.

---

## AAAA Record

Same basic idea, but for **IPv6**.

```text
example.com
      ↓
IPv6 address
```

So:

```text
A     → IPv4
AAAA  → IPv6
```

---

# 14. CNAME

**CNAME = Canonical Name**

Points one hostname to another hostname.

```text
www.example.com
       │
       ▼
example.com
```

Notice:

```text
A:
example.com → IP address

CNAME:
www.example.com → another hostname
```

---

# 15. MX

**MX = Mail Exchange**

Used for email delivery.

For example:

```text
example.com
      │
      ▼
MX record
      │
      ▼
mail server
```

If you want:

```text
imran@example.com
```

DNS needs to know which mail servers handle mail for `example.com`.

---

# 16. TXT

TXT records contain text information associated with a domain.

They're commonly used for:

### Domain verification

```text
TXT
example.com
"verification=abc123"
```

### Email security

For example:

```text
SPF
DKIM
DMARC
```

TXT records are extremely common in real deployments.

---

# 17. NS

**NS = Name Server**

These records identify the authoritative nameservers for a domain/zone.

For example:

```text
example.com
    │
    ▼
NS
    │
    ├── ns-123.awsdns...
    ├── ns-456.awsdns...
    ├── ns-789.awsdns...
    └── ns-012.awsdns...
```

This is how the DNS hierarchy knows which nameservers are authoritative for the domain.

---

# 18. SOA

**SOA = Start of Authority**

It contains authoritative information about a DNS zone.

You normally don't need to manually touch it when you're starting with Route 53.

Just know:

```text
NS → which nameservers are authoritative
SOA → authority/zone metadata
```

---

# 19. So your basic Route 53 Hosted Zone might look like

```text
Hosted Zone: imrancloud.com

┌──────────────────────────────────────────────┐
│ Name                  Type       Value       │
├──────────────────────────────────────────────┤
│ imrancloud.com        A          ALB         │
│ www.imrancloud.com    CNAME      imrancloud.com
│ imrancloud.com        NS         ns-...      │
│ imrancloud.com        SOA        ...         │
│ imrancloud.com        MX         mail server │
│ imrancloud.com        TXT        verification│
└──────────────────────────────────────────────┘
```

---

# 20. Now the interesting part: Health Checks

Suppose you have:

```text
EC2 #1
54.197.28.1
```

and your website suddenly crashes.

DNS doesn't automatically know that your application is broken just because an EC2 instance exists.

This is where **Route 53 Health Checks** come in.

---

# 21. What is a Route 53 health check?

A Route 53 health check periodically checks whether an endpoint is healthy.

For example:

```text
Route 53 Health Check
        │
        │ HTTP
        ▼
https://example.com/health
        │
        ▼
     Server
```

It can check things such as:

```text
IP address
domain name
port
protocol
path
```

For example:

```text
Protocol: HTTP
Port: 80
Path: /health
```

---

# 22. Your application can have a health endpoint

For FastAPI:

```text
GET /health
```

might return:

```json
{
  "status": "healthy"
}
```

Then Route 53 checks:

```text
http://your-server/health
```

If the server responds correctly:

```text
HEALTHY ✅
```

If it repeatedly fails:

```text
UNHEALTHY ❌
```

---

# 23. But here's an important distinction

**Route 53 health checks don't replace ALB health checks.**

You've already learned ALB Target Group health checks.

These are different.

### ALB health check

Checks individual backend targets:

```text
ALB
 │
 ├── EC2 #1 → /health ✅
 ├── EC2 #2 → /health ❌
 └── EC2 #3 → /health ✅
```

ALB stops sending traffic to unhealthy targets.

### Route 53 health check

Can determine whether a DNS endpoint is healthy:

```text
Route 53
      │
      ▼
Endpoint
      │
      ▼
Healthy / Unhealthy
```

They operate at different layers.

---

# 24. Why would Route 53 need health checks?

Consider two regions:

```text
                 Route 53
                    │
             Health-based routing
               /             \
              /               \
             ▼                 ▼
        US Region          Europe Region
             │                 │
            ALB               ALB
             │                 │
            EC2               EC2
```

Suppose the US endpoint fails:

```text
US → ❌
Europe → ✅
```

Route 53 can use health information in a **failover routing configuration**.

Conceptually:

```text
example.com
      │
      ▼
Route 53
      │
      ├── Primary → US ❌
      │
      └── Secondary → Europe ✅
                              │
                              ▼
                         Send traffic
```

---

# 25. Route 53 routing policies

This is another major Route 53 concept.

Route 53 doesn't just answer:

> "What IP?"

It can also answer:

> "Which destination should receive this request?"

Important routing policies include:

### Simple

```text
example.com
     ↓
one destination
```

Basic DNS routing.

---

### Weighted

You can distribute traffic according to weights.

For example:

```text
Version A → 90%
Version B → 10%
```

Useful for things like controlled releases/testing.

---

### Latency-based

Route 53 chooses an endpoint based on network latency.

Conceptually:

```text
User in Asia
     ↓
Asia endpoint

User in Europe
     ↓
Europe endpoint
```

---

### Failover

Primary/secondary.

```text
Primary
   │
   ├── Healthy → use primary
   │
   └── Unhealthy
          ↓
       Secondary
```

---

### Geolocation

Routing based on where the user is geographically located.

For example:

```text
Pakistan → Pakistan/Asia endpoint
US       → US endpoint
Europe   → Europe endpoint
```

---

# 26. The complete architecture

Now let's put **everything together**.

## Simple version

```text
                 User
                  │
                  │ https://imrancloud.com
                  ▼
             DNS Resolver
                  │
                  ▼
          Route 53 Nameserver
                  │
                  ▼
             A Record
                  │
                  ▼
            Elastic IP
                  │
                  ▼
                EC2
                  │
                Nginx
                  │
                  ▼
              Website
```

---

# 27. Production-style AWS version

This is the architecture I want you to remember:

```text
                           USER
                             │
                             │ https://example.com
                             ▼
                        ┌──────────┐
                        │ Route 53 │
                        └────┬─────┘
                             │
                       Alias Record
                             │
                             ▼
                        ┌──────────┐
                        │CloudFront│
                        └────┬─────┘
                             │
                             ▼
                           ALB
                             │
                   ┌─────────┴─────────┐
                   │                   │
                   ▼                   ▼
                EC2 #1              EC2 #2
                   │                   │
                   └─────────┬─────────┘
                             ▼
                         FastAPI
                             │
                             ▼
                            RDS
```

And Route 53 can additionally use:

```text
Health Checks
     │
     ▼
Routing Decisions
     │
     ├── Primary
     ├── Secondary
     ├── Weighted
     ├── Latency
     └── Geolocation
```

---

# 28. The entire setup in chronological order

If we were actually deploying your application:

### Step 1

Create your application.

```text
FastAPI
```

### Step 2

Deploy it to EC2.

```text
EC2
```

### Step 3

Put multiple EC2 instances behind an ALB.

```text
ALB
 │
 ├── EC2
 ├── EC2
 └── EC2
```

### Step 4

Register:

```text
example.com
```

### Step 5

Create Route 53 Hosted Zone:

```text
example.com
```

### Step 6

Configure nameservers if the domain was registered elsewhere.

```text
Registrar
    ↓
Route 53 nameservers
```

### Step 7

Create DNS record:

```text
example.com
   │
   ▼
A / Alias
   │
   ▼
ALB
```

### Step 8

Create:

```text
www.example.com
```

if desired.

### Step 9

Configure health checks if your routing architecture needs Route 53 health-based decisions.

### Step 10

Add an SSL/TLS certificate through **AWS Certificate Manager (ACM)** and use HTTPS.

Then your final user experience becomes:

```text
                    https://example.com
                              │
                              ▼
                         Route 53
                         DNS lookup
                              │
                              ▼
                         CloudFront
                              │
                              ▼
                            ALB
                              │
                  ┌───────────┴───────────┐
                  ▼                       ▼
                EC2 #1                  EC2 #2
                  │                       │
                  └───────────┬───────────┘
                              ▼
                           FastAPI
                              │
                              ▼
                             RDS
```

### The 5 things I want you to remember right now

```text
DOMAIN
  ↓
The human-friendly name
example.com


HOSTED ZONE
  ↓
DNS configuration for the domain


NAMESERVER
  ↓
DNS server that is authoritative for the domain


DNS RECORD
  ↓
Instruction such as:
example.com → ALB


HEALTH CHECK
  ↓
Checks whether an endpoint is healthy
```

And **Route 53 is the AWS service tying these DNS pieces together**.