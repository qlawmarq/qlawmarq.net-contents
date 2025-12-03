---
title: "iPhone to macOS Remote Dev with Tailscale + Shellfish"
description: "Build a secure remote development environment from iPhone to macOS with Tailscale, Shellfish, and tmux. Optimized for AI tools like Claude Code. Start coding anywhere."
tags: ["Software Development", "AI", "Remote Development", "Claude Code"]
publishedAt: "2025-11-01T12:00:00.000Z"
updatedAt: "2025-12-03T12:00:00.000Z"
---

With the proliferation of AI and LLMs, more tasks can now be accomplished through natural conversation alone.
Software development is no exception—thanks to tools like Claude Code and Codex CLI, we're moving toward a world where development happens simply through conversing with AI.

The need to work full-screen in a code editor on your PC is diminishing. While an execution environment is still necessary, remote development from a smartphone has become a viable option.
We've entered an era where you can develop from anywhere by remotely connecting from your smartphone to your home development machine—not just at your desk.

This article shows you how to build a remote access environment that lets you code from your iPhone on a train or update code while relaxing in the bath.

## Requirements

### Prerequisites

This article assumes you're using AI terminal tools like Claude Code or Gemini CLI.
Installation and basic usage of these tools aren't covered here—please refer to their official documentation.

### Devices

- macOS
  - [Tailscale](https://tailscale.com/) - VPN
  - [tmux](https://github.com/tmux/tmux/wiki) - Terminal multiplexer
- iPhone / iPad
  - [Tailscale](https://tailscale.com/) - VPN
  - [Secure ShellFish](https://secureshellfish.app/) - SSH client for iOS

### Connection Architecture

```markdown
iPhone (Shellfish)
↓
Tailscale VPN (Peer-to-peer connection)
↓
macOS (Development machine)
├─ SSH server
└─ tmux session (persistent, multi-device sharing)
```

Each layer works together to enable stable remote development, even in mobile environments:

| Layer        | Technology | Role                                          |
| ------------ | ---------- | --------------------------------------------- |
| **Client**   | Shellfish  | Tap to connect, GUI management                |
| **Network**  | Tailscale  | Secure connection, NAT traversal, fixed IP    |
| **Protocol** | SSH        | Encrypted secure connection                   |
| **Session**  | tmux       | Continues after disconnection, screen sharing |

## Setup Procedure

### 1. macOS Configuration

#### 1.1 Install Required Packages

Install the necessary tools on macOS using Homebrew:

```bash
# Install Tailscale (VPN)
brew install --cask tailscale-app

# Install tmux (terminal multiplexer)
brew install tmux
```

If you're not using Homebrew, download the installers from each official website.

#### 1.2 tmux Configuration (Recommended)

Create a tmux configuration file.

If you're already using tmux, you can keep your existing setup. However, the configuration below is optimized for AI terminal tools—consider using it as a reference.

**Benefits of this configuration**:

- **Claude Code compatible**[^5]: 500,000 lines of scrollback to fully view long outputs
- **True Color**[^4]: Latest color schemes display correctly (using tmux-256color)
- **CJK character optimization**: Improved display quality for Japanese characters
- **iOS optimization**: Design considering touch operations and keyboard position

Save the following content to `~/.tmux.conf`:

```bash
# ====================================================================================
# Basic Settings - 2025 Recommended Configuration
# ====================================================================================

# Terminal Type: tmux-256color (modern recommendation, tmux 2.6+)
set -g default-terminal "tmux-256color"

# Mouse support (for touch operations on iOS)
set -g mouse on

# Start window/pane numbering at 1 (human-friendly)
set -g base-index 1
setw -g pane-base-index 1

# Renumber windows when closing
set -g renumber-windows on

# ====================================================================================
# Claude Code / AI Tool Optimization
# ====================================================================================

# Significantly increase scrollback buffer
# Claude Code generates long outputs, requiring a large buffer
# Recommended: 500000 lines (~250MB)
# Reference: Default is 2000 lines, 50000 lines for typical use
set -g history-limit 500000

# Minimize ESC key response time (comfort when using vim/neovim)
# tmux 3.5 default: 10ms
set -sg escape-time 0

# Enable focus events (vim/neovim integration)
set -g focus-events on

# Extend message display time (for confirming Claude Code output)
set -g display-time 3000

# Extend pane number display time (improved operability on small screens)
set -g display-panes-time 4000

# ====================================================================================
# Key Bindings
# ====================================================================================

# vi mode key bindings (vim keys available in copy mode)
setw -g mode-keys vi

# Status bar uses Emacs style (matches zsh default)
set -g status-keys emacs

# ====================================================================================
# Visual Settings
# ====================================================================================

# Position status bar at top (not hidden by iOS keyboard)
set -g status-position top

# Status bar color settings (simple and readable)
set -g status-style bg=black,fg=white

# Highlight current window
setw -g window-status-current-style bg=blue,fg=white,bold
```

#### 1.3 Enable Remote Login

Enable Remote Login (SSH server) on macOS:

1. Open System Settings
2. Select **General** > **Sharing**
3. Turn on **Remote Login**
4. Select users to allow access (recommended: administrators only)

#### 1.4 Tailscale Setup

```bash
# Launch Tailscale app
open -a Tailscale
```

1. Once the Tailscale app launches, follow the instructions to log in
2. Authenticate with an SSO provider (Google, GitHub, etc.)
3. After connection is complete, **check the Tailscale IP address assigned to macOS**:

```bash
tailscale ip -4
```

**Example output**: `100.64.1.2`

**Important**: This is your macOS machine's Tailscale IP address (the connection destination). **Save this IP** as you'll need it in later steps.

**Note**: Tailscale automatically assigns each device a fixed IP in the format `100.x.y.z`. This IP remains constant as long as the device stays connected to your Tailscale network.

#### 1.5 Enable Tailscale 2FA (Recommended)

1. Access the [Tailscale admin console](https://login.tailscale.com/admin)
2. Select your account from **Settings** > **Users**
3. Enable **Two-factor authentication**

### 2. iPhone Configuration

#### 2.1 Choosing an SSH Client

Several SSH clients are available for iPhone/iPad. After testing multiple options, I found **Secure ShellFish** to be the most user-friendly. Here's an evaluation of each app as of November 2025.

##### **Recommended: Shellfish (Secure ShellFish)**[^1]

**Features**:

- **Excellent tmux support**: Session thumbnail previews, Handoff for session migration between devices
- **Background SSH maintenance**: Feature to maintain SSH connection even when app is in background
- **Files app integration**: Direct integration of SSH servers into iOS Files app
- **iCloud Keychain sync**: Automatic server configuration sync
- **One-time purchase available**: Permanent use for $29.99 (also available as monthly $2.99, annual $14.99, free version available)

**Why I recommend it**:

- Excellent tmux integration—handles Claude Code's long outputs flawlessly
- Great long-term value with one-time purchase option
- Smooth, polished user experience

##### **Alternative 1: Termius**[^2]

**Features**:

- **Cross-platform**: Syncs across Windows, macOS, Linux, iOS, Android
- **Mosh support**: Improved connection stability
- **SFTP integration**: Built-in file transfer functionality

**Notes**:

- **Compatibility issues with Claude Code**: As of November 2025, [scrollback issues have been reported in GitHub Issue #10349](https://github.com/google-gemini/gemini-cli/issues/10349) when using AI terminal tools like Gemini CLI. After long outputs, the app may automatically scroll to the input field, preventing you from viewing previous output. (I stopped using Termius due to this issue.)

**Pricing**: Free version available, Pro plan is $10/month (annual billing), $15/month (monthly billing)

##### **Alternative 2: Blink Shell**[^3]

**Features**:

- **Complete Mosh support**: Maintains connection during network switches, maintains connection after device restart
- **Open source**: Continuous improvement by the community
- **Blink Code**: Browser-based VSCode integration
- **Advanced customization**: High degree of freedom in themes, fonts, and layouts
- **iPad multitasking optimization**: Excellent support for Split Screen / Slide Over

**Pricing**: $19.99/year (no one-time purchase option)

#### 2.2 Install Required Apps

Install the following from the App Store:

1. **Tailscale** - For VPN connection
2. **Shellfish** - SSH client (refer to the recommendation above)

#### 2.3 Tailscale Setup

1. Launch the Tailscale app
2. Log in with the same account as macOS
3. Once connected, you'll join the same VPN network as macOS

#### 2.4 Shellfish Setup

Configure your SSH connection in Shellfish:

1. Launch the Shellfish app
2. Tap the **+** button to add a new host
3. Enter the following details:
   - **Label**: `mac` (or any name you prefer)
   - **Hostname**: Your macOS Tailscale IP (e.g., `100.64.1.2`)
     - This is the IP you saved in step 1.4
   - **User**: Your macOS username
     - Run `whoami` on macOS if you're unsure
   - **Port**: `22` (SSH default)
4. Tap **Save**

That's it! Tap the hostname to connect.

### 3. Connection Test

#### 3.1 Test SSH Connection

Let's test your connection to macOS through Shellfish:

**Steps:**

1. Open Shellfish
2. Tap the `mac` host you just created
3. On first connection, you'll see a host key confirmation—tap **Continue** or **Trust**
4. Enter your macOS password if prompted
5. Success! You should see your macOS terminal

**Verify the connection:**

```bash
hostname
```

If you see your macOS hostname, you're connected successfully.

#### 3.2 Start tmux Session

Now that you're connected, start a tmux session:

```bash
# Create a new session
tmux new -s dev

# Join an existing session
tmux attach -t dev
```

## Basic Usage

### Basic tmux Commands

#### Session Management

```bash
# Create a new session
tmux new -s <session-name>

# Join an existing session
tmux attach -t <session-name>

# Display session list
tmux ls

# Temporarily detach from session (session continues)
# Ctrl+B → d

# End session
exit
```

### Share the Same Session Between iPhone and Mac

With tmux, you can share the same terminal session across your iPhone and Mac.

1. **Start a session (from either device):**

   Connect to your macOS machine and create a tmux session:

   ```bash
   tmux new -s dev
   ```

2. **Join from your other device:**
   - **iPhone**: Tap the `mac` host in Shellfish
   - **Mac**: Run `ssh <Tailscale IP>` or `mosh <Tailscale IP>`

   Then join the session:

   ```bash
   tmux attach -t dev
   ```

3. **Synchronized in real-time:**

   Everything you type on one device instantly appears on the other.
   Start work on your iPhone, seamlessly continue on your Mac—or vice versa.

## Troubleshooting

### Cannot Find macOS in Tailscale

**Possible causes**: Different Tailscale accounts, or connection issues

**Solutions**:

1. Confirm both devices use the same Tailscale account
2. Check macOS connection status:
   ```bash
   tailscale status
   ```
3. Verify Remote Login is enabled on macOS
4. Check Tailscale connection status on iPhone

### SSH "Permission denied" Error

**Possible causes**: Remote Login disabled, or wrong username

**Solutions**:

1. Enable Remote Login on macOS:
   - System Settings → General → Sharing → Remote Login
2. Verify your username:
   ```bash
   whoami  # Run this on macOS
   ```

### tmux Session Not Found

**Possible causes**: Session doesn't exist yet, or wrong name

**Solutions**:

1. List existing sessions:
   ```bash
   tmux ls
   ```
2. Create the session if needed:
   ```bash
   tmux new -s dev
   ```

## Advanced: VS Code in Your Browser

### code-server Setup

Want to use VS Code on your iPhone through Safari?

```bash
# Install code-server on macOS
brew install code-server

# Start the server
code-server

# On iPhone, open Safari and navigate to:
# http://<Tailscale IP>:8080
```

Find the password in `~/.config/code-server/config.yaml`.

---

## Summary

You now have a secure remote development environment that lets you code on macOS from your iPhone.

This setup embodies a development style built for the AI era—mobile, always-connected, and seamless.
By combining Tailscale's secure VPN, Shellfish's persistent SSH connections, and tmux's session sharing, you can collaborate with AI tools from anywhere.

AI-assisted development and mobile-first workflows unlock new levels of productivity.
Debug during your commute, run tests at a cafe, deploy instantly from anywhere—you can maintain an uninterrupted flow in every scenario.

Remember to enable security features like Tailscale 2FA, and enjoy your newfound mobile development freedom.

---

**Disclaimer**: The information in this article is current as of November 2025. Content may become outdated due to software version updates or specification changes. Please check the official documentation of each tool for the latest information.

---

[^1]: **Shellfish (Secure ShellFish)**: [App Store](https://apps.apple.com/app/ssh-client-secure-shellfish/id1336634154) | [Official Site](https://secureshellfish.app/) | MacStories "Secure ShellFish Review" (2019) | 2025 Update: tmux improvements with DECSLRM support

[^2]: **Termius**: [App Store](https://apps.apple.com/app/termius-terminal-ssh-client/id549039908) | [Official Site](https://termius.com/) | Note as of November 2025: Scrollback issues when using AI terminal tools - [GitHub Issue: google-gemini/gemini-cli #10349](https://github.com/google-gemini/gemini-cli/issues/10349)

[^3]: **Blink Shell**: [App Store](https://apps.apple.com/app/blink-shell-build-code/id1594898306) | [Official Site](https://blink.sh/) | [GitHub](https://github.com/blinksh/blink) | Open source, top developer tool on AppStore for over 5 years

[^4]: **Terminal Type Settings**: The tmux-256color vs screen-256color debate is widely discussed in the community. tmux-256color is recommended in multiple forums including [Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/1045/getting-256-colors-to-work-in-tmux), [Stack Overflow](https://stackoverflow.com/questions/10158508/lose-vim-colorscheme-in-tmux), and others.

[^5]: [Brian P. Hogan "Working with Claude Code"](https://bphogan.com/2025/06/19/2025-06-19-claude-code-tips/) (June 2025) - Recommended: "big scrollback buffer" for Claude Code | Related: [pchalasani/claude-code-tools](https://github.com/pchalasani/claude-code-tools) - Claude Code + tmux integration tool | [ooloth/dotfiles](https://github.com/ooloth/dotfiles) - Practical dotfiles compatible with Claude Code

[^6]: **tmux Official**: [tmux Wiki](https://github.com/tmux/tmux/wiki) | [tmux 3.5 Release Notes](https://github.com/tmux/tmux/blob/master/CHANGES) (September 27, 2024, 3.5a October 5, 2024) | [tmux FAQ](https://github.com/tmux/tmux/wiki/FAQ) | **Note**: Latest version is tmux 3.6

[^7]: **Mosh Official**: [Official Site](https://mosh.org/) | [GitHub Repository](https://github.com/mobile-shell/mosh) | Reference information for using Mosh with Blink Shell and others

[^8]: **Tailscale Official**: [Official Site](https://tailscale.com/) | [Security Best Practices](https://tailscale.com/kb/) | [Admin Console](https://login.tailscale.com/admin)
