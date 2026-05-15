# Setup Linux Environment on Windows and macOS

## Available Options

There are multiple ways to run Linux on a Windows or macOS machine:

| Method | Pros | Cons |
|--------|------|------|
| **Cloud VM** (AWS, GCP, Azure) | Fully isolated, production-like | Costs money, needs internet |
| **WSL2** (Windows only) | Tight Windows integration | Not a full Linux environment |
| **VirtualBox / VMware** | Full OS, GUI support | Heavy on RAM and disk |
| **Hyperkit** (macOS only) | Fast, lightweight | macOS only, limited tooling |
| **Docker Container** | Free, instant, portable, no reboot | No GUI by default |

**Recommended approach: Docker container.**

No cost, no connectivity dependency, starts in seconds, and you can spin up any Linux distribution with a single command. This is also the closest to how Linux is used in real DevOps and cloud environments.

---

## Prerequisites — Install Docker Desktop

Download and install Docker Desktop for your OS:

- **Windows:** https://www.docker.com/products/docker-desktop
- **macOS:** https://www.docker.com/products/docker-desktop

After installation, verify Docker is running:
```bash
docker --version
docker ps
```

---

## Step 1 — Create a Persistent Data Folder

Before running the container, create a local folder that will be mounted inside the container. Any files saved in `/data` inside the container will persist even if the container is removed.

**Windows (PowerShell):**
```powershell
mkdir "C:\Users\<your-username>\Downloads\ubuntu-container"
```

**macOS / Linux:**
```bash
mkdir /tmp/ubuntu-data
```

Replace `<your-username>` with your actual Windows username.

---

## Step 2 — Run the Ubuntu Container

### Windows (PowerShell)

```powershell
docker run -dit `
  --name ubuntu-container `
  --hostname ubuntu-dev `
  --restart unless-stopped `
  --cpus="2" `
  --memory="4g" `
  --mount type=bind,source="C:/Users/<your-username>/Downloads/ubuntu-container",target=/data `
  -v /var/run/docker.sock:/var/run/docker.sock `
  -p 2222:22 `
  -p 8080:80 `
  --env TZ=Asia/Kolkata `
  --env LANG=en_US.UTF-8 `
  ubuntu:latest /bin/bash
```

### macOS / Linux (Terminal)

```bash
docker run -dit \
  --name ubuntu-container \
  --hostname ubuntu-dev \
  --restart unless-stopped \
  --cpus="2" \
  --memory="4g" \
  --mount type=bind,source=/tmp/ubuntu-data,target=/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 2222:22 \
  -p 8080:80 \
  --env TZ=Asia/Kolkata \
  --env LANG=en_US.UTF-8 \
  ubuntu:latest /bin/bash
```

> The only difference between the two commands is the bind mount path and the line continuation character (backtick `` ` `` on PowerShell, backslash `\` on bash).

---

## Parameter Reference

| Parameter | Description |
|-----------|-------------|
| `-dit` | Runs detached (`-d`), interactive (`-i`), with a pseudo-TTY (`-t`) — keeps the container alive and ready for shell access |
| `--name ubuntu-container` | Assigns a human-readable name so you don't have to use the container ID |
| `--hostname ubuntu-dev` | Sets the hostname shown inside the container shell prompt |
| `--restart unless-stopped` | Auto-restarts the container when Docker or the machine restarts, unless you manually stop it |
| `--cpus="2"` | Limits the container to 2 CPU cores to avoid starving your host machine |
| `--memory="4g"` | Caps RAM usage at 4 GB |
| `--mount type=bind,source=...,target=/data` | Mounts a host folder into the container at `/data` — files here survive container removal |
| `-v /var/run/docker.sock:/var/run/docker.sock` | Shares the Docker socket into the container, allowing you to run Docker commands from inside (Docker-in-Docker style) |
| `-p 2222:22` | Maps host port 2222 → container port 22 (SSH); lets you SSH into the container from your host |
| `-p 8080:80` | Maps host port 8080 → container port 80; useful for running web servers inside the container |
| `--env TZ=Asia/Kolkata` | Sets the timezone inside the container — change to your local zone (e.g., `America/New_York`, `Europe/London`) |
| `--env LANG=en_US.UTF-8` | Sets the locale to ensure proper character encoding |
| `ubuntu:latest /bin/bash` | Uses the official latest Ubuntu image and starts a Bash shell as the entry point |

---

## Step 3 — Enter the Container

Once the container is running, open a shell inside it:

```bash
docker exec -it ubuntu-container bash
```

You should see a prompt like:
```
root@ubuntu-dev:/#
```

You are now inside a full Ubuntu Linux environment.

---

## Step 4 — First-Time Setup Inside the Container

The base Ubuntu image is minimal. Run these commands after first entry to set up a usable environment:

```bash
# Update package lists
apt update

# Install essential tools
apt install -y curl wget git vim nano net-tools iputils-ping sudo

# Verify
bash --version
```

---

## Managing the Container

| Task | Command |
|------|---------|
| Start the container | `docker start ubuntu-container` |
| Stop the container | `docker stop ubuntu-container` |
| Restart the container | `docker restart ubuntu-container` |
| Enter the container shell | `docker exec -it ubuntu-container bash` |
| View container logs | `docker logs ubuntu-container` |
| Check resource usage | `docker stats ubuntu-container` |
| Remove the container | `docker rm -f ubuntu-container` |
| List all containers | `docker ps -a` |

---

## Timezone Reference

Change `--env TZ=Asia/Kolkata` to match your location:

| Region | Timezone Value |
|--------|---------------|
| India | `Asia/Kolkata` |
| US East | `America/New_York` |
| US West | `America/Los_Angeles` |
| UK | `Europe/London` |
| Germany | `Europe/Berlin` |
| Singapore | `Asia/Singapore` |
| Australia (Sydney) | `Australia/Sydney` |

---

## Why Not WSL2?

WSL2 is convenient but has limitations that make it unsuitable for serious Linux learning:

- File system performance is slow for Linux-native paths accessed from Windows
- Networking is managed by Windows, not Linux — some network tools behave differently
- Systemd support was only added recently and still has quirks
- Not portable — you can't replicate the same environment on a server

A Docker container gives you a real, isolated Linux environment that behaves identically to what you'd find on a cloud server.

---

**← Back to [Linux Distributions](../linux-distributions/README.md)**  
**← Back to [Chapter 1: Getting Started](../README.md)**  
**Next → Chapter 2: The Linux Filesystem**
