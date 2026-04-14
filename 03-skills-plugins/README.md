# 03 — Skills & Plugins

Vetting, installing, and managing ClawHub skills — and avoiding the mistakes that led to ClawHavoc.

---

## Contents

1. How skills work — execution model and permissions
2. The ClawHub registry — what changed after ClawHavoc
3. Vetting third-party skills before installation
4. Installing and managing skills
5. Writing your own skills
6. Pinning versions and auditing updates

---

## 1. How skills work — execution model and permissions

Skills (also called plugins) are Node.js packages that extend the agent's capabilities. They run inside the OpenClaw body layer and have direct access to the tool execution pipeline. When the agent decides a skill is relevant to a task, it calls the skill's exported handler with the current context.

Each skill declares a manifest that specifies what it can do:

```json
{
  "name": "@openclaw/web-search",
  "version": "1.2.0",
  "permissions": ["network", "read"],
  "tools": ["search", "fetch_url"]
}
```

### Permission tiers

| Permission | What it grants |
|------------|---------------|
| `read` | Read files from the workspace directory |
| `write` | Write files to the workspace directory |
| `network` | Make outbound HTTP requests |
| `shell` | Execute shell commands |
| `memory` | Read and write persistent agent memory |

`shell` is the highest-risk permission. Any skill that requests it deserves extra scrutiny — most tasks do not require arbitrary shell access.

### How skills receive data

Skills receive the full message context, tool results from previous steps in the ReAct loop, and any workspace files the agent has loaded. A malicious skill can read everything the agent currently holds in context — treat skill permissions as a trust boundary accordingly.

---

## 2. The ClawHub registry — what changed after ClawHavoc

ClawHub is the official skill registry at [clawhub.ai](https://clawhub.ai). In early 2026, a campaign called **ClawHavoc** distributed 1,467 malicious skills via typosquatted package names — names designed to look like legitimate skills with one or two characters changed (e.g. `@openclaw/web-serach`, `@openclaw/calender`).

Cisco independently confirmed data exfiltration via a third-party skill installed from one of these packages. The exfiltrated data included workspace files and agent memory contents.

### What changed after ClawHavoc

- ClawHub introduced mandatory code signing for all new skill submissions.
- The registry added a verified publisher badge for packages audited by the OpenClaw foundation.
- Existing packages published before 2026.3.1 were not retroactively audited — they remain available but are marked as unverified.
- The npm lookalike packages were removed from the npm registry but may still be referenced in older install guides.

### What did not change

The registry is safer. It is not safe. Code signing verifies that a package came from a specific publisher key — it does not verify that the publisher's code is trustworthy. A signed skill from an unknown publisher is still an unknown publisher's code running in your agent.

---

## 3. Vetting third-party skills before installation

Never install a skill without checking it first. The steps below apply to any skill not published by the OpenClaw foundation.

### 3.1 Check the package name carefully

Typosquatting is the primary attack vector. Before installing, verify the exact package name against the ClawHub website — do not copy names from third-party guides, forum posts, or LLM output.

| Legitimate | Typosquatted lookalike |
|------------|----------------------|
| `@openclaw/web-search` | `@openclaw/web-serach` |
| `@openclaw/whatsapp` | `@0penclaw/whatsapp` |
| `@openclaw/calendar` | `@openclaw/calender` |

For WhatsApp specifically: only install via `openclaw plugins install @openclaw/whatsapp`. A malicious Baileys npm lookalike was found in late 2025 and is still occasionally referenced in older tutorials.

### 3.2 Review the manifest and source

```bash
openclaw plugins info @author/skill-name
```

Check the `permissions` field. If a skill claims `shell` permission but its stated purpose is calendar access or weather lookup, that is a red flag.

Browse the source on GitHub or npm before installing. Look for:

- Outbound network calls to unknown endpoints
- Reads of `~/.openclaw/` or credential paths
- Base64-encoded strings or obfuscated logic
- Dynamic `require()` or `eval()` calls

### 3.3 Check publisher history

A skill published yesterday by an account with no prior history is higher risk than one with years of commits and a public issue tracker. Check the publisher's GitHub profile and the package's npm page.

### 3.4 Run in a sandboxed workspace first

Before using a new skill in your main workspace, test it against a throwaway workspace with no sensitive files or memory:

```bash
openclaw workspace create --name sandbox
openclaw config set workspace sandbox
openclaw plugins install @author/skill-name
# test the skill
openclaw workspace switch default
```

### 3.5 Check against the ClawHavoc list

The OpenClaw security team maintains a list of known malicious packages at [openclaw.ai/security/advisories](https://openclaw.ai/security/advisories). Check any unfamiliar package name before installing.

---

## 4. Installing and managing skills

### Install a skill

```bash
openclaw plugins install @openclaw/web-search
```

Always use the `openclaw plugins install` command rather than `npm install` directly. The OpenClaw installer validates the manifest and registers the skill with the gateway. Installing via npm bypasses this validation.

### List installed skills

```bash
openclaw plugins list
```

### Check skill permissions

```bash
openclaw plugins info @openclaw/web-search
```

### Disable a skill without uninstalling

```bash
openclaw plugins disable @openclaw/web-search
openclaw plugins enable @openclaw/web-search
```

### Uninstall a skill

```bash
openclaw plugins uninstall @openclaw/web-search
```

### Audit all installed skills

```bash
openclaw security audit --skills
```

This checks installed skills against the advisory list and flags any with known CVEs or that match the ClawHavoc signature patterns.

---

## 5. Writing your own skills

Custom skills are the safest option — you control the code and permissions.

### Scaffold a new skill

```bash
openclaw plugins new my-skill
cd my-skill
```

This creates the following structure:

```
my-skill/
  index.js          # skill entry point
  manifest.json     # name, version, permissions, tools
  README.md
  package.json
```

### Minimal skill example

```js
// index.js
module.exports = {
  tools: {
    get_weather: async ({ location }, context) => {
      const res = await fetch(`https://wttr.in/${encodeURIComponent(location)}?format=3`);
      return res.text();
    }
  }
};
```

```json
{
  "name": "my-skill",
  "version": "1.0.0",
  "permissions": ["network"],
  "tools": ["get_weather"]
}
```

### Install a local skill

```bash
openclaw plugins install ./my-skill
```

### Request only the permissions you need

Declare the minimum permissions necessary. If your skill only makes outbound HTTP calls, it should only request `network` — not `shell`, `write`, or `memory`. Over-privileged skills are a liability if the underlying dependency is compromised.

---

## 6. Pinning versions and auditing updates

### Why pinning matters

Skills auto-update by default. An update to a skill you trust is still new code that you have not reviewed. Supply chain attacks work by compromising a trusted package and pushing a malicious update — this is how the ClawHavoc packages operated in several cases.

### Pin a skill to a specific version

```bash
openclaw plugins install @openclaw/web-search@1.2.0
```

Or set a pin in `~/.openclaw/openclaw.json`:

```json
{
  "plugins": {
    "@openclaw/web-search": {
      "version": "1.2.0",
      "pin": true
    }
  }
}
```

### Disable auto-update globally

```bash
openclaw config set plugins.autoUpdate false
```

### Review pending updates before applying

```bash
openclaw plugins outdated
```

Review the changelog and diff before updating a pinned skill:

```bash
openclaw plugins changelog @openclaw/web-search
openclaw plugins diff @openclaw/web-search 1.2.0 1.3.0
```

### Run a full security audit after any update

```bash
openclaw security audit --deep
```

---

## Where to go next

[04-prompt-injection](../04-prompt-injection/) — skills are a major prompt injection surface; understanding the threat model is the logical next step.
[05-network-hardening](../05-network-hardening/) — if you are on Path B (cloud VPS), complete network hardening before using any third-party skills.
