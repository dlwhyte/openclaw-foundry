# 00 — Introduction

What OpenClaw is, how it works, why the security framing matters, and where to go next.

---

## What is OpenClaw

OpenClaw is an open-source personal AI agent that runs on your own hardware and connects large language models to the tools, apps, and messaging platforms you already use. Unlike a chatbot, it does not stop at generating text — it executes tasks: reading and writing files, running shell commands, browsing websites, sending emails, and controlling APIs.

The distinction matters. A chatbot answers questions. An agent acts. That capability gap is also a security gap.

OpenClaw's architecture separates concerns across three layers:

**Channel layer** — the messaging adapters that handle protocol normalization: WhatsApp (via Baileys), Telegram (via grammY), Slack (Bolt), Discord, iMessage, and 20+ others.

**Brain layer** — the agent runtime. A seven-stage loop normalizes input, routes it, assembles context, calls the model, executes the ReAct pattern, loads relevant skills, and persists memory.

**Body layer** — the tools the agent uses to act: shell execution, browser control, file system access, API calls. This is where consequences happen.

---

## Security landscape

**CVE-2026-25253** — CVSS 8.8. Cross-site WebSocket hijacking allowing RCE via a single malicious link. Patched in 2026.1.29.

**CVE-2026-22176** — Command injection via Windows Scheduled Task auto-start. Patched in 2026.2.25. Windows native only.

**Canvas Host binding** — defaults to 0.0.0.0 (GitHub Issue #5263, closed "not planned"). Fix manually on every install.

**ClawHavoc** — 1,467 malicious skills distributed via typosquatted ClawHub names in early 2026. Registry is safer now; it is not safe.

---

## Model support

**Open source** — Ollama + Llama 3.x, Mistral, DeepSeek. No API key. Data stays on your hardware.

**Proprietary** — Claude, GPT-4o, Gemini via API. Pay-as-you-go only — subscription credits do not work as of April 4, 2026.

**Hybrid** — OpenRouter as a single routing key across multiple providers.

See 02-model-config for full configuration.

---

## References

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw docs](https://docs.openclaw.ai)
- [CVE-2026-25253](https://depthfirst.io/cve-2026-25253)
- [Cisco skill audit findings](https://blogs.cisco.com/security)
- [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [OpenClaw Wikipedia](https://en.wikipedia.org/wiki/OpenClaw)

---

## Where to go next

[01-setup](../01-setup/) — install sequence and security hardening.

[04-prompt-injection](../04-prompt-injection/) and [05-network-hardening](../05-network-hardening/) can be read independently.TEST CONTENT
