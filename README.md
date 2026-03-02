# Dev VM

A guide for setting up an isolated development VM on its own VLAN as a sandbox where AI agents can yolo. Connect via SSH from your PC. Keep sensitive data out of the VM.

This guide uses Unraid and an EdgeOS router, but the approach works with any hypervisor and router that supports VLANs.

## Why

AI coding agents need shell access, network access, and the freedom to install packages and run arbitrary commands. Giving them that on your main machine means they're one bad command away from your personal files, credentials, and home network devices.

A dedicated VM solves the blast radius problem. A VLAN goes further — even if the VM is compromised, it can't reach anything on your LAN. The VM gets internet access for pulling packages and pushing code, but nothing else. You SSH in from your PC, keep secrets out, and treat the whole thing as disposable. If something goes wrong, restore and start fresh.

## VLAN

See [vlan.md](docs/vlan.md) for instructions on how to set up the VLAN on the router.

## VM

See [vm.md](docs/vm.md) for instructions on how to create the VM.

## Connect to VM

Access via SSH using the username from your Ubuntu install and the VM's IP (e.g. `10.0.50.10`):
```sh
ssh <username>@<vm-ip>
```

Verify firewall:
```sh
ping 8.8.8.8 # Internet works
ping 192.168.0.1 # LAN fails
```

## Configure Dev Environment

See [ubuntu.md](docs/ubuntu.md) for an example setup of the development environment on Ubuntu.

## Remote Access

Connect to the VM from anywhere. Choose one:

| | Tailscale | WireGuard (Unraid) |
|---|---|---|
| Direct connection | No (DERP relay) | Yes |
| Open ports | None | 1 UDP port forwarded |
| VM gains access | No (ACLs) | No (not a peer) |
| Third-party dependency | Yes (Tailscale account) | No (self-hosted) |
| Setup | Minimal to moderate | More involved |
| Dynamic DNS | Not needed | Needed |

- [Tailscale](docs/remote/tailscale.md) — zero-config mesh VPN, no port forwarding
- [WireGuard](docs/remote/wireguard.md) — direct connections via Unraid's built-in WireGuard server

For phone access, see [mobile.md](docs/remote/mobile.md).
