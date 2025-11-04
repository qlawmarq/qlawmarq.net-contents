---
title: "Remote Development from iPhone to macOS in the AI Era: Tailscale + Termius + Mosh + tmux"
description: "A comprehensive guide to securely accessing macOS from your iPhone while on the go and enabling comfortable development."
tags: ["Software Development"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-11-01T12:00:00.000Z"
---

In the AI era, software development is no longer confined to powerful laptops or multi-monitor setups.
With tools like Claude Code and Codex CLI, developers can now build and iterate through natural conversations with AI — reducing the need for heavy local environments.

This shift also changes how and where we work.
Instead of always coding at a desk, we can now connect to our development machines remotely — even from an iPhone on the go.

In this guide, I’ll explore how to set up a lightweight, secure, and persistent remote development environment using Tailscale, Termius, Mosh, and tmux — enabling you to code anywhere, seamlessly switching between your Mac and iPhone without losing your workflow.

## Overview

This guide shows how to achieve the following by combining Tailscale and Mosh:

- Secure access to your home Mac from anywhere without port forwarding
- Persistent connections when switching between Wi-Fi and LTE
- Continuous work sessions that survive disconnections and allow seamless resumption

## Requirements

### macOS Side

- [Homebrew (package manager for macOS)](https://brew.sh/)
  - [Tailscale](https://tailscale.com/)
  - [Mosh](https://mosh.org/)
  - [tmux](https://github.com/tmux/tmux/wiki)

### iPhone/iOS Device

- App Store access
  - [Tailscale](https://tailscale.com/)
  - [Termius](https://termius.com/)

### Tools Used

| Tool          | Role               | Problem It Solves                                                                |
| ------------- | ------------------ | -------------------------------------------------------------------------------- |
| **Tailscale** | Peer-to-peer VPN   | Secure remote access without port forwarding                                     |
| **Termius**   | SSH/Mosh client    | iPhone GUI operation, tap-to-connect, built-in Mosh support                      |
| **Mosh**      | Mobile shell       | Connection persistence during network switches, responsive input despite latency |
| **tmux**      | Session management | Work persistence across disconnections, multi-device screen sharing              |

#### Connection Architecture

```markdown
iPhone (Termius)
↓
Tailscale VPN (peer-to-peer connection)
↓
macOS (development machine)
├─ SSH or Mosh server (selected in Termius)
└─ tmux session (persistent, multi-device sharing)
```

Each layer works together to provide stable remote development in mobile environments:

| Layer        | Technology | Role                                       |
| ------------ | ---------- | ------------------------------------------ |
| **Client**   | Termius    | Tap-to-connect, GUI management             |
| **Network**  | Tailscale  | Secure connection, NAT traversal, fixed IP |
| **Protocol** | Mosh       | Connection persistence, local echo         |
| **Session**  | tmux       | Session persistence, screen sharing        |

## Setup Instructions

### 1. macOS Configuration

#### 1.1 Installing Required Packages

Install the necessary tools using Homebrew:

```bash
# Install Tailscale (VPN)
brew install --cask tailscale-app

# Install Mosh (mobile shell)
brew install mosh

# Install tmux (terminal multiplexer)
brew install tmux
```

#### 1.2 tmux Configuration (Optional)

Create a tmux configuration file. Save the following to `~/.tmux.conf`:

```bash
# Enable mouse support (allows pane selection via touch)
set -g mouse on

# vi mode key bindings
setw -g mode-keys vi

# Automatically renumber windows
set -g renumber-windows on

# Eliminate ESC input delay (improves vim usage)
set -s escape-time 0

# Status bar color settings
set -g status-style bg=black,fg=white

# Enable 256-color terminal
set -g default-terminal "screen-256color"
```

#### 1.3 Enable Remote Login

Enable macOS remote login (SSH server):

1. Open System Settings
2. Go to **General** > **Sharing**
3. Enable **Remote Login**
4. Select authorized users (recommended: administrators only)

#### 1.4 Tailscale Setup

```bash
# Launch Tailscale app
open -a Tailscale
```

1. When Tailscale launches, follow the login instructions
2. Authenticate with an SSO provider (Google, GitHub, etc.)
3. Once connected, note the **Tailscale IP address assigned to macOS**:

```bash
tailscale ip -4
```

**Example output**: `100.64.1.2`

**Important**: This is your macOS Tailscale IP address. **Save this IP** for use in later steps.

**Note**: Tailscale automatically assigns fixed IPs in the `100.x.y.z` format. This IP remains constant while the device is part of the Tailscale network.

#### 1.5 Enable Tailscale 2FA (Recommended)

1. Access the [Tailscale admin console](https://login.tailscale.com/admin)
2. Navigate to **Settings** > **Users** and select your account
3. Enable **Two-factor authentication**

### 2. iPhone Configuration

#### 2.1 Install Required Apps

Install from the App Store:

1. **Tailscale** - VPN connection
2. **Termius** - SSH/Mosh client

#### 2.2 Tailscale Setup

1. Launch the Tailscale app
2. Log in with the same account used on macOS
3. Once connected, you'll join the same VPN network as your Mac

#### 2.3 Termius Setup

Termius is GUI-based, so all configuration happens through the interface.

**Adding a Host:**

1. Launch Termius
2. Tap the **Hosts** tab at the bottom
3. Tap the **+** button (top right) to add a new host
4. In the **New Host** screen, enter:
   - **Alias**: `mac` (choose any recognizable name)
   - **Hostname**: `<macOS Tailscale IP>`
     - Example: `100.64.1.2`
     - **This is the IP from step 1.4 that you saved**
   - **Username**: `<macOS username>`
     - Confirm with `whoami` command on macOS
   - **Port**: `22` (SSH default, no change needed)
   - **Protocol**: `SSH` (can change to Mosh later)
5. Tap **Save** (top right)

The `mac` host now appears in your list. Connect with a single tap.

### 3. Connection Testing

#### 3.1 SSH Connection Test

Test the macOS connection using Termius.

**Connection Steps:**

1. Open Termius
2. Tap `mac` from the **Hosts** tab
3. On first connection, tap **Continue** or **Trust** in the host key dialog
4. Enter your macOS user password if prompted
5. You'll see the macOS terminal upon successful connection

**Verification:**
Confirm the connection with:

```bash
hostname
```

If the macOS hostname appears, the connection succeeded.

#### 3.2 Mosh Connection Setup

Mosh is better suited for mobile environments than SSH. Skip this if regular SSH works well for you.

**Configuring Mosh in Termius:**

Termius has built-in Mosh support with easy GUI configuration.

1. Open the **Hosts** tab in Termius
2. **Long press** the `mac` host and select **Edit**
3. In the **Connection** section:
   - **Protocol**: Select `Mosh` (default is `SSH`)
4. Tap **Save**

**First Connection Behavior:**

1. Tap the `mac` host to connect
2. Termius automatically connects via SSH
3. Authenticate with password or SSH key
4. mosh-server installs automatically (macOS side, no root required)
5. Connection switches to UDP (ports 60000-61000)
6. Connection complete

**Subsequent Connections:**

Simply tap the `mac` host for automatic Mosh connection.

**Testing Mosh Benefits:**

After connecting, test these Mosh features:

- Sleep iPhone for a few minutes, then wake → connection maintained
- Switch from Wi-Fi to LTE → connection continues
- Type characters → instant local echo (responsive even with latency)

#### 3.3 Starting a tmux Session

Once connected to macOS, start or join a tmux session:

```bash
# Create a new session
tmux new -s dev

# Or join an existing session
tmux attach -t dev
```

## Basic Usage

### tmux Basic Commands

#### Session Management

```bash
# Create a new session
tmux new -s <session_name>

# Join an existing session
tmux attach -t <session_name>

# List sessions
tmux ls

# Temporarily detach from session (session continues)
# Ctrl+B → d

# End session
exit
```

### Sharing Sessions Between iPhone and Mac

tmux allows you to share the same terminal session across devices.

1. **Start a session (on either device):**

   Connect to macOS and create a tmux session:

   ```bash
   tmux new -s dev
   ```

2. **Join from the other device:**
   - iPhone: Tap the `mac` host in Termius
   - Mac: Use `ssh <Tailscale IP>` or `mosh <Tailscale IP>` in Terminal

   After connecting, join the session:

   ```bash
   tmux attach -t dev
   ```

3. **Synchronized screens:**

   Any input from either device appears immediately on both screens. This enables starting work on iPhone and continuing on Mac, or vice versa.

### Useful tmux Features

The configuration above enables:

- **Mouse support**: Touch-based pane selection
- **vi key bindings**: vim keys in scroll mode
- **Automatic renumbering**: Window numbers compress when closed
- **No ESC delay**: Eliminates vim usage delays

Customize by adding configurations to `~/.tmux.conf` as needed.

## Troubleshooting

### Tailscale Can't Find macOS

**Cause**: Different accounts on devices, or connection not established

**Solution**:

1. Verify both devices use the same Tailscale account
2. Check macOS Tailscale connection:
   ```bash
   tailscale status
   ```
3. Verify macOS remote login is enabled
4. Check iPhone Tailscale app connection status

### Can't Connect with Mosh (`command not found: mosh-server`)

**Cause**: PATH not properly set for SSH-executed commands

When Termius attempts Mosh connection, you may see:

```
Command executed with error: zsh:1: command not found: mosh-server
No response from Mosh server
```

This occurs because `.zprofile` or `.zshrc` aren't loaded in SSH non-interactive shells, so Homebrew's PATH isn't available.

**Solution**:

1. Manual configuration:

   Create `~/.zshenv` and add Homebrew's PATH:

   ```bash
   # Homebrew
   eval "$(/opt/homebrew/bin/brew shellenv)"
   ```

2. Verify the configuration:

   ```bash
   zsh -c 'which mosh-server'
   ```

   Success if you see `/opt/homebrew/bin/mosh-server`.

3. Retry Mosh connection from Termius.

### Other Mosh Connection Issues

**Cause**: Mosh not installed, or UDP ports blocked by firewall

**Solution**:

1. Verify Mosh installation on macOS:
   ```bash
   which mosh-server
   ```
2. Check firewall settings (rarely an issue with Tailscale):
   - System Settings > Network > Firewall
3. Verify SSH connection works (Mosh requires SSH for initial connection)

### SSH "Permission denied" Error

**Cause**: Remote login disabled, or user permission issues

**Solution**:

1. Verify macOS remote login is enabled:
   - System Settings > General > Sharing > Remote Login
2. Verify correct username:
   ```bash
   # Check current username on macOS
   whoami
   ```

### tmux Session Not Found

**Cause**: Session not created, or incorrect session name

**Solution**:

1. Check existing sessions:
   ```bash
   tmux ls
   ```
2. Create new session if none exist:
   ```bash
   tmux new -s dev
   ```

## Advanced: VS Code on Browser

### code-server Setup

To use VS Code in iPhone Safari:

```bash
# Install code-server on macOS
brew install code-server

# Start code-server
code-server

# Access from iPhone Safari:
# http://<Tailscale IP>:8080
```

Find the password in `~/.config/code-server/config.yaml`.

---

## Summary

You’ve now built a remote development environment that embodies the AI era of coding — mobile, connected, and resilient.

This setup isn’t just about convenience; it represents a fundamental shift in how developers work.
By combining Tailscale’s secure networking, Mosh’s persistent connections, and tmux’s session sharing, you can collaborate with AI tools or teammates from anywhere — all through your pocket-sized device.

AI-assisted coding and mobile-first workflows are redefining productivity.
Whether you’re debugging during your commute, monitoring builds from a café, or quickly deploying fixes from your iPhone, this setup lets you stay in flow — the way modern development was meant to be.
