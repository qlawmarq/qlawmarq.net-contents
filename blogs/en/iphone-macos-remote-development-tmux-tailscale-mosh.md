---
title: "Remote Development from iPhone to macOS in the AI Era: Tailscale + Shellfish/Blink + Mosh + tmux"
description: "A comprehensive guide optimized for Claude Code and AI tools to securely access macOS from your iPhone while on the go"
tags: ["Software Development", "AI", "Remote Development", "Claude Code"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-11-11T12:00:00.000Z"
---

In the AI era, software development is no longer confined to powerful laptops or multi-monitor setups.
With tools like Claude Code and Codex CLI, developers can now build and iterate through natural conversations with AI — reducing the need for heavy local environments.

This shift also changes how and where we work.
Instead of always coding at a desk, we can now connect to our development machines remotely — even from an iPhone on the go.

In this guide, I'll explore how to set up a lightweight, secure, and persistent remote development environment using Tailscale, Shellfish/Blink Shell, Mosh, and tmux — enabling you to code anywhere, seamlessly switching between your Mac and iPhone without losing your workflow.

## Overview

This guide shows how to achieve the following by combining Tailscale and Mosh:

- Secure access to your home Mac from anywhere without port forwarding
- Persistent connections when switching between Wi-Fi and LTE
- Continuous work sessions that survive disconnections and allow seamless resumption

## Requirements

### macOS Side

- [Homebrew](https://brew.sh/) - Package manager for macOS
- The following tools (install with Homebrew):
  - [Tailscale](https://tailscale.com/) - VPN
  - [Mosh](https://mosh.org/) - Mobile shell
  - [tmux](https://github.com/tmux/tmux/wiki) - Terminal multiplexer

### iPhone/iOS Device

- App Store access
- The following apps (install from App Store):
  - [Tailscale](https://tailscale.com/) - VPN
  - SSH/Mosh client (choose from recommended apps below)

### Tools Used

| Tool                  | Role               | Problem It Solves                                                                |
| --------------------- | ------------------ | -------------------------------------------------------------------------------- |
| **Tailscale**         | Peer-to-peer VPN   | Secure remote access without port forwarding                                     |
| **Shellfish / Blink** | SSH/Mosh client    | iPhone GUI operation, tap-to-connect, built-in Mosh support                      |
| **Mosh**              | Mobile shell       | Connection persistence during network switches, responsive input despite latency |
| **tmux**              | Session management | Work persistence across disconnections, multi-device screen sharing              |

#### Connection Architecture

```markdown
iPhone (Shellfish or Blink Shell)
↓
Tailscale VPN (peer-to-peer connection)
↓
macOS (development machine)
├─ SSH or Mosh server
└─ tmux session (persistent, multi-device sharing)
```

Each layer works together to provide stable remote development in mobile environments:

| Layer        | Technology        | Role                                       |
| ------------ | ----------------- | ------------------------------------------ |
| **Client**   | Shellfish / Blink | Tap-to-connect, GUI management             |
| **Network**  | Tailscale         | Secure connection, NAT traversal, fixed IP |
| **Protocol** | Mosh              | Connection persistence, local echo         |
| **Session**  | tmux              | Session persistence, screen sharing        |

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

#### 1.2 tmux Configuration (Recommended)

Create a tmux configuration file optimized for Claude Code and AI tools.

Save the following to `~/.tmux.conf`:

```bash
# ====================================================================================
# Basic Settings - 2025 Recommended Configuration
# ====================================================================================

# Terminal Type: tmux-256color (modern recommendation, tmux 2.6+)
# Benefits over screen-256color:
#   - Italics support (screen doesn't support it)
#   - Better color rendering
#   - Improved CJK (Japanese/Chinese/Korean) character handling
set -g default-terminal "tmux-256color"

# True Color (24-bit RGB) Support
# Modern tmux 3.2+ recommended method
set -as terminal-features ",*:RGB"
set-option -sa terminal-overrides ",tmux*:Tc"

# Enable mouse support (essential for iOS touch operations)
set -g mouse on

# Start window/pane numbering from 1 (human-friendly)
set -g base-index 1
setw -g pane-base-index 1

# Renumber windows when one is closed (no gaps)
set -g renumber-windows on

# ====================================================================================
# Claude Code / AI Tools Optimization
# ====================================================================================

# Dramatically increase scrollback buffer
# Claude Code generates verbose output requiring large buffer
# Recommended: 500000 lines (~250MB)
# Reference: Default is 2000 lines, typical usage is 50000 lines
set -g history-limit 500000

# Minimize ESC key response time (vim/neovim comfort)
# tmux 3.5 default: 10ms
set -sg escape-time 0

# Enable focus events (vim/neovim integration)
set -g focus-events on

# Extend message display time (for reviewing Claude Code output)
set -g display-time 3000

# Extend pane number display time (better usability on small screens)
set -g display-panes-time 4000

# ====================================================================================
# Key Bindings
# ====================================================================================

# vi mode key bindings (use vim keys in copy mode)
setw -g mode-keys vi

# Emacs-style status bar (matches zsh default)
set -g status-keys emacs

# ====================================================================================
# Visual Settings
# ====================================================================================

# Status bar at top (iOS keyboard won't cover it)
set -g status-position top

# Status bar color settings (simple and readable)
set -g status-style bg=black,fg=white

# Highlight current window
setw -g window-status-current-style bg=blue,fg=white,bold
```

**Benefits of this configuration**:

- **Claude Code Ready**[^6]: 500000-line scrollback to fully review long outputs
- **True Color**[^5]: Modern color schemes display correctly (using tmux-256color)
- **CJK Character Optimized**: Better Japanese/Chinese/Korean text quality
- **iOS Optimized**: Touch operations and keyboard position considered

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

#### 2.1 SSH Client Selection

Multiple SSH clients are available for iPhone/iPad. Here are the recommendations as of November 2025.

##### **Recommended: Shellfish (Secure ShellFish)**[^1]

**Features**:

- ✅ **Excellent tmux Support**: Session thumbnail previews, continuously improved in 2025
- ✅ **Native Mosh Support**: High connection stability
- ✅ **Files App Integration**: Direct SSH server integration into iOS Files app
- ✅ **Handoff Support**: Session migration across devices (iPhone ↔ iPad ↔ Mac)
- ✅ **iCloud Keychain Sync**: Automatic server settings sync
- ✅ **One-time Purchase**: $29.99 for lifetime use (monthly $2.99, yearly $14.99 also available)

**Why Recommended**:

- Strong long-term cost-performance (one-time purchase)
- Excellent tmux integration, handles Claude Code's long outputs well
- Highly responsive developer support

##### **Alternative: Blink Shell**[^2]

**Features**:

- ✅ **Full Mosh Support**: Connection maintained even after device reboot
- ✅ **Open Source**: Continuous community improvements
- ✅ **Blink Code**: Browser-based VSCode integration
- ✅ **Advanced Customization**: High flexibility in themes, fonts, and layouts
- ✅ **iPad Multitasking Optimized**: Excellent Split Screen / Slide Over support

**Price**: $19.99/year (no one-time purchase option)

**Why Recommended**:

- Highly rated as "Most complete SSH implementation on iOS"
- Ideal for those who want to use VSCode on mobile

#### 2.2 Install Required Apps

Install from the App Store:

1. **Tailscale** - VPN connection
2. **Shellfish** or **Blink Shell** - SSH/Mosh client (see recommendations above)

#### 2.3 Tailscale Setup

1. Launch the Tailscale app
2. Log in with the same account used on macOS
3. Once connected, you'll join the same VPN network as your Mac

#### 2.4 SSH Client Setup

Configure SSH connection in your chosen SSH client (Shellfish or Blink Shell).

##### **For Shellfish**

1. Launch Shellfish
2. Tap the **+** button to add a new host
3. Enter the following:
   - **Label**: `mac` (any recognizable name)
   - **Hostname**: `<macOS Tailscale IP>` (e.g., `100.64.1.2`)
     - This is the IP from step 1.4 that you saved
   - **User**: `<macOS username>`
     - Confirm with `whoami` command on macOS
   - **Port**: `22` (SSH default)
4. Tap **Save**

To use Mosh:

1. Tap the saved host and select **Edit**
2. In the **Advanced** section, enable **Use Mosh**
3. Tap **Save**

##### **For Blink Shell**

1. Launch Blink Shell
2. Type `config` command to add a host
3. Tap **+** in the **Hosts** tab
4. Enter the following:
   - **Host**: `mac` (any host name)
   - **HostName**: `<macOS Tailscale IP>` (e.g., `100.64.1.2`)
   - **User**: `<macOS username>`
   - **Port**: `22`
5. Tap **Save**

To use Mosh:

- Blink Shell has native Mosh support, so simply type `mosh mac` to connect via Mosh

Setup complete. Connect by tapping the host name.

### 3. Connection Testing

#### 3.1 SSH Connection Test

Test the macOS connection using your SSH client.

**Connection Steps:**

1. Open your SSH client (Shellfish or Blink Shell)
2. Tap the `mac` host you created
3. On first connection, tap **Continue** or **Trust** in the host key dialog
4. Enter your macOS user password if prompted
5. You'll see the macOS terminal upon successful connection

**Verification:**
Confirm the connection with:

```bash
hostname
```

If the macOS hostname appears, the connection succeeded.

#### 3.2 Mosh Connection Benefits

Mosh is better suited for mobile environments than SSH. If you've already configured it (see step 2.4), you can experience these benefits:

**Mosh Benefits:**

- 📱 **Connection Stability**: Connection maintained even after sleeping iPhone for minutes
- 🔄 **Network Switching**: Connection continues when switching from Wi-Fi to LTE
- ⚡ **Local Echo**: Instant local character echo (responsive even with latency)
- 🔌 **Device Reboot**: With Blink Shell, connection maintained even after device reboot (full Mosh implementation)

**When to Use Mosh vs SSH:**

- **Mosh Recommended**: While traveling, unstable connections, long sessions
- **SSH is OK**: Stable home Wi-Fi, short tasks

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
   - iPhone: Tap the `mac` host in your SSH client (Shellfish/Blink Shell)
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

When your SSH client attempts Mosh connection, you may see:

```
Command executed with error: zsh:1: command not found: mosh-server
No response from Mosh server
```

This occurs because `.zprofile` or `.zshrc` aren't loaded in SSH non-interactive shells, so Homebrew's PATH isn't available.

**Solution**:

1. Create `~/.zshenv` and add Homebrew's PATH:

   ```bash
   # Homebrew
   eval "$(/opt/homebrew/bin/brew shellenv)"
   ```

2. Verify the configuration:

   ```bash
   zsh -c 'which mosh-server'
   ```

   Success if you see `/opt/homebrew/bin/mosh-server`.

3. Retry Mosh connection from your SSH client.

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
Whether you're debugging during your commute, monitoring builds from a café, or quickly deploying fixes from your iPhone, this setup lets you stay in flow — the way modern development was meant to be.

---

**Disclaimer**: The information in this article is current as of November 2025. Software updates and specification changes may make some content outdated. Please refer to each tool's official documentation for the latest information.

---

[^1]: **Shellfish (Secure ShellFish)**: [App Store](https://apps.apple.com/app/ssh-client-secure-shellfish/id1336634154) | [Official Site](https://secureshellfish.app/) | MacStories "Secure ShellFish Review" (2019) | 2025 Update: DECSLRM support for improved tmux compatibility

[^2]: **Blink Shell**: [App Store](https://apps.apple.com/app/blink-shell-build-code/id1594898306) | [Official Site](https://blink.sh/) | [GitHub](https://github.com/blinksh/blink) | Open source, top developer tool on AppStore for 5+ years

[^3]: [GitHub Issue: google-gemini/gemini-cli #10349](https://github.com/google-gemini/gemini-cli/issues/10349) - "Automatic scrolling to input on iOS Termius prevents viewing previous content" (October 2025)

[^4]: [Termius - iOS Background Limitations](https://support.termius.com/hc/en-us/articles/900006226306)

[^5]: **Terminal Type Configuration**: The tmux-256color vs screen-256color debate is widely discussed in the community. Multiple forums including [Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/1045/getting-256-colors-to-work-in-tmux) and [Stack Overflow](https://stackoverflow.com/questions/10158508/lose-vim-colorscheme-in-tmux) recommend tmux-256color for modern terminals.

[^6]: [Brian P. Hogan "Working with Claude Code"](https://bphogan.com/2025/06/19/2025-06-19-claude-code-tips/) (June 2025) - Recommendation: "big scrollback buffer" for Claude Code | Related: [pchalasani/claude-code-tools](https://github.com/pchalasani/claude-code-tools) - Claude Code + tmux integration tools | [ooloth/dotfiles](https://github.com/ooloth/dotfiles) - Practical dotfiles for Claude Code

[^7]: **tmux Official**: [tmux Wiki](https://github.com/tmux/tmux/wiki) | [tmux 3.5 Release Notes](https://github.com/tmux/tmux/blob/master/CHANGES) (October 2024) | [tmux FAQ](https://github.com/tmux/tmux/wiki/FAQ)

[^8]: **Mosh Official**: [Official Site](https://mosh.org/) | [GitHub Repository](https://github.com/mobile-shell/mosh) | [GitHub Issue #122](https://github.com/mobile-shell/mosh/issues/122) - Scrollback limitations (Recommended solution: Use with tmux)

[^9]: **Tailscale Official**: [Official Site](https://tailscale.com/) | [Security Best Practices](https://tailscale.com/kb/) | [Admin Console](https://login.tailscale.com/admin)
