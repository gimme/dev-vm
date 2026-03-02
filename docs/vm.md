# VM

VM setup on Unraid using the [VLAN](./vlan.md) for isolation.

## Prerequisites

- VLAN configured on Unraid: enable VLAN and add id `50` (br0.50) in network settings (requires temporarily disabling Docker and VMs)
- Download the latest [Ubuntu Server LTS](https://ubuntu.com/download/server) ISO and place it in Unraid's `isos` share
- Make sure the `/mnt/user/domains` share lives on your cache disk (better VM performance)

## Create VM

VMs → Add VM → Ubuntu.

Adjust resources according to your hardware and needs.

| Setting | Value | Notes |
|---|---|---|
| vCPUs | 4 | Helps with parallel builds |
| Initial Memory | 4096 MB | Enough for light tasks and OS overhead |
| Max Memory | 16384 MB | Headroom for builds, Docker, and agents |
| OS Install ISO | *(the Ubuntu Server LTS ISO)* | |
| Primary vDisk Size | 100G | Enough for projects and Docker images |
| Network Source | br0.50 | Must match the VLAN interface |
| Network Model | virtio | Might offer better performance |

Complete the Ubuntu installation. Select "Install OpenSSH server" when prompted.

## Post-install

- Assign a static IP for the VM in your router (for consistent SSH access)
- Save a snapshot of the VM (for quick rebuilds from scratch)

## Rebuild

Restore the snapshot and start the VM. Follow [ubuntu.md](./ubuntu.md) to set up the dev environment.
