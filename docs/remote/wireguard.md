# WireGuard (Unraid)

Unraid includes a built-in WireGuard server. Since it runs on the same host as the VM, connections route directly to the VM through the hypervisor's virtual network — no relay servers, no hairpin NAT.

The VM doesn't participate in WireGuard. It's not a peer and has no WireGuard client. Traffic flows from your device through the tunnel to Unraid, which routes it to the VM. The VLAN firewall still controls what the VM can initiate — no firewall changes needed for this approach.

## Dynamic DNS

If your ISP assigns a dynamic public IP, set up a DDNS service so your devices can find your network when you're away from home. Options include [DuckDNS](https://www.duckdns.org/) (free), a [Cloudflare](https://www.cloudflare.com/) DNS-only record (gray cloud, not proxied) updated via API, or whatever your router supports natively. EdgeOS has built-in DDNS support under `service dns dynamic`.

## Router

Forward WireGuard's UDP port from the router's public IP to Unraid's LAN IP. The default port is `51820` but you can choose any port when creating the tunnel in Unraid.

WireGuard is designed for public exposure — it's silent to unauthenticated packets (port scans get no response) and uses cryptokey routing.

## Unraid

In the Unraid web UI under **Settings** → **VPN Manager**:

1. Add a new tunnel named "Home VPN" (or whatever) and set the `Local endpoint` to your server address with port `51820` (e.g. `vpn.example.com:51820`). This is the address your devices will connect to.
2. Add a peer for each device (PC, phone) and choose **Remote access to LAN** as the peer type.

### Access to the VLAN

Each peer controls what of their own traffic gets routed through the tunnel (`AllowedIPs` on the CLIENT side). You need to include the VM's VLAN subnet (and optionally remove `192.168.0.0/24` if you don't need main LAN access):

```ini
[Peer]
AllowedIPs = <tunnel-subnet>, 10.0.50.0/24
```

This routes only VLAN and tunnel traffic through WireGuard. Your device's other traffic goes direct.

## Connect

Import the peer config into a WireGuard client on your device:

- **PC** — [WireGuard client](https://www.wireguard.com/install/)
- **Phone** — WireGuard app ([App Store](https://apps.apple.com/app/wireguard/id1441195209) / [Play Store](https://play.google.com/store/apps/details?id=com.wireguard.android))

Activate the tunnel, then SSH to the VM's VLAN IP:

```sh
ssh <username>@<vm-ip>
```
