# Cloud Support Lab

A hands-on, week-long lab building foundational cloud support / DevOps skills:
Linux administration, process and service troubleshooting, networking, Docker,
and a small containerized Python service deployed and debugged end-to-end.

Environment: Ubuntu (WSL2) on Windows.

---

## Day 1 — Lab Setup

**Goal:** get a working Linux environment and practice the fundamentals:
SSH, directory navigation, files & permissions, environment variables, `sudo`.

### SSH basics

| Command | What it does | Notes |
|---|---|---|
| `sudo apt install openssh-server -y` | Installs the OpenSSH server package | Needed before you can SSH into the machine at all |
| `sudo service ssh start` | Starts the SSH daemon | WSL doesn't auto-start services on boot like a normal Linux install would |
| `ssh asus@localhost` | Connects via SSH to this same machine | First connection prompts to accept the host's key fingerprint (`yes`), then asks for the password |
| `exit` | Closes the current shell / SSH session | Prints "Connection to localhost closed" when leaving an SSH session |
| `ssh -v localhost` | Same connection, but verbose | Shows the handshake step by step: key exchange, auth method offered, auth success, exit status |

### Directory navigation

| Command | What it does | Notes |
|---|---|---|
| `pwd` | Prints the current working directory | Home directory is `/home/asus` |
| `ls -la` | Lists all files, including hidden dotfiles, with details | `-l` = long format (permissions, owner, size, date), `-a` = show hidden |
| `cd ..` | Moves up one directory level | |
| `cd ~` | Jumps straight back to the home directory | Plain `cd` with no argument does the same thing |
| `mkdir testdir` | Creates a new directory | |
| `cd testdir` / `cd ..` | Moves into and back out of it | |

### Files and permissions

| Command | What it does | Notes |
|---|---|---|
| `touch file.txt` | Creates an empty file | |
| `ls -l file.txt` | Shows permission string, owner, group, size | Default was `-rw-r--r--` |
| `chmod 644 file.txt` | Sets permissions to owner read/write, everyone else read-only | `6=rw`, `4=r`, `4=r` (r=4, w=2, x=1 added per group) |
| `chmod 600 file.txt` | Sets permissions to owner read/write, no access for group/others | Confirmed via `ls -l` → `-rw-------` |
| `whoami` | Prints the current logged-in user | `asus` |
| `chown asus:asus file.txt` | Sets the file's owner and group | No visible change here since the file was already owned by `asus` — this is the command you'd use to hand a file to a different user/group |

### Environment variables

| Command | What it does | Notes |
|---|---|---|
| `echo $HOME` | Prints the value of the `HOME` variable | `/home/asus` |
| `printenv` | Lists all currently exported environment variables | WSL's `PATH` is huge because it merges Linux and Windows paths |
| `export MY_VAR=hello` | Creates a variable and exports it to child processes | Without `export`, a variable stays local to the current shell only |
| `echo $MY_VAR` | Prints the variable's value | `hello` |
| `bash` then `echo $MY_VAR` | Launches a nested child shell and checks the variable there | Still printed `hello` — proves `export` makes it inherit into child processes |
| `exit` | Leaves the nested shell, back to the parent | |

### sudo

| Command | What it does | Notes |
|---|---|---|
| `sudo whoami` | Runs a single command as root | Printed `root`, confirming elevation worked |
| `sudo -l` | Lists what the current user is allowed to run via sudo | Returned `(ALL : ALL) ALL` — this user can run any command as any user |
| `su` | Tries to switch fully to the root user | Failed with "Authentication failure" — expected, since Ubuntu disables the root account's password by default and expects `sudo` to be used instead of `su` |

**Key distinction learned:** `sudo <command>` runs one command as root using *your own* password. `su` tries to become the root user directly, which requires *root's* password — one that doesn't exist on a default Ubuntu install, which is exactly why it failed.

---

## Day 2 — Processes and Services

**Goal:** answer, from the command line: is the process running, is the
server out of disk space, is memory exhausted, what does the service log say.

| Command | What it does | Notes |
|---|---|---|
| `ps aux` | Snapshot of every running process | `%CPU`/`%MEM` show resource usage, `COMMAND` shows what's running |
| `ps aux \| grep <name>` | Filters the process list to just matches for `<name>` | Practical way to answer "is process X running?" without scrolling everything |
| `top` | Live, auto-refreshing view of running processes | `q` to exit |
| `free -h` | Shows memory usage in human-readable units | Had 7.6GB total RAM, only 615MB used — not memory-exhausted |
| `df -h` | Shows disk space per mounted filesystem | Root filesystem (`/`) was only 1% used — plenty of space |
| `du -sh ~` | Total size of the home directory | Home dir was only 148K, essentially empty |
| `systemctl status <service>` | Shows whether a systemd service is running, plus recent log lines inline | SSH showed `inactive (dead)` but `TriggeredBy: ssh.socket` — socket-activated, starts on demand rather than always running |
| `sudo systemctl start/stop <service>` | Starts or stops a systemd-managed service | Confirmed status flipped between `active (running)` and `inactive (dead)` |
| `journalctl -u <service> -n 20` | Shows the last 20 log lines for one specific systemd unit | This is what answers "what does the service log say?" in a real investigation, beyond just the current snapshot |

**Service start/stop practice:**
- Used the `ssh` service (installed on Day 1) as the practice target since it's a real, already-installed systemd unit
- Learned SSH on this system is socket-activated: `systemctl status ssh` shows `inactive (dead)` at rest, but `TriggeredBy: ssh.socket` means systemd is still listening on port 22 and will spin the service up automatically on an incoming connection
- Forcing it with `sudo systemctl start ssh` showed `active (running)`, `Main PID`, and log lines like "Server listening on 0.0.0.0 port 22" directly in the status output
- `sudo systemctl stop ssh` cleanly returned it to `inactive (dead)`

---

## Day 3 — Linux Networking

**Goal:** inspect network config, run a local web server, identify its port,
block and restore access.

| Command | What it does | Notes |
|---|---|---|
| `ip addr` | | |
| `ip route` | | |
| `ping` | | |
| `curl` | | |
| `ss -tulpn` | | |
| `nslookup` | | |

---

## Day 4 — Docker Fundamentals

**Concepts:**
- Image —
- Container —
- Port mapping —
- Volume —
- Network —

**Practice:** ran the official Nginx container, accessed it from the browser at `http://localhost:<port>`.

---

## Day 5 — Containerize a Tiny Python Service

A minimal Python API with a `/health` endpoint, packaged with Docker.

- [`app/`](./app) — the service source
- [`app/Dockerfile`](./app/Dockerfile)
- [`app/requirements.txt`](./app/requirements.txt)

Run instructions: see [`app/README.md`](./app/README.md).

---

## Day 6 — Break and Troubleshoot It

Deliberate failures introduced, then diagnosed with `docker ps`, `docker logs`,
`docker inspect`, `curl`, `ss -tulpn`.

| Problem | Evidence | Fix | Prevention |
|---|---|---|---|
| Wrong port mapped | | | |
| Container stopped | | | |
| Missing environment variable | | | |
| Invalid configuration | | | |

---

## Day 7 — Wrap-up

### Screenshot

_(service running, added here)_

### What I learned

-

---

## Repo Contents

- `app/` — Python service, Dockerfile, requirements.txt
- `README.md` — this file: setup notes, command log, troubleshooting log, and takeaways
