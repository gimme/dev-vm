# Mobile Access

Connect to the VM from your phone over SSH. Useful for steering AI agents, quick fixes, and code review on the go. Not a replacement for a real workstation — the workflow leans on natural language prompts and reviewing generated code rather than heavy typing.

## Prerequisites

- VM reachable remotely via [Tailscale](./tailscale.md) or [WireGuard](./wireguard.md)
- mosh and tmux installed on the VM (included in the [Ubuntu setup](../ubuntu.md#essentials))

## Mosh

SSH over mobile connections drops constantly — network switches, device sleep, cellular handoffs. [Mosh](https://mosh.org/) handles all of this transparently and picks up where it left off.

Mosh uses UDP ports 60000–61000. When connecting through Tailscale or WireGuard, Mosh traffic is encapsulated inside the tunnel, so no additional firewall rules are needed on the router.

## Phone Setup

Install your VPN app on your phone — Tailscale ([App Store](https://apps.apple.com/app/tailscale/id1470499037) / [Play Store](https://play.google.com/store/apps/details?id=com.tailscale.ipn)) or WireGuard ([App Store](https://apps.apple.com/app/wireguard/id1441195209) / [Play Store](https://play.google.com/store/apps/details?id=com.wireguard.android)). Sign in or import your config.

If password authentication is disabled on the VM, each device needs an SSH key. Generate one on your phone, add the public key to your [GitHub SSH keys](https://github.com/settings/keys), then re-import on the VM:

```sh
ssh-import-id gh:$GITHUB_USERNAME
```

### iOS

Install [Blink Shell](https://blink.sh/). Supports Mosh natively and includes a key management system.

Generate an SSH key: Settings → Keys → **+** → Create New (Ed25519). Tap the key and copy the public key, then add it to GitHub.

Connect:

```
mosh <username>@<vm-ip>
```

### Android

Install [Termux](https://termux.dev/). Then install Mosh and generate an SSH key:

```sh
pkg install -y mosh
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub
```

Copy the public key and add it to GitHub.

Connect:

```sh
mosh <username>@<vm-ip>
```

## Usage

Once connected, attach to an existing tmux session or start a new one:

```sh
tmux attach || tmux new -s mobile
```

Run Claude Code inside the session:

```sh
claude
```

If your connection drops, reconnect with the same `mosh` command and reattach with `tmux attach`. Nothing is lost.

<details>
<summary>Running multiple tasks</summary>

Split tmux panes to run Claude Code alongside other tools:

| Action | Keys |
|---|---|
| Vertical split | `prefix %` |
| Horizontal split | `prefix "` |
| Navigate panes | `prefix arrow` |
| Maximize/restore pane | `prefix z` |

`prefix` is `Ctrl+b` by default. Maximizing a pane (`prefix z`) is especially useful on a small screen — expand the active pane to full screen while working, then restore to check the other.

</details>
