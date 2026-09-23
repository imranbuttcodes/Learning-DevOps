Yep bro. Now that we've solved it, **don't repeat the entire troubleshooting process every time**. 😭

## For every new EC2 instance, use this workflow

### 1. Launch the EC2 instance

For your normal DevOps practice:

* AMI: **Amazon Linux 2023**
* Instance type: `t3.micro` is fine for basic labs
* Create/select a key pair, e.g. `myserver-key`
* Enable **Auto-assign Public IP**
* Security Group:

  * **SSH TCP 22** → your IP, ideally
  * HTTP 80 → `0.0.0.0/0` only if you're actually hosting HTTP

You'll get something like:

```text
Public DNS:
ec2-3-93-194-213.compute-1.amazonaws.com
```

---

### 2. Put the `.pem` file somewhere safe

For example:

```bash
~/Downloads/Bro-AWS/myserver-key.pem
```

Then:

```bash
chmod 400 ~/Downloads/Bro-AWS/myserver-key.pem
```

---

### 3. First try the standard AWS SSH command

```bash
ssh -i ~/Downloads/Bro-AWS/myserver-key.pem \
ec2-user@YOUR_EC2_PUBLIC_DNS
```

For example:

```bash
ssh -i ~/Downloads/Bro-AWS/myserver-key.pem \
ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

### If it works

You're done. ✅

You'll see:

```text
[ec2-user@ip-172-31-xx-xx ~]$
```

**Use port 22 normally if it works.**

---

# What if you get our annoying error? 😂

If you get:

```text
kex_exchange_identification: read: Connection reset by peer
```

**Don't immediately recreate the EC2.**

Do this:

### Step A — Try port 443

But remember: a brand-new EC2 instance will **not automatically have SSH listening on 443**.

You need to configure it first.

From EC2 Instance Connect/browser terminal:

```bash
sudo sh -c 'echo "Port 443" >> /etc/ssh/sshd_config'
```

Validate:

```bash
sudo sshd -t
```

Restart:

```bash
sudo systemctl restart sshd
```

Verify:

```bash
sudo ss -tlnp | grep sshd
```

You want:

```text
0.0.0.0:22
0.0.0.0:443
```

---

### Step B — Allow TCP 443 in the Security Group

Add:

```text
Type: Custom TCP
Port: 443
Source: your IP
```

For a temporary troubleshooting test, you can use:

```text
0.0.0.0/0
```

but restricting it to your IP is preferable when practical.

---

### Step C — Connect from your laptop

```bash
ssh -p 443 \
-i ~/Downloads/Bro-AWS/myserver-key.pem \
ec2-user@YOUR_EC2_PUBLIC_DNS
```

That's it.

---

# So your future mental checklist is

```text
CREATE EC2
    ↓
Public IP/DNS?
    ↓
Key pair?
    ↓
Security Group allows SSH?
    ↓
chmod 400 key
    ↓
Try SSH :22
    ↓
     ┌───────────────┐
     │               │
   WORKS           RESET
     │               │
     ↓               ↓
   DONE         Configure :443
                     ↓
               SG allows :443
                     ↓
                 SSH :443
                     ↓
                   DONE
```

## Was the original issue specific to your laptop?

**We cannot conclusively say it was specific to your ThinkPad/laptop.**

What we established is more precise:

* The problem happened with your Ubuntu laptop.
* It happened against **multiple fresh EC2 instances**.
* It happened with **different EC2 key pairs**.
* It even happened with a **fresh EC2 instance before you configured anything**.
* It persisted when you switched to a **phone hotspot**.
* TCP/22 could connect, but the SSH exchange was reset.
* SSH over TCP/443 worked.

So the evidence points toward **something affecting SSH traffic on TCP/22 along your network path**, rather than an EC2 configuration problem.

But we **didn't prove exactly which component** caused it. It could potentially be something in the local/network path, ISP/network policy, or another intermediary.

Therefore, don't think:

> "Every laptop has this problem."

That's **not established**.

And don't think:

> "Every AWS EC2 needs SSH on 443."

Also **not true**.

The normal situation is:

```text
Most machines
     ↓
SSH :22
     ↓
EC2
```

Your situation is effectively:

```text
Your environment
     ↓
SSH :22  ❌ problematic
SSH :443 ✅ working
     ↓
EC2
```

### One important improvement for your future EC2 labs

If you're creating lots of temporary EC2 instances, I'd actually keep a small **EC2 SSH checklist** in your DevOps notes:

```text
[ ] Amazon Linux 2023
[ ] Public IPv4/DNS
[ ] Key pair downloaded
[ ] chmod 400 key
[ ] SG: TCP 22 allowed
[ ] Try SSH :22
[ ] If reset → configure SSH :443
[ ] SG: TCP 443 allowed
[ ] SSH using -p 443
```

That'll save you from the **5-hour SSH boss fight** we just went through. 💀
