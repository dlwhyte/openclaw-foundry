# OpenClaw Foundry

A practitioner's guide to building [OpenClaw](https://github.com/openclaw/openclaw) securely — with open source and proprietary frontier models.

---

## What this guide gives you

```mermaid
flowchart LR
    START(["🖥️ You &amp; a machine"])

    subgraph INSTALL ["⚙️  INSTALL"]
        direction TB
        M00["📖 **00 · Introduction**
        Architecture · CVEs
        Threat landscape"]
        M01["🔧 **01 · Setup**
        3 deploy paths
        Secure install
        Channel hardening"]
        M00 --> M01
    end

    subgraph CONFIGURE ["🤖  CONFIGURE"]
        direction TB
        M02["🧠 **02 · Model Config**
        API providers
        Local via Ollama
        Hybrid routing"]
        M03["🧩 **03 · Skills**
        ClawHub registry
        Vetting &amp; pinning
        Supply chain"]
        M02 --> M03
    end

    subgraph HARDEN ["🔒  HARDEN"]
        direction TB
        M04["💉 **04 · Prompt Injection**
        Attack patterns
        Defense strategies
        Testing"]
        M05["🌐 **05 · Network**
        Loopback · SSH
        Tailscale · UFW
        Reverse proxy"]
        M04 --> M05
    end

    END(["✅ OpenClaw,
    running securely"])

    START --> INSTALL
    INSTALL --> CONFIGURE
    CONFIGURE --> HARDEN
    HARDEN --> END

    style START fill:#d4edda,stroke:#28a745,color:#155724
    style END   fill:#d4edda,stroke:#28a745,color:#155724

    style INSTALL   fill:#fffde7,stroke:#f9a825
    style CONFIGURE fill:#e3f2fd,stroke:#1565c0
    style HARDEN    fill:#fce4ec,stroke:#c62828

    style M00 fill:#fff9c4,stroke:#f9a825,color:#333
    style M01 fill:#fff9c4,stroke:#f9a825,color:#333
    style M02 fill:#bbdefb,stroke:#1565c0,color:#333
    style M03 fill:#bbdefb,stroke:#1565c0,color:#333
    style M04 fill:#ffcdd2,stroke:#c62828,color:#333
    style M05 fill:#ffcdd2,stroke:#c62828,color:#333
```

| Phase | Modules | What you get |
|-------|---------|--------------|
| ⚙️ Install | 00 · 01 | Architecture understanding + a running, patched instance |
| 🤖 Configure | 02 · 03 | Model backend wired up + skills vetted and pinned |
| 🔒 Harden | 04 · 05 | Injection defenses + network locked down |

---

## What is this

OpenClaw is an open-source personal AI agent with shell access, browser control, and messaging integrations. That power creates a large attack surface. This guide covers how to install, configure, and harden OpenClaw so you get the productivity benefits without leaving your infrastructure exposed.

Each module is self-contained and can be read independently, but the recommended path is sequential.

---

## Modules

| # | Module | Status | Description |
|---|--------|--------|-------------|
| 00 | [Introduction](00-introduction/) | ✅ | What OpenClaw is, architecture, security landscape |
| 01 | [Setup](01-setup/) | ✅ | Deployment paths, install, security hardening |
| 02 | [Model Config](02-model-config/) | ✅ | Provider setup, local models, hybrid routing |
| 03 | [Skills & Plugins](03-skills-plugins/) | ✅ | ClawHub registry, vetting third-party skills |
| 04 | [Prompt Injection](04-prompt-injection/) | ✅ | Threat model, defenses, testing strategies |
| 05 | [Network Hardening](05-network-hardening/) | ✅ | VPS lockdown, Tailscale, firewall rules |

---

## Prerequisites

- Familiarity with the command line
- A machine or VPS to run OpenClaw on (see [01-setup](01-setup/))
- An API key from at least one model provider, or Ollama for local inference

---

## License

This work is licensed under the [MIT License](LICENSE).
