# 01 — Setup

Secure installation of OpenClaw across three deployment paths and two channel options.

Read this module before running any install command. The security hardening steps in section 4 are not optional — they must be completed before connecting any channel or installing any skill.

---

## Contents

1. Prerequisites
2. Deployment paths
3. Install
4. Security hardening
5. Model provider
6. Channel connection
7. Verification

---

## 1. Prerequisites

- Internet access during install
- An API key from at least one model provider, or Ollama installed for local models
- Node.js 24 (recommended) or 22.14+ LTS — the install script handles this automatically

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 2 GB | 4 GB |
| Disk | 10 GB | 20 GB |
| CPU | Any modern x86/ARM | — |

For local models via Ollama: 8 GB VRAM minimum for 7B parameter models, 24 GB+ for 70B models.

---

## 2. Deployment paths

### Path A — Spare computer

Any machine you control running macOS, Linux, or Windows.

macOS / Linux — the install script handles Node.js detection automatically.

Windows — use WSL2, not native PowerShell. Native Windows installs before 2026.2.25 are affected by CVE-2026-22176 (command injection via Scheduled Task auto-start). WSL2 is not affected.

```powershell
# Run as Administrator
wsl --install
```

Reboot, then continue from inside the WSL2 terminal.

### Path B — Cloud VPS

| Provider | Cost | Notes |
|----------|------|-------|
| Hostinger KVM 2 | $7/mo | Most polished onboarding, Docker template |
| DigitalOcean Droplet | $12–24/mo | 1-click marketplace image, hardened defaults |
| Oracle Cloud Free Tier | $0 | 4 ARM OCPUs, 24 GB RAM, never expires |

Minimum: 2 vCPUs, 4 GB RAM, Ubuntu 22.04+. Use a separate provider account from any production infrastructure.

Bind the gateway to loopback and access via Tailscale or SSH tunnel. See [05-network-hardening](../05-network-hardening/) for the full VPS hardening sequence.

### Path C — Raspberry Pi

| Model | RAM | Notes |
|-------|-----|-------|
| Pi 5 8GB | 8 GB | Recommended. ~3× faster than Pi 4 |
| Pi 4 8GB | 8 GB | Works, slower on complex tasks |
| Pi 4 4GB | 4 GB | Functional but not recommended for sustained use |
| Pi 3 or earlier | — | Not viable |

Runs on a residential IP — relevant for WhatsApp (see section 6).

---

## 3. Install

```bash
curl -fsSL https://openclaw.ai/install-cli.sh | bash
```

```bash
openclaw --version
openclaw doctor
```

Then run onboarding:

```bash
openclaw onboard --install-daemon
```

Onboarding covers: security warning acknowledgement, model provider selection, API key entry, workspace creation, optional channel connection. The --install-daemon flag registers the gateway as a background service.

---

## 4. Security hardening

Complete these steps before connecting any channel or installing any skill.

### 4.1 Patch CVEs

```bash
openclaw --version
```

| CVE | Fixed in | Scope |
|-----|----------|-------|
| CVE-2026-25253 | 2026.1.29 | All platforms — WebSocket hijacking → RCE |
| CVE-2026-22176 | 2026.2.25 | Windows native only — command injection |

Update if needed:

```bash
npm install -g openclaw@latest
```

### 4.2 Fix the Canvas Host binding

Edit ~/.openclaw/openclaw.json:

```json
{
  "gateway": {
    "bind": "127.0.0.1"
  }
}
```

```bash
openclaw gateway restart
openclaw gateway status
```

Status should show 127.0.0.1:18789, not 0.0.0.0:18789.

### 4.3 Verify the auth token

```bash
openclaw dashboard
```

If the Dashboard URL is missing the token parameter:

```bash
openclaw gateway token --rotate
```

Treat this token as a password. Do not commit openclaw.json to version control.

### 4.4 Restrict channel access

```json
{
  "channels": {
    "telegram": {
      "allowFrom": ["your_telegram_user_id"]
    },
    "whatsapp": {
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"]
    }
  }
}
```

---

## 5. Model provider

API key required — subscription credits do not work. As of April 4, 2026, Claude Pro and Max subscription credits no longer work with third-party tools. A pay-as-you-go API key is required. The same applies to OpenAI.

For local models: select "Skip for now" during onboarding, then configure Ollama per [02-model-config](../02-model-config/).

```bash
openclaw models list
```

If the list is empty, the API key is invalid or billing is not configured.

---

## 6. Channel connection

### Telegram (recommended for all paths)

1. Message @BotFather in Telegram
2. Send /newbot and follow the prompts
3. Copy the bot token
4. Add to ~/.openclaw/openclaw.json:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "your_bot_token_here",
      "allowFrom": ["your_telegram_user_id"]
    }
  }
}
```

5. Restart the gateway and send /start to your bot

### WhatsApp (residential IPs only)

Uses Baileys — an unofficial library that reverse-engineers the WhatsApp Web protocol. Works reliably from residential IPs (Paths A and C). High ban risk from datacenter IPs (Path B).

Key risks:
- WhatsApp bans accounts silently — messages stop with no error
- Sessions expire and require periodic QR re-scanning
- Only install via `openclaw plugins install @openclaw/whatsapp` — a malicious Baileys npm lookalike was found in late 2025
- Session credentials at ~/.openclaw/credentials/whatsapp/ are sensitive

Use a dedicated phone number. Do not use your personal account.

```bash
openclaw channels login --channel whatsapp
```

Scan the QR code immediately — it expires in approximately 60 seconds. On your phone: WhatsApp → Settings → Linked Devices → Link a Device.

---

## 7. Verification

```bash
openclaw gateway status
openclaw channels status
openclaw security audit --deep
```

Open the dashboard at http://127.0.0.1:18789. Send a test message from your connected channel and confirm the agent responds.

---

## Compatibility matrix

| | Telegram | WhatsApp |
|--|----------|----------|
| Path A — Spare computer | Works | Works |
| Path B — Cloud VPS | Works | High ban risk |
| Path C — Raspberry Pi | Works | Works |

---

## Where to go next

[02-model-config](../02-model-config/) — configure your model backend in detail.

[05-network-hardening](../05-network-hardening/) — if you are on Path B, complete network hardening before using OpenClaw for anything sensitive.
