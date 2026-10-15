# Ubuntu

Dev environment setup inside the Ubuntu VM.

## Variables

Set these before running the commands below:

```sh
GIT_USER_NAME="<name>"
GIT_USER_EMAIL="<email>"
GITHUB_USERNAME="<github-username>"
```

## User

Passwordless sudo:

```sh
echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
```

## Essentials

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y qemu-guest-agent wget curl gnupg2 build-essential jq git unzip zip mosh
```

## SSH Access

Import your public keys from GitHub and disable password authentication:

```sh
ssh-import-id gh:$GITHUB_USERNAME
printf "PasswordAuthentication no\nKbdInteractiveAuthentication no\n" | sudo tee /etc/ssh/sshd_config.d/01-disable-password.conf
sudo systemctl restart ssh
```

## tmux

Session persistence across SSH disconnects — if your connection drops, `tmux attach` picks up where you left off. Also useful for running multiple tasks in parallel (e.g. an AI agent in one pane, tests in another).

```sh
sudo apt install -y tmux
```

Keep panes alive when a process inside runs out of memory. Ubuntu's tmux runs each pane in its own systemd scope, and by default systemd stops the whole scope after an OOM kill — the shell dies with it, and if it was the last pane, the session is gone. This makes systemd kill only the offending process:

```sh
mkdir -p ~/.config/systemd/user/tmux-spawn-.scope.d
printf "[Scope]\nOOMPolicy=continue\n" > ~/.config/systemd/user/tmux-spawn-.scope.d/oom.conf
systemctl --user daemon-reload
```

<details>
<summary>Configuration and usage</summary>

```sh
cat << 'EOF' > ~/.tmux.conf
# General settings (good defaults)
set -g default-terminal "tmux-256color"
set -sg escape-time 50
set -g mouse on
setw -g mode-keys vi

# Prefix: C-a instead of C-b
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Window/pane numbering from 1 (matches keyboard layout)
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
run '~/.tmux/plugins/tpm/tpm'
EOF
```

Install [TPM](https://github.com/tmux-plugins/tpm) and fetch plugins:

```sh
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
~/.tmux/plugins/tpm/bin/install_plugins
```

**Usage:**

```sh
tmux                   # start new session
tmux new -s work       # start named session
tmux ls                # list sessions
tmux a -t work         # attach to session
tmux kill-ses -t work  # end session
```

Inside tmux: `prefix` is `Ctrl+a`.

| Action | Keys |
|---|---|
| Detach (session keeps running) | `prefix d` |
| Rename session | `prefix $` |
| Kill session | `prefix :` then `kill-ses` |
| Choose session / last session | `prefix s` / `prefix L` |
| Vertical split | `prefix %` |
| Horizontal split | `prefix "` |
| Navigate panes | `prefix arrow` |
| Kill pane | `prefix x` |
| Maximize/restore pane | `prefix z` |
| New window | `prefix c` |
| Next/previous window | `prefix n` / `prefix p` |
| Choose window / last window | `prefix w` / `prefix l` or `prefix a` |
| Copy mode (scroll, search, copy) | `prefix [` then vi keys, `q` to exit |
| Paste / choose from copies | `prefix ]` / `prefix =` |
| Reload config | `prefix R` |
| Save/restore session | `prefix Ctrl-s` / `prefix Ctrl-r` |

</details>

## Git Config

```sh
git config --global user.name "$GIT_USER_NAME"
git config --global user.email "$GIT_USER_EMAIL"
git config --global core.autocrlf input
git config --global pull.ff only
```

## Source Control

Authenticate with scoped tokens that only allow pushing and pulling code. Create PRs from the browser. Pair with protected branches to prevent direct pushes to `main`.

### GitHub CLI

```sh
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install -y gh
gh auth login
```

<details>
<summary>Personal Access Token</summary>

Create a [fine-grained personal access token](https://github.com/settings/personal-access-tokens) with minimal permissions:

- **Contents** — read/write (code and branches)
- **Metadata** — read-only (always included)

During `gh auth login`, select **GitHub.com** → **HTTPS** → **Paste an authentication token**.

</details>

### GitLab CLI

```sh
GLAB_VERSION=$(curl -sL https://gitlab.com/api/v4/projects/gitlab-org%2Fcli/releases/permalink/latest | jq -r '.tag_name' | tr -d 'v')
wget "https://gitlab.com/gitlab-org/cli/-/releases/v${GLAB_VERSION}/downloads/glab_${GLAB_VERSION}_linux_amd64.deb"
sudo dpkg -i glab_${GLAB_VERSION}_linux_amd64.deb
rm glab_${GLAB_VERSION}_linux_amd64.deb
glab auth login
```

<details>
<summary>Personal Access Token</summary>

Create a [personal access token](https://gitlab.com/-/user_settings/personal_access_tokens) with minimal scopes:

- **read_user** (authenticates `glab` — without this, you will be prompted for credentials)
- **write_repository** (clone, push, pull, branches)

The `api` scope is intentionally excluded — GitLab requires it for creating merge requests, but it grants full API access. Create merge requests from the browser instead.

During `glab auth login`, select **GitLab.com** → **Token** and paste the token.

</details>

## Docker

```sh
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker
```

## Node (via nvm)

```sh
NVM_VERSION=$(curl -s https://api.github.com/repos/nvm-sh/nvm/releases/latest | jq -r '.tag_name')
curl -o- "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_VERSION/install.sh" | bash
source ~/.bashrc
nvm install --lts
```

## Java (via SDKMAN)

```sh
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java
```

<details>
<summary>Picking a specific version</summary>

The default installs SDKMAN's recommended JDK. To pick a specific version:

```sh
sdk list java                    # browse available versions
sdk install java <identifier>    # e.g. 21-tem for Temurin 21
```

</details>

## Claude Code

AI coding agent from Anthropic. Requires a [subscription](https://claude.com/pricing) or [API key](https://console.anthropic.com/settings/keys).

```sh
curl -fsSL https://claude.ai/install.sh | bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
echo 'alias claude="claude --dangerously-skip-permissions"' >> ~/.bashrc
source ~/.bashrc
```

Run inside a project directory, ideally in a tmux session so it survives SSH disconnects:

```sh
claude
```
