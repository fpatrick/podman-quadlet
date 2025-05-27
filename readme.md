Check out my blog for guides on podman, fedora core os, SELinux in containers and more: https://blog.nerdon.eu/tag/containers-virtualization/

# Podman with Quadlet - Getting Started Guide

This guide will help you get started with **Podman** and **Quadlet** in a simple, non-technical way. Scroll down for TEMPLATES for `.container`, `.network` and `.env` .

**Did you know? You can use Cockpit or Portainer with podman**

## What is Quadlet?

Quadlet is a way to run and manage containers in Podman using systemd services. You write `.container` files, and systemd takes care of running and managing the container.

## Rootless Setup

Rootless means you don't need admin (root) permissions to run containers. If something malicious break out of the container, it won't do so much damage.

### Step-by-Step Guide

1. **Set up the directories**:
   ```
   mkdir -p ~/.config/containers/systemd/
   ```
   Later, put your `.container`, `.network`, and `.env` files in this directory.

2. **Create or modify a .container file** 🛠️:
   ```
   nano ~/.config/containers/systemd/myapp.container
   ```
   This is where you define your container. Example template below.

3. **Reload systemd**:
   ```
   systemctl --user daemon-reload
   ```

4. **Prepare persistent storage** (important):
   Before starting the container, create the directories for persistent storage.
   ```
   mkdir -p /path/to/storage/containerfolder
   ```

5. **Start the container**:
   ```
   systemctl --user start myapp.service
   ```

6. **Troubleshooting**:
   If something goes wrong, you can try to use this command to check logs:
   ```
   journalctl --user -u myapp.service --no-pager -n 50
   ```

## Rootful Setup (Admin Access)

In rootful mode, you need admin (root) permissions.

1. **Use sudo**: Prefix every command with `sudo`.
2. **Change directory for container files**: Put your `.container` files in `/etc/containers/systemd/`.
3. **Run commands**:
   - Same as rootless, but **without** the `--user` flag:
     ```
     sudo systemctl start myapp.service
     ```

## Updates

### Auto-updating Containers

To automatically update your containers:

1. Add the line `AutoUpdate=registry` in your `.container` file.
2. Enable the Podman auto-update service:
   ```
   systemctl enable --now podman-auto-update.timer
   ```
   That will make podman periodically check for new images. If you want to run a check for updates run ``` podman auto-update ```

   You can check if the timer is active with ``` systemctl list-timers | grep podman-auto-update ```

### Manual Updates

1. Pull the latest image:
   ```
   podman pull docker.io/my-image:latest
   ```
2. Restart the container:
   ```
   systemctl --user restart myapp.service
   ```

Another way is follow the Auto-update instructions, but don't activate the auto-update timer, instead run podman auto-update

## Example Templates

#### In production use # Comments on top of the line (like below) and not in front!

### .container file template

```
[Unit]
# (Optional) A brief description of the service
Description=
# (Optional) Services you want to run with this one
Wants=
# (Optional) Services that need to start before this one
After=

[Container]
# (Mandatory) The container's name
ContainerName=
# (Mandatory) The container image to use (e.g., docker.io/library/alpine)
Image=
# (Optional) Path to an .env file
EnvironmentFile=
# (Optional) Key=value pairs for environment variables
Environment=
# (Optional) Persistent storage paths (host:container)
Volume=
# (Optional) Custom network for the container
Network=
# (Optional) Ports to expose (host:container)
PublishPort=
# (Optional) Custom command to run in the container
Exec=
# (Optional) Additional Podman arguments
PodmanArgs=
# (Optional) Extra capabilities to add to the container
AddCapability=
# (Optional) Add host devices to the container
AddDevice=
# (Optional) Disable SELinux labels
SecurityLabelDisable=
# (Optional) Run as a specific user inside the container
User=
# (Optional) Add metadata labels to the container
Label=
# (Optional) User ID mapping. Example: 0:10000:10 (Inside:Outside:Range)
UIDMap=
# (Optional) Group ID mapping Example: 0:10000:10 (Inside:Outside:Range)
GIDMap=

[Service]
# (Optional) Set to 'always' or 'on-failure' to restart on failure
Restart=
# (Optional) Time to wait before considering a failure
TimeoutStartSec=

[Install]
# (Optional) Target to start with (default: multi-user.target). For graphical user interface systems default.target
WantedBy=
```

### .network file template

For setting up custom container networks:

```
[Network]
# (Mandatory) Subnet for the network
Subnet=192.168.99.0/24  
# (Mandatory) Gateway IP address
Gateway=192.168.99.1    
# (Optional) Custom label for the network
Label=              
```

### .env file template

Define environment variables:

```
ENVIROMENT_FIELD=your_secret_value # Add your custom variables here. Such as PGID=200
```

---

That's it! You're ready to manage containers with Quadlet.
