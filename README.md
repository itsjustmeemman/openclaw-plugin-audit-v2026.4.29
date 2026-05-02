# OpenClaw Plugin Audit — v2026.4.29

**Every bundled plugin. What it does. When it loads. Whether you can disable it. What breaks if you do.**

<img src="https://drive.google.com/uc?export=view&id=1nbrMnHupdSQ4sZEcTJVSUXHnwbeGP3O-" alt="OpenClaw Plugin Audit" width="100%" />

---

## Why This Exists

OpenClaw ships with **120 bundled plugins** across 13 categories. Nobody has documented all of them in one place. Until now.

Most OpenClaw users — and even people deploying it in production — don't know:

- **Which plugins load at startup** (22 of them — affecting cold-boot latency and first-response time)
- **Which plugins you can safely disable** (90 of them — reducing memory, dependency bloat, and attack surface)
- **Which plugins you must NEVER disable** (4 of them — `openai`, `memory-core`, `acpx`, `web-readability`)
- **What each plugin actually does** beyond its directory name
- **What breaks if you disable a specific plugin** — exact error messages, cascading failures, which other plugins depend on it
- **How heavy each plugin is** — npm dependency weight, whether it needs a browser, a GPU, an API key
- **What the `hosted-minimal` profile actually disables** on Fly.io first boot

This repo answers all of those questions, exhaustively, for every single bundled plugin in OpenClaw v2026.4.29.

---

## How This Was Built

This audit was performed by 20 parallel AI agents scanning the full OpenClaw source tree (`openclaw-sources/v2026.4.29/`). Each agent read every plugin's:

- `openclaw.plugin.json` — the plugin manifest (activation, contracts, providers, config schema, defaults)
- `package.json` — runtime dependencies, npm weight, plugin metadata
- `index.ts` — what it actually registers at runtime (tools, commands, hooks, providers, services)

The analysis covers:
- Cross-plugin dependencies (what depends on what)
- The full plugin loading pipeline (discovery → manifest → dedup → Jiti import → register)
- Runtime dependency auto-installation (when npm/pnpm is spawned at startup)
- Fly.io deployment plugin flow (from Dockerfile build through first-boot config seeding)

---

## What's in This Repo

### [`PLUGIN-CATALOG.md`](./PLUGIN-CATALOG.md)

All 120 plugins, one by one, with:

| Column | What It Tells You |
|--------|-------------------|
| **Plugin** | Plugin ID and directory name |
| **Load** | 🔴 startup or 🟢 lazy |
| **Default** | ✅ enabled or ❌ disabled by default |
| **Rating** | 🟡 safe / 🔵 check config / ⚠️ never disable |
| **What It Does** | 1-sentence purpose |
| **Disable Notes** | Full paragraph: what it registers, what breaks, what depends on it, what heavy deps you free, what alternatives exist |

### [`PLUGIN-ARCHITECTURE.md`](./PLUGIN-ARCHITECTURE.md)

How the plugin system actually works:

- Plugin types and the registration API (37 methods)
- The loading pipeline — step by step from `openclaw.mjs` through `loadOpenClawPlugins()`
- Discovery mechanism (bundled, global, workspace, config)
- Runtime dependency installation (build-time, bulk pre-stage, per-plugin)
- Priority/ordering systems (origin precedence, hook priority, channel preferOver, autoSelect)
- Fly.io deployment plugin flow (Dockerfile → deploy script → start.sh → gateway startup)
- All environment variables that control plugin behavior

---

## Quick Reference: The 4 You Must Never Disable

| Plugin | Why |
|--------|-----|
| **`openai`** | 40+ GPT models, the `codex-cli` CLI backend, system prompt personality overlays, 8 capability contracts. The `codex` harness plugin depends on the `codex-cli` backend registered here. |
| **`memory-core`** | Hardcoded default memory slot. All `memory_search`/`memory_get` tools, dreaming engine, flush pipeline. If removed without replacing `plugins.slots.memory`, every agent loses memory. |
| **`acpx`** | Sole ACP runtime backend. Disabling kills Claude Code via ACP, Gemini CLI via ACP, Codex-ACP, OpenCode ACP. Error: `runtime="acp" is unavailable... Enable the acpx plugin or use runtime="subagent"`. |
| **`web-readability`** | Primary web content quality path. Every HTML web fetch runs through Mozilla Readability via this plugin. Disable it and agents get raw HTML instead of clean article text. |

---

## Quick Reference: Top 10 Heaviest Plugins to Disable

| Plugin | What You Free |
|--------|---------------|
| **`browser`** | Chromium (~300MB), Playwright, Express, WS, MCP SDK |
| **`matrix`** | WASM crypto, fake-indexeddb, full matrix-js-sdk |
| **`msteams`** | Azure Identity SDK, Express, JWT, Teams SDK |
| **`whatsapp`** | @whiskeysockets/baileys (~15MB+), jimp (~5MB) |
| **`google`** | @google/genai SDK, @mariozechner/pi-ai |
| **`diagnostics-otel`** | 11 @opentelemetry/* packages |
| **`tlon`** | AWS S3 SDK (~60MB) |
| **`memory-lancedb`** | @lancedb/lancedb native vector DB binary |
| **`acpx`** | @agentclientprotocol/claude-agent-acp, codex-acp, acpx |
| **`diffs`** | playwright-core (separate Chromium launch) |

---

## Quick Reference: Startup Plugins (impact cold-boot)

These 22 plugins load at gateway startup — every single one adds to first-response latency and memory pressure:

`memory-wiki`, `google-meet`, `browser`, `device-pair`, `diffs`, `file-transfer`, `llm-task`, `lobster`, `phone-control`, `skill-workshop`, `thread-ownership`, `webhooks`, `acpx`, `openshell`, `diagnostics-otel`, `diagnostics-prometheus`, `bonjour`, `voice-call`, `active-memory`, `talk-voice`

If you're not using these, explicitly deny them in your `openclaw.json` `plugins.deny` list for faster boots.

---

## Fly.io Default: `hosted-minimal` Profile

On first boot, the `THIRDCLAW_PLUGIN_PROFILE` env var determines which plugins are disabled. The default (`hosted-minimal`) disables exactly 4:

| Plugin | Why Disabled |
|--------|--------------|
| `amazon-bedrock` | Heavy AWS SDK, rarely used in managed hosting |
| `amazon-bedrock-mantle` | Heavy AWS SDK + Anthropic SDK |
| `anthropic-vertex` | Requires GCP ADC setup |
| `codex` | Heavy Codex agent harness, keep disabled if using non-Codex backends |

To enable all plugins: set `THIRDCLAW_PLUGIN_PROFILE=upstream-default` at deploy time.

---

## Version

- **OpenClaw version**: v2026.4.29
- **Audit date**: 2026-05-02
- **Source scanned**: `openclaw-sources/v2026.4.29/extensions/`
- **Method**: 20 parallel AI agents reading every manifest, package.json, and source file

---

## How This Was Made

This audit was powered by **DeepSeek V4 Pro** via Codex CLI, running **10 parallel AI agents** reading every plugin manifest, package.json, and source file simultaneously. Then again with **10 more agents** for detailed disable-safety analysis. 

**19 million tokens. $1.27 in API costs.** Worth every cent.

<img src="https://drive.google.com/uc?export=view&id=1CPAelBHNJ46flxid5ZkZqkNDL4yCnMyZ" alt="DeepSeek V4 Pro usage statistics" width="100%" />

---

## License

This audit is provided as reference documentation. The original OpenClaw source code is © the OpenClaw project and its contributors. This repo contains only derived analysis and documentation.
