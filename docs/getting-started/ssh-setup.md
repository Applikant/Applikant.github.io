# SSH Setup

How to configure SSH access for Applikant using `as` as a forced command.

## Overview

Applikant uses SSH **forced commands** to control git access. Instead of giving users shell access, every SSH key is configured to run `as` — which only allows git operations and checks authorization.

## Step-by-Step

### 1. Create a git user

```bash
sudo adduser --disabled-password --shell /bin/bash git
```

### 2. Build and install `as`

```bash
cd applikant.git-in/as
rebar3 escriptize
sudo cp _build/default/bin/as /home/git/as
sudo chmod +x /home/git/as
sudo chown git:git /home/git/as
```

### 3. Add SSH keys to `authorized_keys`

Edit `/home/git/.ssh/authorized_keys`:

```
command="/home/git/as user-42",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2E... alice@laptop
command="/home/git/as user-73",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2E... bob@workstation
```

Each line:

- **Runs `as`** with a user ID as argument (instead of a shell)
- **Disables** port forwarding, X11, agent forwarding and PTY allocation
- **Maps** the SSH key to a specific Applikant user ID

### 4. Create a bare repository

```bash
sudo -u git mkdir -p /home/git/repos/team/project.git
sudo -u git git init --bare /home/git/repos/team/project.git
```

### 5. Install git hooks

```bash
cd /home/git/repos/team/project.git/hooks
for hook in pre-receive update post-update post-receive; do
    sudo -u git ln -sf /home/git/ah $hook
done
```

### 6. Make sure `af` is running

```bash
cd applikant.git-in/af
rebar3 shell
```

### 7. Test

```bash
git clone ssh://git@yourserver/team/project.git
```

## How the Forced Command Works

When a user connects via SSH:

```
User: git clone ssh://git@server/team/project.git

SSH sets:  SSH_ORIGINAL_COMMAND=git-upload-pack 'team/project.git'
SSH runs:  /home/git/as user-42

as:
  1. Reads SSH_ORIGINAL_COMMAND
  2. Parses: git-upload-pack → read access, repo = team/project.git
  3. Connects to af, calls check_access("user-42", "team/project.git", read)
  4. If allowed → executes git-upload-pack
  5. If denied → prints error, exits with code 1
```

## Security

The `authorized_keys` options ensure:

| Option | Effect |
|---|---|
| `command="..."` | Only this command is executed, regardless of what the user requests |
| `no-port-forwarding` | No SSH tunnels |
| `no-X11-forwarding` | No X11 display forwarding |
| `no-agent-forwarding` | No SSH agent forwarding |
| `no-pty` | No interactive terminal |

!!! note
    Users **cannot** get a shell, run arbitrary commands, or create SSH tunnels — even if they try. The forced command overrides everything.

