# 05 — Network Hardening

VPS lockdown, Tailscale integration, firewall rules, and reverse proxy configuration for Path B (cloud VPS) deployments.

If you are running OpenClaw on a spare computer or Raspberry Pi on a home network, most of this module is optional — though sections 1 and 2 are worth reviewing regardless of deployment path.

---

## Contents

1. [Binding the gateway to loopback](#1-binding-the-gateway-to-loopback)
2. [SSH hardening and key-only authentication](#2-ssh-hardening-and-key-only-authentication)
3. [Tailscale for zero-trust access](#3-tailscale-for-zero-trust-access)
4. [UFW firewall rules](#4-ufw-firewall-rules)
5. [Reverse proxy with HTTPS](#5-reverse-proxy-with-https)
6. [Monitoring and alerting](#6-monitoring-and-alerting)

---

## 1. Binding the gateway to loopback

The single most important hardening step. By default, the OpenClaw gateway binds to `0.0.0.0:18789`, meaning it is reachable from any network interface — including the public internet if you are on a VPS. CVE-2026-25253 exploited this directly.

Check your current bind address:

```bash
openclaw gateway status
```

The output should show `127.0.0.1:18789`. If it shows `0.0.0.0:18789`, fix it immediately.

Edit `~/.openclaw/openclaw.json`:

```json
{
  "gateway": {
    "bind": "127.0.0.1",
    "port": 18789
  }
}
```

Restart and verify:

```bash
openclaw gateway restart
openclaw gateway status
# Should show: Gateway listening on 127.0.0.1:18789
```

With the gateway bound to loopback, it is only reachable from processes on the same machine. Remote access requires either an SSH tunnel or Tailscale (see sections 2 and 3).

---

## 2. SSH hardening and key-only authentication

### 2.1 Generate an SSH key pair (if you do not have one)

Run this on your local machine, not the VPS:

```bash
ssh-keygen -t ed25519 -C "openclaw-vps"
```

Copy the public key to your VPS:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-vps-ip
```

Verify you can log in with the key before disabling password auth:

```bash
ssh -i ~/.ssh/id_ed25519 user@your-vps-ip
```

### 2.2 Disable password authentication

On the VPS, edit `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

Test from a new terminal session before closing your existing one.

### 2.3 Change the default SSH port (optional but recommended)

Change `Port 22` to a non-standard port (e.g. 2222) in `/etc/ssh/sshd_config`. This reduces automated scanning noise. Update your firewall rules to allow the new port (see section 4).

### 2.4 Access the OpenClaw dashboard over SSH tunnel

With the gateway bound to loopback, use an SSH tunnel to access the dashboard from your local browser:

```bash
ssh -L 18789:127.0.0.1:18789 user@your-vps-ip -N
```

Then open `http://127.0.0.1:18789` in your local browser. The connection is encrypted end-to-end through the SSH tunnel.

Add to your `~/.ssh/config` for convenience:

```
Host openclaw-vps
    HostName your-vps-ip
    User your-user
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 18789 127.0.0.1:18789
```

---

## 3. Tailscale for zero-trust access

Tailscale creates a private WireGuard mesh network between your devices. It is a more ergonomic alternative to SSH tunnels for ongoing access and works well for multi-device setups.

### 3.1 Install Tailscale on the VPS

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Follow the authentication link in the output to add the VPS to your Tailscale network.

### 3.2 Install Tailscale on your local machine

Download from tailscale.com for macOS, Windows, or Linux. Authenticate with the same account.

### 3.3 Bind OpenClaw to the Tailscale interface

Once Tailscale is running, your VPS has a stable Tailscale IP (100.x.y.z). You can bind OpenClaw to this IP instead of loopback to make it reachable from your other Tailscale devices — but not from the public internet.

```bash
# Get your Tailscale IP
tailscale ip -4
```

```json
{
  "gateway": {
    "bind": "100.x.y.z",
    "port": 18789
  }
}
```

Restart the gateway. You can now access the dashboard at `http://100.x.y.z:18789` from any device on your Tailscale network, with all traffic encrypted by WireGuard.

### 3.4 Tailscale ACLs

Use Tailscale ACLs (in the admin console at login.tailscale.com) to restrict which devices can reach port 18789 on the VPS. This is especially useful if you have multiple devices on your Tailnet and want to limit OpenClaw access to specific machines.

---

## 4. UFW firewall rules

UFW (Uncomplicated Firewall) provides a simple interface for managing iptables rules on Ubuntu/Debian. These rules assume you have completed section 2 or 3 and are not exposing the OpenClaw gateway directly.

### 4.1 Basic setup

```bash
# Check current status
sudo ufw status verbose

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (adjust port if you changed it in section 2.3)
sudo ufw allow 22/tcp

# Enable the firewall
sudo ufw enable
```

### 4.2 Do not open port 18789 publicly

Do not add `sudo ufw allow 18789`. The OpenClaw gateway should remain bound to loopback or your Tailscale IP and accessed via tunnel only.

### 4.3 If using a reverse proxy with HTTPS (section 5)

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### 4.4 Verify rules

```bash
sudo ufw status numbered
```

Review the output and remove any rules you do not recognise with `sudo ufw delete <number>`.

---

## 5. Reverse proxy with HTTPS

If you want to access the OpenClaw dashboard via a domain name with a valid TLS certificate — rather than an SSH tunnel or raw Tailscale IP — set up a reverse proxy. This is optional and adds complexity. Only do this if you have a specific reason to expose the dashboard to the web; an SSH tunnel or Tailscale is simpler and more secure for personal use.

**Warning:** Exposing the OpenClaw dashboard to the public internet, even behind a reverse proxy, significantly increases your attack surface. Ensure you have completed sections 1–4 and that your dashboard auth token is rotated and strong before proceeding.

### 5.1 Prerequisites

A domain name with DNS pointing to your VPS IP. Caddy (recommended) or nginx installed.

### 5.2 Caddy configuration

Caddy handles TLS certificate issuance and renewal automatically via Let's Encrypt.

Install Caddy:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy
```

Edit `/etc/caddy/Caddyfile`:

```
openclaw.yourdomain.com {
    reverse_proxy 127.0.0.1:18789
    basicauth {
        # Add a second layer of auth in front of the dashboard token
        # Generate a hash with: caddy hash-password
        your-username $2a$14$...hashed-password...
    }
}
```

Restart Caddy:

```bash
sudo systemctl restart caddy
sudo systemctl status caddy
```

### 5.3 Additional hardening for public exposure

Restrict dashboard access by IP using Caddy's `remote_ip` matcher if you have a fixed IP:

```
openclaw.yourdomain.com {
    @allowed remote_ip 203.0.113.42
    handle @allowed {
        reverse_proxy 127.0.0.1:18789
    }
    respond 403
}
```

---

## 6. Monitoring and alerting

### 6.1 Check the OpenClaw security audit regularly

```bash
openclaw security audit --deep
```

Run this after any system update, after installing a new skill, and periodically on a schedule. The audit checks bind address, auth token presence, CVE patch status, channel allow-lists, and skill integrity.

### 6.2 Monitor auth logs for SSH brute-force attempts

```bash
sudo journalctl -u ssh --since "24 hours ago" | grep "Failed password"
sudo journalctl -u ssh --since "24 hours ago" | grep "Invalid user"
```

If you see large volumes of failed attempts, consider installing fail2ban:

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
```

The default fail2ban configuration bans IPs after 5 failed SSH attempts in 10 minutes.

### 6.3 Monitor outbound connections

If you are concerned about data exfiltration by a compromised skill or agent action, monitor outbound network connections:

```bash
# Show current established connections
ss -tunp | grep ESTABLISHED

# Watch connections in real time
watch -n 2 'ss -tunp | grep ESTABLISHED'
```

Unexpected outbound connections to unfamiliar IPs during an agent session are a red flag. Cross-reference against the skill audit output and the advisory list at openclaw.ai/security/advisories.

### 6.4 Review OpenClaw logs

```bash
openclaw logs --tail 100
openclaw logs --level warn
```

Look for repeated tool execution errors, unexpected skill calls, or auth failures in the gateway log.

---

## Hardening checklist

Before using OpenClaw for anything sensitive on a cloud VPS, verify all of the following:

Gateway bind address — `openclaw gateway status` shows `127.0.0.1:18789`, not `0.0.0.0:18789`

CVE patch status — `openclaw --version` shows 2026.1.29 or later for CVE-2026-25253; 2026.2.25 or later for CVE-2026-22176 (Windows only)

Auth token present — `openclaw gateway status` includes a token parameter in the dashboard URL

SSH password auth disabled — `PasswordAuthentication no` in `/etc/ssh/sshd_config`

UFW enabled with default-deny incoming — `sudo ufw status` shows Status: active

Channel allow-lists configured — `openclaw channels status` shows restricted allowFrom for each connected channel

Skills audited — `openclaw security audit --skills` shows no flagged packages

---

## Where to go next

Return to the [module index](../README.md) — all modules are now complete.
