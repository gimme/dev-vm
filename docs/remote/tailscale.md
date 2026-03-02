# Tailscale

[Tailscale](https://tailscale.com/) creates a private mesh VPN between your devices — access the VM from anywhere without port forwarding or dynamic DNS.

Requires the optional Tailscale firewall rule in [vlan.md](../vlan.md).

## Install

```sh
curl -fsSL https://tailscale.com/install.sh | sh
```

Don't run `tailscale up` yet — set up ACLs first so the VM never joins the tailnet with full access.

## ACLs

By default, Tailscale allows all devices on your tailnet to reach each other. Without ACLs, the VM can initiate connections to your PC and phone — bypassing the VLAN isolation you set up.

In the [Tailscale admin console](https://login.tailscale.com/admin/acls), replace the default policy:

```jsonc
{
  "acls": [
    {
      // Allow your devices to reach the dev VM
      "action": "accept",
      "src": ["autogroup:admin"],
      "dst": ["tag:dev-vm:*"]
    }
  ],
  "tagOwners": {
    "tag:dev-vm": ["autogroup:admin"]
  }
}
```

Only `autogroup:admin` (you) can assign or remove the `tag:dev-vm` tag. Tagged devices are owned by the tag, not by a user — the VM itself cannot change or remove its tags.

## Connect

Register the VM with its tag:

```sh
sudo tailscale up --advertise-tags=tag:dev-vm
```

Follow the printed URL to authenticate. Once connected, note the VM's Tailscale IP:

```sh
tailscale ip -4
```

Install Tailscale on your other devices and sign in with the same account. Connect via SSH using the Tailscale IP:

```sh
ssh <username>@<tailscale-ip>
```

The VM can respond to incoming connections (SSH, Mosh) but cannot initiate connections to your other devices.

Tailscale normally upgrades to direct peer-to-peer connections for better performance. This won't happen here — the VLAN firewall blocks direct traffic between the VM and your LAN, so Tailscale falls back to its DERP relay servers. For SSH and Mosh the extra hop is negligible.
