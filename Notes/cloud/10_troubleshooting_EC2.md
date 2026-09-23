# AWS EC2 SSH Connection Troubleshooting

## Overview

This document records the complete troubleshooting process for connecting from a local **Ubuntu 24.04** machine to an **Amazon Linux 2023 EC2 instance** using SSH.

The original SSH connection on the standard SSH port **22** repeatedly failed with:

```text
kex_exchange_identification: read: Connection reset by peer
Connection reset by <EC2_PUBLIC_IP> port 22
```

After testing multiple EC2 instances and verifying the server configuration, we configured `sshd` to also listen on **TCP port 443**.

SSH over port 443 successfully connected:

```text
[ec2-user@ip-172-31-20-22 ~]$
```

The final working architecture was:

```text
Ubuntu Laptop
     |
     | SSH
     | TCP 443
     v
Internet
     |
     v
AWS EC2 Public IP / DNS
     |
     | TCP 443
     v
Security Group
     |
     v
Amazon Linux 2023
     |
     | sshd
     v
ec2-user shell
```

---

# 1. Environment

## Local Machine

* OS: Ubuntu 24.04
* Architecture: x86_64
* SSH client: OpenSSH 9.6p1
* Local machine: Lenovo ThinkPad T480s

Example SSH client version:

```bash
ssh -V
```

Output:

```text
OpenSSH_9.6p1 Ubuntu-3ubuntu13.19
```

## AWS EC2

* OS: Amazon Linux 2023
* Instance type: t3.micro
* Architecture: x86_64
* SSH username: `ec2-user`
* SSH server: `sshd`
* Default SSH port: `22`

The EC2 instances used during troubleshooting included:

* `mywebserver`
* `broserver`
* `ssh-test`
* `imranserver`

The important point is that the problem reproduced across multiple instances.

---

# 2. Normal EC2 SSH Connection

The normal SSH command supplied by AWS was:

```bash
ssh -i "imranserver-key.pem" ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

The private key permissions were set correctly:

```bash
chmod 400 "imranserver-key.pem"
```

However, the connection failed with:

```text
kex_exchange_identification: read: Connection reset by peer
Connection reset by 3.93.194.213 port 22
```

---

# 3. What Does This Error Mean?

SSH has several stages.

A simplified connection looks like this:

```text
1. TCP connection
       |
       v
2. SSH identification exchange
       |
       v
3. Key exchange
       |
       v
4. Authentication
       |
       v
5. Interactive shell
```

Our connection was failing very early.

The verbose SSH output showed:

```text
debug1: Connecting to 3.93.194.213 port 22.
debug1: Connection established.
debug1: Local version string SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.19
kex_exchange_identification: read: Connection reset by peer
```

This was important.

The TCP connection itself was succeeding.

The failure happened after the connection was established, during the early SSH protocol exchange.

Therefore, the problem was **not simply "the server is unreachable."**

---

# 4. First Server-Side Checks

We used the EC2 browser-based terminal to inspect the server.

## Check whether SSH is running

```bash
sudo systemctl status sshd
```

The result showed that `sshd` was active and running.

Therefore:

```text
EC2
 |
 +-- sshd running
```

---

# 5. Check Whether Port 22 Was Listening

We ran:

```bash
sudo ss -tlnp | grep :22
```

The server showed:

```text
LISTEN 0 128 0.0.0.0:22 0.0.0.0:* users:(("sshd",...))
LISTEN 0 128 [::]:22 [::]:* users:(("sshd",...))
```

This confirmed:

```text
sshd
  |
  +-- IPv4 → 0.0.0.0:22
  |
  +-- IPv6 → [::]:22
```

So `sshd` was listening correctly on port 22.

---

# 6. Check the Server Firewall

We checked for `firewalld`:

```bash
sudo firewall-cmd --state
```

The system returned:

```text
command not found
```

So `firewalld` was not the source of the problem.

---

# 7. Check SSH Logs

We inspected the SSH server logs:

```bash
sudo journalctl -u sshd --since "10 minutes ago" --no-pager
```

The logs showed normal SSH activity, including successful EC2 Instance Connect sessions.

There were also connections such as:

```text
banner exchange: Connection from ... invalid format
```

These were unrelated Internet scanning attempts.

The important observation was that the EC2 SSH server itself was functioning.

---

# 8. Security Group Investigation

The EC2 Security Group controlled inbound network traffic.

The relevant rule was:

```text
SSH
TCP
22
Source: 0.0.0.0/0
```

At some points we also tested restricting SSH to the current public IP.

For example:

```text
110.38.160.4/32
```

However, the problem persisted.

Eventually, for the cleanest troubleshooting test, SSH was allowed from:

```text
0.0.0.0/0
```

This removed the local-IP restriction as a variable.

---

# 9. Testing Multiple EC2 Instances

One important part of the investigation was that the problem was not limited to one particular EC2 instance.

We created a completely fresh instance:

```text
ssh-test
```

It used:

* Amazon Linux 2023
* New SSH key
* New public IP
* No custom user data
* Fresh configuration

SSH was tested immediately.

Command:

```bash
ssh -i ~/Downloads/Bro-AWS/ssh-test-key.pem ec2-user@34.224.82.141
```

It still failed:

```text
kex_exchange_identification: read: Connection reset by peer
Connection reset by 34.224.82.141 port 22
```

This was an important experiment.

It made an instance-specific configuration mistake much less likely.

---

# 10. Testing From a Different Network

The SSH connection was also tested using a phone hotspot instead of the normal network.

The same reset occurred.

Therefore, simply changing the Wi-Fi/network connection did not solve the problem.

---

# 11. TCP-Level Investigation

We used `nc` from the Ubuntu laptop:

```bash
nc -vz -w 5 100.55.56.93 22
```

The result indicated that the TCP connection succeeded.

We also used:

```bash
timeout 10 nc 100.55.56.93 22
```

The connection did not produce the expected SSH banner.

This was consistent with the SSH connection being terminated during the early protocol exchange.

---

# 12. Packet Capture

We also inspected the traffic using `tcpdump` on the local Ubuntu machine.

A simplified version of what we observed was:

```text
Laptop                         EC2
  |                             |
  |-------- TCP SYN ----------->|
  |<------- TCP SYN/ACK --------|
  |-------- TCP ACK ----------->|
  |                             |
  |---- SSH identification ---->|
  |                             |
  |<----------- RST ------------|
  |                             |
```

The laptop successfully established the TCP connection.

Then it sent:

```text
SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.19
```

The public endpoint responded with a TCP reset.

This confirmed that the failure was happening **after TCP establishment but before normal SSH authentication**.

---

# 13. SSH Verbose Mode

We used:

```bash
ssh -vvv -i ~/Downloads/Bro-AWS/ssh-test-key.pem ec2-user@34.224.82.141
```

The important output was:

```text
debug1: Connecting to 34.224.82.141 [34.224.82.141] port 22.
debug1: Connection established.
debug1: identity file ... type -1
debug1: Local version string SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.19
kex_exchange_identification: read: Connection reset by peer
```

### Important clarification about `type -1`

The line:

```text
identity file ... type -1
```

looked suspicious at first, but it was not the root cause.

SSH had not reached the authentication stage yet.

The server reset the connection during the earlier SSH negotiation.

Therefore, the private key was not responsible for this particular failure.

---

# 14. AWS EC2 Instance Connect Worked

The browser-based EC2 Instance Connect terminal worked during the investigation.

The server prompt was:

```text
[ec2-user@ip-172-31-27-240 ~]$
```

This demonstrated that:

* The instance was running.
* Amazon Linux was booting correctly.
* `ec2-user` existed.
* SSH functionality itself was operational.
* AWS could reach the instance through its own connection mechanism.

The EC2 Instance Connect logs also showed successful public-key authentication.

---

# 15. Another Fresh Instance

We later tested another instance:

```text
imranserver
```

AWS itself provided the command:

```bash
chmod 400 "imranserver-key.pem"

ssh -i "imranserver-key.pem" ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

The exact AWS-provided command still produced:

```text
kex_exchange_identification: read: Connection reset by peer
Connection reset by 3.93.194.213 port 22
```

At this point, the evidence strongly indicated that repeatedly recreating EC2 instances or changing SSH keys was not addressing the actual problem.

---

# 16. The Key Insight: SSH Does Not Have to Use Port 22

A common misconception is:

```text
SSH = Port 22
```

More accurately:

```text
SSH = Protocol
22  = Conventional/default TCP port
```

The SSH daemon (`sshd`) can listen on another TCP port.

For example:

```text
Port 22
Port 443
Port 2222
```

can all be used for SSH if the server and network firewall are configured accordingly.

We decided to test SSH over TCP port **443**.

---

# 17. Configure sshd to Listen on Port 443

This command was executed **inside the Amazon Linux EC2 instance**, using the EC2 browser terminal:

```bash
sudo sh -c 'echo "Port 443" >> /etc/ssh/sshd_config'
```

This added:

```text
Port 443
```

to:

```text
/etc/ssh/sshd_config
```

---

# 18. Validate the SSH Configuration

Before restarting SSH, we checked the configuration:

```bash
sudo sshd -t
```

There was no output, which indicates the configuration syntax was valid.

This is an important DevOps practice:

```text
Edit configuration
       |
       v
Validate configuration
       |
       v
Restart service
```

Rather than blindly restarting a potentially broken configuration.

---

# 19. Restart sshd

We restarted the SSH service:

```bash
sudo systemctl restart sshd
```

Then checked the listening ports:

```bash
sudo ss -tlnp | grep sshd
```

The server showed:

```text
LISTEN 0 128 0.0.0.0:443 0.0.0.0:* users:(("sshd",...))
LISTEN 0 128 [::]:443 [::]:* users:(("sshd",...))
```

This confirmed that `sshd` was now listening on port 443.

The intended configuration became:

```text
sshd
 |
 +---- TCP 22
 |
 +---- TCP 443
```

---

# 20. Allow Port 443 Through the AWS Security Group

The AWS Security Group also needs an inbound rule for TCP 443.

Conceptually:

```text
Inbound Rules

TCP 22  → SSH
TCP 443 → SSH
```

For the troubleshooting test, port 443 can be temporarily allowed from:

```text
0.0.0.0/0
```

A tighter source restriction can be used later if desired.

---

# 21. Connect Using SSH Over Port 443

From the Ubuntu laptop:

```bash
ssh -p 443 -i ~/Downloads/Bro-AWS/imranserver-key.pem ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

The first successful connection displayed:

```text
The authenticity of host '[ec2-3-93-194-213.compute-1.amazonaws.com]:443
([3.93.194.213]:443)' can't be established.
```

This was expected.

Because SSH was now being accessed using a different host/port combination, OpenSSH treated:

```text
hostname:443
```

as a separate known-host entry.

We answered:

```text
yes
```

OpenSSH then permanently stored the host key:

```text
Warning: Permanently added
'[ec2-3-93-194-213.compute-1.amazonaws.com]:443'
```

---

# 22. Successful Connection

The server displayed the Amazon Linux login banner:

```text
Amazon Linux 2023
```

and then:

```text
Last login: Tue Sep 22 15:53:28 2026 from ...
[ec2-user@ip-172-31-20-22 ~]$
```

This was the definitive proof that SSH authentication and interactive access were working.

The final connection path was:

```text
Ubuntu Laptop
     |
     | SSH
     | TCP 443
     v
EC2 Public DNS
     |
     v
AWS Security Group
     |
     v
EC2 Network Interface
     |
     v
sshd :443
     |
     v
SSH Authentication
     |
     v
ec2-user
```

---

# 23. Why Port 443 Worked

The exact underlying cause of the original port-22 reset was **not conclusively identified**.

However, the experiments established several facts:

### We know:

* The EC2 instance was healthy.
* `sshd` was running.
* `sshd` was listening on port 22.
* The Security Group allowed port 22.
* The TCP connection to port 22 could be established.
* The failure happened during early SSH negotiation.
* The same behavior appeared on multiple EC2 instances.
* A fresh EC2 instance showed the same behavior.
* Changing networks did not resolve it.
* SSH over port 443 worked successfully.

### Therefore:

The practical conclusion is that something in the **network path affecting TCP/22** was interfering with the SSH protocol exchange, while TCP/443 was able to pass through successfully.

We should **not claim with certainty** that the ISP, router, firewall, NAT, or another specific component was responsible without additional evidence.

The important operational discovery was:

```text
TCP 22 → connection reset during SSH negotiation
TCP 443 → SSH works
```

---

# 24. Why Port 443 Is Useful

Port 443 is conventionally used for HTTPS:

```text
HTTP  → 80
HTTPS → 443
SSH   → 22
```

But port numbers do not fundamentally define the application protocol.

A server can run SSH on 443:

```text
TCP 443
   |
   v
sshd
   |
   v
SSH session
```

This can be useful when normal SSH traffic on TCP 22 is blocked or interfered with somewhere along the network path.

### Important

Running SSH on port 443 does **not** turn SSH into HTTPS.

It is still SSH.

The only change is the TCP destination port.

---

# 25. Current Configuration

The EC2 instance currently has SSH listening on both ports:

```text
TCP 22  → sshd
TCP 443 → sshd
```

Verified with:

```bash
sudo ss -tlnp | grep sshd
```

Expected output:

```text
LISTEN ... 0.0.0.0:22 ...
LISTEN ... [::]:22 ...
LISTEN ... 0.0.0.0:443 ...
LISTEN ... [::]:443 ...
```

---

# 26. Normal SSH Command

The original/default method remains:

```bash
ssh -i ~/Downloads/Bro-AWS/imranserver-key.pem ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

This uses:

```text
TCP 22
```

---

# 27. Working SSH Command

The currently working method is:

```bash
ssh -p 443 -i ~/Downloads/Bro-AWS/imranserver-key.pem ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

The `-p 443` option tells the SSH client:

```text
Connect to TCP port 443 instead of the default port 22.
```

---

# 28. Important Distinction: Local Machine vs EC2

During troubleshooting, one command was accidentally executed on the Ubuntu laptop:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

This failed because:

```text
/etc/ssh/sshd_config
```

is the SSH server configuration on the **EC2 Amazon Linux machine**.

The local Ubuntu machine is primarily running the SSH **client**.

Conceptually:

```text
Ubuntu Laptop
    |
    | ssh client
    |
    +------------------> EC2
                           |
                           | sshd server
                           |
                           +-- /etc/ssh/sshd_config
```

Therefore, commands modifying `sshd_config` must be executed on the EC2 server.

---

# 29. How the Whole System Works

## Step 1 — SSH Client

On Ubuntu:

```bash
ssh -p 443 -i imranserver-key.pem ec2-user@EC2_DNS
```

The SSH client reads the private key and starts a network connection.

## Step 2 — DNS

The EC2 public DNS name:

```text
ec2-3-93-194-213.compute-1.amazonaws.com
```

resolves to the EC2 public IP:

```text
3.93.194.213
```

## Step 3 — TCP

The laptop creates a TCP connection:

```text
Laptop → 3.93.194.213:443
```

## Step 4 — AWS Security Group

The Security Group checks the inbound connection.

Conceptually:

```text
Internet
   |
   v
TCP 443
   |
   v
Security Group
   |
   +-- Allowed → continue
```

## Step 5 — EC2

The packet reaches the EC2 instance.

## Step 6 — sshd

`sshd` is listening on port 443:

```text
0.0.0.0:443
```

Therefore it accepts the connection.

## Step 7 — SSH Protocol

The client and server perform:

```text
SSH identification
        ↓
Key exchange
        ↓
Host verification
        ↓
Public-key authentication
        ↓
Encrypted SSH session
```

## Step 8 — Shell

The user receives:

```text
[ec2-user@ip-172-31-20-22 ~]$
```

and can execute commands on the EC2 server.

---

# 30. Useful Commands Learned

### Check SSH service

```bash
sudo systemctl status sshd
```

### Restart SSH

```bash
sudo systemctl restart sshd
```

### Validate SSH configuration

```bash
sudo sshd -t
```

### Check listening ports

```bash
sudo ss -tlnp | grep sshd
```

### View SSH logs

```bash
sudo journalctl -u sshd --since "10 minutes ago" --no-pager
```

### Set private-key permissions

```bash
chmod 400 myserver-key.pem
```

### SSH using default port 22

```bash
ssh -i myserver-key.pem ec2-user@SERVER
```

### SSH using port 443

```bash
ssh -p 443 -i myserver-key.pem ec2-user@SERVER
```

### Verbose SSH debugging

```bash
ssh -vvv -i myserver-key.pem ec2-user@SERVER
```

---

# 31. Troubleshooting Methodology Learned

This incident demonstrated an important DevOps troubleshooting principle:

> Do not immediately assume the application is broken. Identify which layer is failing.

The investigation moved through multiple layers:

```text
Application
    ↓
SSH protocol
    ↓
TCP
    ↓
AWS Security Group
    ↓
EC2 networking
    ↓
sshd
    ↓
Operating system
```

For this incident:

```text
EC2 instance       → Healthy
sshd               → Running
Port 22 listening  → Yes
Security Group     → Port 22 allowed
TCP connection     → Successful
SSH negotiation    → Reset
Port 443           → Successful
```

This narrowed the problem considerably.

---

# 32. Final Takeaway

The most important lesson is:

```text
SSH is a protocol.
Port 22 is only its default port.
```

If TCP/22 is unavailable or interfered with, SSH can be configured to listen on another port.

In this case:

```text
Port 22
   ↓
Connection reset during SSH negotiation
   ↓
Investigation
   ↓
Configure sshd on port 443
   ↓
Allow TCP 443 in AWS Security Group
   ↓
SSH over port 443
   ↓
SUCCESS
```

The final working command:

```bash
ssh -p 443 -i ~/Downloads/Bro-AWS/imranserver-key.pem ec2-user@ec2-3-93-194-213.compute-1.amazonaws.com
```

---

# 33. Optional Cleanup / Reverting the Change

If port 443 is no longer needed, remove the line:

```text
Port 443
```

from:

```text
/etc/ssh/sshd_config
```

Then validate:

```bash
sudo sshd -t
```

and restart:

```bash
sudo systemctl restart sshd
```

After that, SSH will return to listening only on the default port 22.

**Keep your existing SSH session open while changing SSH configuration.** This prevents accidentally locking yourself out if a configuration mistake is made.

---

## Final Status

```text
┌───────────────────────────────┐
│       EC2 SSH STATUS          │
├───────────────────────────────┤
│ Instance:       imranserver   │
│ OS:             Amazon Linux  │
│ SSH user:       ec2-user      │
│ Port 22:        Problematic   │
│ Port 443:       WORKING       │
│ SSH key:        Working       │
│ EC2 sshd:       Working       │
│ Authentication: Working       │
└───────────────────────────────┘
```

**Result: EC2 SSH access successfully established from the Ubuntu laptop using TCP port 443.**
