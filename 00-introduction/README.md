# 00 — Introduction

What OpenClaw is, how it works, why the security framing matters, and where to go next.

---

## What is OpenClaw

OpenClaw is an open-source personal AI agent that runs on your own hardware and connects large language models to the tools, apps, and messaging platforms you already use. Unlike a chatbot, it does not stop at generating text — it executes tasks: reading and writing files, running shell commands, browsing websites, sending emails, and controlling APIs.

The distinction matters. A chatbot answers questions. An agent acts. That capability gap is also a security gap.

OpenClaw's architecture separates concerns across three layers:

**Channel layer** — the messaging adapters that handle protocol normalization: WhatsApp (via Baileys), Telegram (via grammY), Slack (Bolt), Discord, iMessage, and 20+ others. This is where your instructions arrive.

**Brain layer** — the agent runtime. A seven-stage loop normalizes input, routes it, assembles context, calls the model, executes the ReAct pattern, loads relevant skills, and persists memory. This is where reasoning happens.

**Body layer** — the tools the agent uses to act: shell execution, browser control, file system access, API calls. This is where consequences happen.

Understanding which layer a security concern lives in is the first step toward addressing it.

---

## History and naming

OpenClaw was first published in November 2025 by Peter Steinberger under the name Clawdbot. Within two months it was renamed twice — first to Moltbot following trademark complaints from Anthropic, then to OpenClaw three days later.

By March 2026 the project had 247,000 GitHub stars and 47,700 forks. The founder joined OpenAI, with a non-profit foundation established to steward the project going forward.

---

## Security landscape

**CVE-2026-25253** — CVSS 8.8. Cross-site WebSocket hijacking allowing RCE via a single malicious link. Patched in 2026.1.29.

**CVE-2026-22176** — Command injection via Windows Scheduled Task auto-start. Patched in 2026.2.25. Windows native installs only.

**Canvas Host binding** — defaults to 0.0.0.0 (GitHub Issue #5263, closed "not planned"). Must be fixed manually on every install.

**ClawHavoc** — 1,467 malicious skills distributed via typosquatted ClawHub names in early 2026. Cisco independently confirmed data exfiltration via a third-party skill. The registry is safer now; it is not safe.

**Prompt injection** — any content the agent reads (email, web pages, documents, skill output) is a potential attack vector. This is the fundamental threat model for any agentic system with broad tool access. See 04-prompt-injection.

---

## Model support

**Open source** — Ollama + Llama 3.x, Mistral, DeepSeek. No API key required. Data stays on your hardware. 8 GB VRAM minimum for 7B models.

**Proprietary** — Claude, GPT-4o, Gemini via API. Pay-as-you-go billing required. As of April 4, 2026, subscription credits no longer work with OpenClaw.

**Hybrid** — OpenRouter as a single routing key across multiple providers.

See [02-model-config](../02-model-config/) for full configuration.

---

## References

### Primary
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw documentation](https://docs.openclaw.ai)
- [OpenClaw — Wikipedia](https://en.wikipedia.org/wiki/OpenClaw)
- [ClawHub skill registry](https://clawhub.ai)
- [NVIDIA NemoClaw](https://www.nvidia.com/en-us/ai/nemoclaw/)

### Security
- [CVE-2026-25253 — depthfirst](https://depthfirst.io/cve-2026-25253)
- [Cisco AI security skill audit](https://blogs.cisco.com/security)
- [OpenClaw security guide — Milvus](https://milvus.io/blog/openclaw-formerly-clawdbot-moltbot-explained-a-complete-guide-to-the-autonomous-ai-agent.md)
- [Build and secure a personal AI agent — freeCodeCamp](https://www.freecodecamp.org/news/how-to-build-and-secure-a-personal-ai-agent-with-openclaw/)
- [Hardened DigitalOcean deploy](https://www.digitalocean.com/community/tutorials/how-to-run-openclaw)
- [Isolated VPS guide — centminmod](https://github.com/centminmod/explain-openclaw)

### Context
- [Autonomous agents and accountability — Platformer](https://www.platformer.news)
- [OpenClaw ecosystem overview — KDnuggets](https://www.kdnuggets.com/openclaw-explained-the-free-ai-agent-tool-going-viral-already-in-2026)
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

## Where to go next

[01-setup](../01-setup/) — hardware options and the secure install sequence.

[04-prompt-injection](../04-prompt-injection/) and [05-network-hardening](../05-network-hardening/) can be read independently if you want to understand the threat model before installing.
