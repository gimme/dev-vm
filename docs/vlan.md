# VLAN

VLAN setup on an EdgeOS router (e.g. Ubiquiti EdgeRouter). Change the values below to match your setup.

## Assumptions

- Router running EdgeOS with LAN on `switch0` and WAN on `eth0`
- LAN subnet `192.168.0.0/24`, router at `192.168.0.1` with user `admin`
- VLAN id `50`, subnet `10.0.50.0/24`

## Goal

Create an isolated network for the dev VM that:
- Has internet access (HTTP, HTTPS, SSH, ICMP)
- Cannot reach the main LAN
- Gets its own DHCP and DNS via the router

The commands below implement this on EdgeOS. For other routers, replicate the same rules.

## Connect to the router

```sh
ssh admin@192.168.0.1
```

## Configure the VLAN

```sh
# Start edit mode
configure

# ----- VLAN -----

# Create VLAN 50 on the LAN interface
set interfaces switch switch0 vif 50 address 10.0.50.1/24
set interfaces switch switch0 vif 50 description "Dev VM"

# Create DHCP server for the VLAN
set service dhcp-server shared-network-name DEV_VLAN subnet 10.0.50.0/24 default-router 10.0.50.1
set service dhcp-server shared-network-name DEV_VLAN subnet 10.0.50.0/24 dns-server 10.0.50.1
set service dhcp-server shared-network-name DEV_VLAN subnet 10.0.50.0/24 start 10.0.50.10 stop 10.0.50.50
set service dhcp-server shared-network-name DEV_VLAN subnet 10.0.50.0/24 lease 86400

# Make sure DNS listens on the new VLAN
set service dns forwarding listen-on switch0.50

# Create a NAT rule for the VLAN (so it can access the internet; 5001 is an arbitrary rule number)
set service nat rule 5001 outbound-interface eth0
set service nat rule 5001 type masquerade
set service nat rule 5001 description "NAT for Dev VLAN"
set service nat rule 5001 source address 10.0.50.0/24


# ----- Firewall in — controls traffic to everywhere else -----

# Default: drop
set firewall name DEV_IN default-action drop

# Allow established/related connections (so responses come back)
set firewall name DEV_IN rule 10 action accept
set firewall name DEV_IN rule 10 description "Allow established/related"
set firewall name DEV_IN rule 10 state established enable
set firewall name DEV_IN rule 10 state related enable

# Block access to main LAN (before the other allow-rules)
set firewall group network-group MAIN_LAN network 192.168.0.0/24
set firewall name DEV_IN rule 15 action drop
set firewall name DEV_IN rule 15 description "Block LAN"
set firewall name DEV_IN rule 15 destination group network-group MAIN_LAN

# Allow HTTPS (general internet access)
set firewall name DEV_IN rule 20 action accept
set firewall name DEV_IN rule 20 description "Allow HTTPS"
set firewall name DEV_IN rule 20 protocol tcp_udp
set firewall name DEV_IN rule 20 destination port 443

# Allow HTTP (some registries, apt repos)
set firewall name DEV_IN rule 30 action accept
set firewall name DEV_IN rule 30 description "Allow HTTP"
set firewall name DEV_IN rule 30 protocol tcp
set firewall name DEV_IN rule 30 destination port 80

# Allow SSH (git@github.com)
set firewall name DEV_IN rule 40 action accept
set firewall name DEV_IN rule 40 description "Allow SSH"
set firewall name DEV_IN rule 40 protocol tcp
set firewall name DEV_IN rule 40 destination port 22

# Allow ICMP (ping)
set firewall name DEV_IN rule 50 action accept
set firewall name DEV_IN rule 50 description "Allow ICMP"
set firewall name DEV_IN rule 50 protocol icmp

# Optional: allow Tailscale (only if using Tailscale for remote access)
set firewall name DEV_IN rule 60 action accept
set firewall name DEV_IN rule 60 description "Allow Tailscale"
set firewall name DEV_IN rule 60 protocol udp
set firewall name DEV_IN rule 60 destination port 41641


# ----- Firewall local — controls traffic to the router itself -----

# Default: drop
set firewall name DEV_LOCAL default-action drop

# Allow DNS (resolve hostnames)
set firewall name DEV_LOCAL rule 10 action accept
set firewall name DEV_LOCAL rule 10 description "Allow DNS"
set firewall name DEV_LOCAL rule 10 protocol udp
set firewall name DEV_LOCAL rule 10 destination port 53

# Allow DHCP (get an IP address)
set firewall name DEV_LOCAL rule 20 action accept
set firewall name DEV_LOCAL rule 20 description "Allow DHCP"
set firewall name DEV_LOCAL rule 20 protocol udp
set firewall name DEV_LOCAL rule 20 destination port 67


# ----- Apply -----

# Apply the firewalls to the VLAN Interface
set interfaces switch switch0 vif 50 firewall in name DEV_IN
set interfaces switch switch0 vif 50 firewall local name DEV_LOCAL

# Apply changes
commit
# Persist changes
save
```
