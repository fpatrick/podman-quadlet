
# Podman with Quadlet - Getting Started Guide 🐋

This guide will help you get started with **Podman** and **Quadlet** in a simple, non-technical way. You'll learn how to manage containers with Quadlet using `.container` files. 🚀

## What is Quadlet?

Quadlet is a way to run and manage containers in Podman using systemd services. You write `.container` files, and systemd takes care of running and managing the container.

## Rootless Setup

Rootless means you don't need admin (root) permissions to run containers. If something malicious break out of the container, it won't do so much damage.

### Step-by-Step Guide

1. **Set up the directories** 📂:
   ```
   mkdir -p ~/.config/containers/systemd/
   ```
   Later, put your `.container`, `.network`, and `.env` files in this directory.

2. **Create or modify a .container file** 🛠️:
   ```
   nano ~/.config/containers/systemd/myapp.container
   ```
   This is where you define your container. Example template below.

3. **Reload systemd** 🔄:
   ```
   systemctl --user daemon-reload
   ```

4. **Prepare persistent storage** 🗂️ (important):
   Before starting the container, create the directories for persistent storage.
   ```
   mkdir -p /path/to/storage/containerfolder
   ```

5. **Start the container** ▶️:
   ```
   systemctl --user start myapp.service
   ```

6. **Troubleshooting** ❗:
   If something goes wrong, you can try to use this command to check logs:
   ```
   journalctl --user -u myapp.service --no-pager -n 50
   ```

## Rootful Setup (Admin Access)

In rootful mode, you need admin (root) permissions.

1. **Use sudo** 🛑: Prefix every command with `sudo`.
2. **Change directory for container files** 📁: Put your `.container` files in `/etc/containers/systemd/`.
3. **Run commands**:
   - Same as rootless, but **without** the `--user` flag:
     ```
     sudo systemctl start myapp.service
     ```

## Updates

### Auto-updating Containers 🔄

To automatically update your containers:

1. Add the line `AutoUpdate=registry` in your `.container` file.
2. Enable the Podman auto-update service:
   ```
   systemctl --user enable podman-auto-update
   ```

### Manual Updates 🔧

1. Pull the latest image:
   ```
   podman pull docker.io/my-image:latest
   ```
2. Restart the container:
   ```
   systemctl --user restart myapp.service
   ```

## Example Templates

### .container file template

```
[Unit]
Description=  # (Optional) A brief description of the service
Wants=        # (Optional) Services you want to run with this one
After=        # (Optional) Services that need to start before this one

[Container]
ContainerName=  # (Mandatory) The container's name
Image=          # (Mandatory) The container image to use (e.g., docker.io/library/alpine)
EnvironmentFile= # (Optional) Path to an .env file
Environment=    # (Optional) Key=value pairs for environment variables
Volume=         # (Optional) Persistent storage paths (host:container)
Network=        # (Optional) Custom network for the container
PublishPort=    # (Optional) Ports to expose (host:container)
Exec=           # (Optional) Custom command to run in the container
PodmanArgs=     # (Optional) Additional Podman arguments
AddCapability=  # (Optional) Extra capabilities to add to the container
AddDevice=      # (Optional) Add host devices to the container
SecurityLabelDisable= # (Optional) Disable SELinux labels
User=           # (Optional) Run as a specific user inside the container
Label=          # (Optional) Add metadata labels to the container
UIDMap=         # (Optional) User ID mapping. Example: 0:10000:10 (Inside:Outside:Range)
GIDMap=         # (Optional) Group ID mapping Example: 0:10000:10 (Inside:Outside:Range)

[Service]
Restart=        # (Optional) Set to 'always' or 'on-failure' to restart on failure
TimeoutStartSec= # (Optional) Time to wait before considering a failure

[Install]
WantedBy=       # (Optional) Target to start with (default: multi-user.target). For graphical user interface systems default.target
```

### .network file template

For setting up custom container networks:

```
[Network]
Subnet=192.168.99.0/24  # (Mandatory) Subnet for the network
Gateway=192.168.99.1    # (Mandatory) Gateway IP address
Label                   # (Optional) Custom label for the network
```

### .env file template

Define environment variables:

```
MY_VARIABLE=my_value  # Add your custom variables here
```

---

That's it! You're ready to manage containers with Quadlet. 😊
