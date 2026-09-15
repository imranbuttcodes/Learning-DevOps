# 🐧 Linux CLI & Bash Fundamentals

A practical and structured reference for learning **Linux Command Line, Bash, system administration fundamentals, and automation** as part of a DevOps learning path.

The goal is not to memorize commands. The goal is to understand **what each command does, why it exists, and how it is used in real development and DevOps workflows**.

---

## 📚 Learning Path

```text
Linux Fundamentals
        │
        ├── CLI / Bash
        ├── Processes
        ├── Permissions
        ├── Package Management
        └── Vim
                │
                ▼
Networking & Security
                │
                ▼
Scripting & Automation
                │
                ▼
Git & Version Control
                │
                ▼
Cloud
                │
                ▼
DevOps
```

---

# 1. 🖥️ CLI & Filesystem

## Basic Navigation

| Command | Description                            | Example          |
| ------- | -------------------------------------- | ---------------- |
| `pwd`   | Displays the current working directory | `pwd`            |
| `ls`    | Lists directory contents               | `ls -la`         |
| `cd`    | Changes the current directory          | `cd /var/log`    |
| `tree`  | Displays directories as a tree         | `tree ~/project` |

### Important Path Concepts

```text
/       → Root directory
~       → Current user's home directory
.       → Current directory
..      → Parent directory
```

### Absolute Path

An absolute path starts from `/`.

```bash
cd /home/user/Documents
```

### Relative Path

A relative path starts from the current directory.

```bash
cd Documents
```

---

# 2. 📁 File & Directory Management

| Command | Description                                    | Example                          |
| ------- | ---------------------------------------------- | -------------------------------- |
| `mkdir` | Creates a directory                            | `mkdir backend`                  |
| `rmdir` | Removes an empty directory                     | `rmdir old_folder`               |
| `touch` | Creates an empty file or updates its timestamp | `touch app.py`                   |
| `cp`    | Copies files/directories                       | `cp app.py backup.py`            |
| `mv`    | Moves or renames files/directories             | `mv old.py new.py`               |
| `rm`    | Removes files/directories                      | `rm file.txt`                    |
| `ln`    | Creates links                                  | `ln -s /var/log/app.log app.log` |
| `file`  | Determines the type of a file                  | `file app.py`                    |
| `stat`  | Displays detailed file metadata                | `stat app.py`                    |

## Useful Options

```bash
ls -l
```

Detailed listing.

```bash
ls -a
```

Shows hidden files.

```bash
ls -la
```

Detailed listing including hidden files.

```bash
cp -r dir1 dir2
```

Copies a directory recursively.

```bash
rm -r folder
```

Removes a directory recursively.

```bash
rm -rf folder
```

Forcefully removes a directory recursively.

> ⚠️ **Be extremely careful with `rm -rf`.** Always verify the path before executing it.

---

# 3. 📄 Reading Files

| Command | Description                              | Example                |
| ------- | ---------------------------------------- | ---------------------- |
| `cat`   | Displays file contents                   | `cat config.txt`       |
| `less`  | Views a file interactively               | `less /var/log/syslog` |
| `more`  | Displays a file page by page             | `more file.txt`        |
| `head`  | Displays the beginning of a file         | `head -n 20 app.log`   |
| `tail`  | Displays the end of a file               | `tail -n 20 app.log`   |
| `nl`    | Displays file contents with line numbers | `nl app.py`            |
| `wc`    | Counts lines, words, and bytes           | `wc -l app.py`         |

## Following Logs

One of the most useful DevOps commands:

```bash
tail -f app.log
```

The `-f` option means **follow**.

It keeps the terminal attached to the file and displays new lines as they are written.

This is commonly used when monitoring application logs.

---

# 4. 🔎 Searching & Text Processing

| Command | Description                           | Example                            |
| ------- | ------------------------------------- | ---------------------------------- |
| `find`  | Searches for files/directories        | `find . -name "*.py"`              |
| `grep`  | Searches for text patterns            | `grep "ERROR" app.log`             |
| `sort`  | Sorts lines                           | `sort names.txt`                   |
| `uniq`  | Removes consecutive duplicate lines   | `sort names.txt \| uniq`           |
| `cut`   | Extracts fields or characters         | `cut -d: -f1 /etc/passwd`          |
| `tr`    | Translates/deletes characters         | `echo "hello" \| tr a-z A-Z`       |
| `sed`   | Performs text transformations         | `sed 's/dev/prod/g' config.txt`    |
| `awk`   | Processes structured text and fields  | `awk '{print $1}' access.log`      |
| `xargs` | Converts input into command arguments | `find . -name "*.tmp" \| xargs rm` |

## `grep`

Suppose `app.log` contains:

```text
INFO Server started
INFO User connected
ERROR Database failed
INFO Request completed
```

Run:

```bash
grep "ERROR" app.log
```

Output:

```text
ERROR Database failed
```

This makes `grep` extremely useful when troubleshooting logs.

---

# 5. 🔀 Pipes & Redirection

Pipes and redirection are **Bash operators rather than standalone commands**, but they are fundamental to Linux CLI usage.

---

## Pipe `|`

Passes the output of one command into another command.

```bash
ps aux | grep python
```

Conceptually:

```text
ps aux
   │
   ▼
output
   │
   ▼
grep python
   │
   ▼
matching processes
```

---

## Output Redirection `>`

Writes output to a file and **overwrites existing content**.

```bash
ls > files.txt
```

---

## Append `>>`

Adds output to the end of a file.

```bash
echo "Server started" >> app.log
```

---

## Input Redirection `<`

Uses a file as input to a command.

```bash
sort < names.txt
```

---

## Error Redirection `2>`

Redirects standard error (`stderr`) to a file.

```bash
python app.py 2> errors.log
```

Normal output remains in the terminal while errors are written to `errors.log`.

---

## Append Errors `2>>`

```bash
python app.py 2>> errors.log
```

Appends errors instead of overwriting the file.

---

## Background Execution `&`

Runs a command in the background.

```bash
python app.py &
```

---

## Conditional Execution `&&`

Runs the second command only if the first command succeeds.

```bash
mkdir project && cd project
```

---

## OR `||`

Runs the second command if the first command fails.

```bash
mkdir project || echo "Could not create directory"
```

---

## Command Separator `;`

Runs commands sequentially regardless of whether previous commands succeed.

```bash
pwd; ls; echo "done"
```

---

## `tee`

Displays output on the terminal while also writing it to a file.

```bash
echo "Server started" | tee app.log
```

Append instead:

```bash
echo "Another event" | tee -a app.log
```

---

# 6. 🌎 Shell & Environment

| Command    | Description                                  | Example                       |
| ---------- | -------------------------------------------- | ----------------------------- |
| `echo`     | Prints text or values                        | `echo "Hello"`                |
| `printf`   | Prints formatted output                      | `printf "User: %s\n" "$USER"` |
| `env`      | Displays environment variables               | `env`                         |
| `printenv` | Displays environment variables               | `printenv PATH`               |
| `export`   | Creates/exports an environment variable      | `export API_KEY="abc"`        |
| `unset`    | Removes an environment variable              | `unset API_KEY`               |
| `which`    | Shows the location of an executable          | `which python3`               |
| `whereis`  | Locates binaries, source, and manuals        | `whereis python3`             |
| `type`     | Determines what type of command something is | `type cd`                     |
| `alias`    | Creates a command shortcut                   | `alias ll='ls -la'`           |
| `history`  | Displays command history                     | `history`                     |
| `source`   | Executes/reloads a shell file                | `source ~/.bashrc`            |

---

# 7. 🛣️ Understanding `PATH`

`PATH` is one of the most important concepts in Linux.

Check it with:

```bash
echo $PATH
```

Example:

```text
/usr/local/bin:/usr/bin:/bin
```

When you type:

```bash
python3
```

Bash searches the directories listed in `$PATH` to find the executable.

You can find the executable with:

```bash
which python3
```

Example:

```text
/usr/bin/python3
```

### Mental Model

```text
You type:

python3
   │
   ▼
Bash checks PATH
   │
   ├── /usr/local/bin
   ├── /usr/bin
   └── /bin
          │
          ▼
      python3 found
          │
          ▼
       Execute
```

---

# 8. 🆘 Getting Help

| Command  | Description                      | Example          |
| -------- | -------------------------------- | ---------------- |
| `man`    | Opens the manual page            | `man chmod`      |
| `help`   | Provides help for Bash built-ins | `help cd`        |
| `--help` | Displays command usage/options   | `ls --help`      |
| `info`   | Provides detailed documentation  | `info coreutils` |

Example:

```bash
man grep
```

Press:

```text
q
```

to exit the manual.

---

# 9. 📦 Package Management

Linux distributions use package managers to install, update, and remove software.

## Ubuntu / Debian

The primary package management tools include:

```text
apt
apt-get
dpkg
```

### Update Package Information

```bash
sudo apt update
```

Refreshes the package repository information.

### Upgrade Installed Packages

```bash
sudo apt upgrade
```

### Install a Package

```bash
sudo apt install nginx
```

### Remove a Package

```bash
sudo apt remove nginx
```

### Search for a Package

```bash
apt search nginx
```

### `dpkg`

`dpkg` is a lower-level Debian package management tool.

Example:

```bash
dpkg -i package.deb
```

---

## Red Hat-Based Systems

You may encounter:

```text
dnf
yum
rpm
```

Modern Fedora/RHEL-based systems commonly use:

```bash
sudo dnf install nginx
```

Older systems may use:

```bash
sudo yum install nginx
```

---

# 10. 👤 Users & Permissions

| Command  | Description                                 | Example                      |
| -------- | ------------------------------------------- | ---------------------------- |
| `whoami` | Shows the current user                      | `whoami`                     |
| `id`     | Shows user and group IDs                    | `id`                         |
| `who`    | Shows logged-in users                       | `who`                        |
| `groups` | Shows the user's groups                     | `groups`                     |
| `sudo`   | Executes a command with elevated privileges | `sudo apt update`            |
| `su`     | Switches user                               | `su - user2`                 |
| `chmod`  | Changes file permissions                    | `chmod 755 script.sh`        |
| `chown`  | Changes file ownership                      | `sudo chown imran script.py` |
| `chgrp`  | Changes group ownership                     | `chgrp developers app.py`    |
| `umask`  | Controls default permissions                | `umask 022`                  |

---

# 11. 🔐 Linux Permissions

A typical permission string looks like:

```text
-rwxr-xr--
```

Break it down:

```text
        owner   group   others
          │       │       │
          ▼       ▼       ▼
         rwx     r-x     r--
```

The permissions are:

```text
r = read
w = write
x = execute
```

## Making a Script Executable

```bash
chmod +x deploy.sh
```

Now the script can be executed:

```bash
./deploy.sh
```

---

# 12. ⚙️ Processes

A Linux system runs many processes simultaneously.

| Command   | Description                          | Example                 |
| --------- | ------------------------------------ | ----------------------- |
| `ps`      | Displays running processes           | `ps aux`                |
| `top`     | Live process monitor                 | `top`                   |
| `htop`    | Interactive process monitor          | `htop`                  |
| `pgrep`   | Finds process IDs                    | `pgrep python`          |
| `pkill`   | Sends signals to matching processes  | `pkill python`          |
| `kill`    | Sends a signal to a process          | `kill 1234`             |
| `killall` | Sends signals to processes by name   | `killall firefox`       |
| `jobs`    | Displays shell jobs                  | `jobs`                  |
| `bg`      | Resumes a job in the background      | `bg %1`                 |
| `fg`      | Brings a job to the foreground       | `fg %1`                 |
| `nohup`   | Keeps a process running after logout | `nohup python app.py &` |

### Finding a Process

```bash
ps aux | grep python
```

### Terminating a Process

```bash
kill 1234
```

Here `1234` is the process ID (PID).

---

# 13. 💻 System Information

| Command    | Description                        | Example            |
| ---------- | ---------------------------------- | ------------------ |
| `uname`    | Kernel/system information          | `uname -a`         |
| `hostname` | Displays system hostname           | `hostname`         |
| `uptime`   | Shows system uptime                | `uptime`           |
| `free`     | Displays memory usage              | `free -h`          |
| `df`       | Displays filesystem disk usage     | `df -h`            |
| `du`       | Displays file/directory disk usage | `du -sh ~/project` |
| `lsblk`    | Lists block/storage devices        | `lsblk`            |
| `lscpu`    | Displays CPU information           | `lscpu`            |
| `lsusb`    | Lists USB devices                  | `lsusb`            |

### `df` vs `du`

`df`:

```bash
df -h
```

Answers:

> How much disk space is available on the filesystem?

`du`:

```bash
du -sh folder/
```

Answers:

> How much disk space is this directory using?

---

# 14. ⏱️ Scheduling & Time

| Command   | Description                       | Example       |
| --------- | --------------------------------- | ------------- |
| `date`    | Displays current date/time        | `date`        |
| `sleep`   | Pauses execution                  | `sleep 5`     |
| `watch`   | Repeatedly executes a command     | `watch df -h` |
| `crontab` | Manages recurring scheduled tasks | `crontab -e`  |

Example:

```bash
watch free -h
```

This repeatedly displays the system's memory usage.

---

# 15. 📝 Vim

Vim is a terminal-based text editor frequently encountered when working with Linux servers.

Open a file:

```bash
vim filename
```

## Vim Modes

```text
              ┌─────────────┐
              │    NORMAL   │
              └──────┬──────┘
                     │ i
                     ▼
              ┌─────────────┐
              │    INSERT   │
              └──────┬──────┘
                     │ Esc
                     ▼
              ┌─────────────┐
              │    NORMAL   │
              └──────┬──────┘
                     │ :
                     ▼
              ┌─────────────┐
              │   COMMAND   │
              └─────────────┘
```

## Essential Vim Commands

| Command   | Description           |
| --------- | --------------------- |
| `i`       | Enter insert mode     |
| `Esc`     | Return to normal mode |
| `:w`      | Save                  |
| `:q`      | Quit                  |
| `:wq`     | Save and quit         |
| `:q!`     | Quit without saving   |
| `dd`      | Delete current line   |
| `yy`      | Copy current line     |
| `p`       | Paste                 |
| `u`       | Undo                  |
| `/search` | Search for text       |

---

# 16. 🌐 Networking Commands

> These commands belong primarily to the **Networking & Security** stage of the learning path. They are listed here for reference but should be studied after Linux fundamentals.

| Command      | Description                             | Example                             |
| ------------ | --------------------------------------- | ----------------------------------- |
| `ip`         | Displays/configures network information | `ip addr`                           |
| `ping`       | Tests network connectivity              | `ping google.com`                   |
| `ss`         | Displays sockets/network connections    | `ss -tuln`                          |
| `curl`       | Makes network/HTTP requests             | `curl https://example.com`          |
| `wget`       | Downloads resources                     | `wget https://example.com/file.zip` |
| `dig`        | Performs DNS queries                    | `dig example.com`                   |
| `nslookup`   | Performs DNS queries                    | `nslookup example.com`              |
| `traceroute` | Shows the path to a destination         | `traceroute example.com`            |
| `ssh`        | Connects to a remote system             | `ssh user@server`                   |
| `scp`        | Copies files over SSH                   | `scp app.py user@server:/tmp/`      |
| `rsync`      | Synchronizes files/directories          | `rsync -av project/ server:/app/`   |

---

# 17. 🐚 Bash Scripting Fundamentals

These are not individual commands, but they form the foundation of Bash automation.

---

## Variables

```bash
name="Imran"
echo "$name"
```

---

## Exit Status

Bash stores the exit status of the previous command in:

```bash
$?
```

Check it with:

```bash
echo $?
```

Typically:

```text
0      → success
non-0  → failure
```

Example:

```bash
ls /does-not-exist
echo $?
```

The result will be a non-zero exit status because the directory does not exist.

---

## Command Substitution

Use:

```bash
$(command)
```

to capture the output of a command.

Example:

```bash
current_dir=$(pwd)
echo "$current_dir"
```

---

## Script Arguments

Suppose:

```bash
./deploy.sh production
```

Inside the script:

```text
$0 → script name
$1 → first argument
$2 → second argument
$@ → all arguments
$# → number of arguments
```

For the example above:

```text
$1 = production
```

---

## Conditions

```bash
if [ "$ENV" = "production" ]; then
    echo "Deploying production"
else
    echo "Not production"
fi
```

---

## Loops

```bash
for server in server1 server2 server3
do
    echo "$server"
done
```

---

## Functions

```bash
deploy() {
    echo "Deploying application"
}

deploy
```

---

# 18. 🎯 CLI/Bash Completion Checklist

Before considering Linux CLI/Bash fundamentals complete, you should be comfortable with:

### Filesystem

* [ ] `pwd`
* [ ] `ls`
* [ ] `cd`
* [ ] Absolute/relative paths
* [ ] `mkdir`
* [ ] `touch`
* [ ] `cp`
* [ ] `mv`
* [ ] `rm`
* [ ] `rmdir`
* [ ] `ln`
* [ ] `file`
* [ ] `stat`

### File Reading

* [ ] `cat`
* [ ] `less`
* [ ] `more`
* [ ] `head`
* [ ] `tail`
* [ ] `nl`
* [ ] `wc`

### Search & Text

* [ ] `find`
* [ ] `grep`
* [ ] `sort`
* [ ] `uniq`
* [ ] `cut`
* [ ] `tr`
* [ ] `sed`
* [ ] `awk`
* [ ] `xargs`

### Bash Operators

* [ ] `|`
* [ ] `>`
* [ ] `>>`
* [ ] `<`
* [ ] `2>`
* [ ] `2>>`
* [ ] `&`
* [ ] `&&`
* [ ] `||`
* [ ] `;`
* [ ] `tee`

### Environment

* [ ] `echo`
* [ ] `printf`
* [ ] `env`
* [ ] `printenv`
* [ ] `export`
* [ ] `unset`
* [ ] `which`
* [ ] `whereis`
* [ ] `type`
* [ ] `alias`
* [ ] `history`
* [ ] `source`
* [ ] `$PATH`

### Help

* [ ] `man`
* [ ] `help`
* [ ] `--help`
* [ ] `info`

### Package Management

* [ ] `apt`
* [ ] `apt-get`
* [ ] `dpkg`
* [ ] `dnf`
* [ ] `yum`
* [ ] `rpm`

### Users & Permissions

* [ ] `whoami`
* [ ] `id`
* [ ] `who`
* [ ] `groups`
* [ ] `sudo`
* [ ] `su`
* [ ] `chmod`
* [ ] `chown`
* [ ] `chgrp`
* [ ] `umask`

### Processes

* [ ] `ps`
* [ ] `top`
* [ ] `htop`
* [ ] `pgrep`
* [ ] `pkill`
* [ ] `kill`
* [ ] `killall`
* [ ] `jobs`
* [ ] `bg`
* [ ] `fg`
* [ ] `nohup`

### System

* [ ] `uname`
* [ ] `hostname`
* [ ] `uptime`
* [ ] `free`
* [ ] `df`
* [ ] `du`
* [ ] `lsblk`
* [ ] `lscpu`
* [ ] `lsusb`

### Scheduling

* [ ] `date`
* [ ] `sleep`
* [ ] `watch`
* [ ] `crontab`

### Vim

* [ ] Normal mode
* [ ] Insert mode
* [ ] Command mode
* [ ] Save/quit
* [ ] Editing
* [ ] Search

---

# 19. 🚀 DevOps Connection

Linux CLI/Bash is not an isolated skill.

It becomes the foundation for almost everything that follows:

```text
                    Linux
                      │
          ┌───────────┼───────────┐
          │           │           │
       Processes   Networking   Files
          │           │           │
          └───────────┼───────────┘
                      │
                 Bash Scripts
                      │
                      ▼
                 Automation
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        Git         Cloud       Docker
          │           │           │
          └───────────┼───────────┘
                      ▼
                    DevOps
```

Linux knowledge allows you to understand what is happening **underneath** your applications, containers, servers, deployments, and cloud infrastructure.

---

# 🗺️ Next Stages

After completing Linux CLI/Bash fundamentals:

```text
STEP 1
Linux Fundamentals
│
├── CLI / Bash
├── Processes
├── Permissions
├── Package Management
└── Vim
        │
        ▼
STEP 2
Networking & Security
│
├── OSI Model
├── TCP/IP
├── IP Addresses
├── Subnetting
├── DNS
├── HTTP / HTTPS
├── SSH
├── Network Troubleshooting
├── Firewalls
├── Proxy Servers
├── Load Balancers
└── Caching
        │
        ▼
STEP 3
Scripting & Automation
│
└── Python
        │
        ▼
STEP 4
Git & Version Control
        │
        ▼
STEP 5
Cloud
│
├── Compute
├── Storage
├── Databases
├── IAM
└── Virtual Networks
        │
        ▼
DEVOPS 🚀
```

---

## 💡 Learning Philosophy

> **Don't memorize commands. Understand the system.**

For every command, aim to know:

1. **What does it do?**
2. **Why would I use it?**
3. **What input does it take?**
4. **What output does it produce?**
5. **How can I combine it with other commands?**
6. **Where would I use it in a real DevOps environment?**

That is what turns Linux knowledge into an actual engineering skill.