# 1. `docker logs` — See what the container's application is saying

Think:

> **Container is running → what is happening inside it?**

You use:

```bash
docker logs <container>
```

For example:

```bash
docker logs my-api
```

This shows the container's **stdout/stderr output** — basically the output that the main process inside the container is writing.

### Example

Suppose your FastAPI container starts:

```text
INFO:     Started server process [1]
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000
```

Then:

```bash
docker logs my-api
```

might show:

```text
INFO:     Started server process [1]
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000
```

If your application crashes:

```text
ModuleNotFoundError: No module named 'langchain'
```

you can find that with:

```bash
docker logs my-api
```

### Very useful options

Follow logs live:

```bash
docker logs -f my-api
```

`-f` = **follow**

It's similar to watching logs continuously.

Stop following with:

```text
Ctrl + C
```

Show only recent logs:

```bash
docker logs --tail 50 my-api
```

Show timestamps:

```bash
docker logs -t my-api
```

---

# 2. `docker exec` — Run a command inside a running container

Think:

> **I want to enter/interact with the container and execute something inside it.**

Syntax:

```bash
docker exec <container> <command>
```

Example:

```bash
docker exec my-api ls
```

Docker asks the daemon:

```text
"Run ls inside my-api"
```

The command executes **inside the container**, not your host.

---

## The most important `docker exec` command

Interactive shell:

```bash
docker exec -it my-api bash
```

If the image doesn't contain Bash, often:

```bash
docker exec -it my-api sh
```

Now you get something like:

```text
root@a8f31c:/app#
```

You are now operating **inside the container**.

For example:

```bash
ls
```

might show:

```text
app.py
requirements.txt
```

And:

```bash
python --version
```

might show:

```text
Python 3.12.3
```

You can inspect things:

```bash
env
```

```bash
pwd
```

```bash
ps
```

etc.

Exit:

```bash
exit
```

You're back on your Ubuntu host.

---

# `docker logs` vs `docker exec`

This distinction is **very important**:

| Command       | Purpose                                                         |
| ------------- | --------------------------------------------------------------- |
| `docker logs` | See what the container's main application/process is outputting |
| `docker exec` | Execute a command inside a running container                    |

Think:

```text
                 Docker Container
                ┌───────────────────┐
                │                   │
docker logs ──→ │   FastAPI         │ ──→ application output
                │                   │
docker exec ──→ │   filesystem      │
                │   shell           │
                │   processes       │
                └───────────────────┘
```

### Real-world debugging

Your API isn't working.

First:

```bash
docker ps
```

Find the container.

Then:

```bash
docker logs my-api
```

Maybe you discover:

```text
Connection refused
```

Then you might inspect the container:

```bash
docker exec -it my-api bash
```

and investigate:

```bash
env
```

```bash
ls
```

```bash
ps
```

So the workflow is often:

```text
docker ps
    ↓
docker logs
    ↓
Understand the error
    ↓
docker exec
    ↓
Inspect container
    ↓
Fix Dockerfile/config/application
    ↓
Rebuild + recreate container
```

### One important detail

`docker exec` requires the container to be **running**.

If the container is stopped:

```bash
docker exec my-api bash
```

won't work.

But:

```bash
docker logs my-api
```

can still show logs from a stopped container — which is extremely useful when a container **starts and immediately crashes**.

So remember:

> **`docker logs` = observe the application.**
> **`docker exec` = interact with the running container.**
