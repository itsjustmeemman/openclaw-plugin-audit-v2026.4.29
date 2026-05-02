# OpenClaw v2026.4.29 — Complete Plugin Catalog with Disable Analysis

**Date:** 2026-05-02  
**Source:** `openclaw-sources/v2026.4.29/extensions/`  
**Tool:** Codex CLI (deepseek-v4-pro), 20 parallel agents total  
**Role:** [codex-cli] [executor]

---

## Plugin Profile Legend

| Icon | Meaning |
|------|---------|
| 🔴 | Loads at **startup** (`onStartup: true`) |
| 🟢 | Lazy-loaded on demand |
| ✅ | Enabled by default |
| ❌ | NOT enabled by default |
| 🟡 | Safe to disable if unused |
| 🔵 | Only disable if no config references it |
| ⚠️ | Do NOT disable (core infrastructure) |

---

## 1. LLM Provider Plugins (51 plugins)

### Batch A

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 1 | `alibaba` | 🟢 | ✅ | 🟡 | Video generation only (DashScope/Qwen API) | Safe to disable — provides only a `videoGenerationProviders` contract via `MODELSTUDIO_API_KEY`/`DASHSCOPE_API_KEY`/`QWEN_API_KEY`. No other plugin depends on Alibaba video generation. No commands, harnesses, or chat providers registered. `onStartup: false`, zero impact. Alternative: `byteplus`, `comfy` for video generation. |
| 2 | `amazon-bedrock-mantle` | 🟢 | ✅ | 🟡 | Bedrock with OpenAI-compatible API + Anthropic streams | Safe to disable — single chat provider (provider ID `amazon-bedrock-mantle`) with OpenAI-compatible wrapper. No contracts, no commands, no harnesses. If disabled, the OpenAI-compatible Bedrock Mantle layer disappears but the native `amazon-bedrock` plugin remains available. Zero AWS-SDK-free alternative for this specific wrapper. |
| 3 | `amazon-bedrock` | 🟢 | ✅ | 🔵 | Native Bedrock LLM + memory embedding provider | Disable only if no memory features use `bedrock` embeddings. Registers the `amazon-bedrock` chat provider AND fulfills `memoryEmbeddingProviders` contract `bedrock`. If memory-lancedb or memory-wiki are configured to use `bedrock` embeddings, disabling causes memory embedding failures. Extensive Bedrock-specific config (guardrails, discovery, cache points). The `amazon-bedrock-mantle` plugin is NOT a substitute for the embedding contract. |
| 4 | `anthropic-vertex` | 🟢 | ✅ | 🟡 | Claude via GCP Vertex AI (ADC credentials) | Safe to disable — single chat provider (`anthropic-vertex`) routing Anthropic through GCP Vertex AI using `@anthropic-ai/vertex-sdk`. Requires GCP Application Default Credentials setup. No contracts, no downstream dependencies. The main `anthropic` plugin (direct API/CLI) is the standard alternative for Claude models. |
| 5 | `anthropic` | 🟢 | ✅ | 🔵 | Anthropic API (Claude) + Claude CLI backend + media understanding | **Critical tier-1 provider** — disables cautiously. Registers: `anthropic` chat provider (Claude API key, OAuth, setup-token auth), `claude-cli` CLI backend (`api.registerCliBackend`), and `mediaUnderstandingProviders` contract `anthropic` (image/PDF via Claude Opus 4.7). Disabling breaks ALL Claude model usage, the Claude CLI backend, and Anthropic media understanding. Any agent using `claude-*` models fails. `anthropic-vertex` covers only the GCP chat path, not CLI backend or media understanding. |
| 6 | `arcee` | 🟢 | ✅ | 🟡 | Arcee AI (GLM models, direct or via OpenRouter) | Safe to disable — single chat provider (`arcee`) with dual auth: direct Arcee API key or OpenRouter passthrough. Uses OpenAI-compatible replay hooks. No contracts, no downstream dependents. Alternative GLM models available via `zai`, `byteplus`, `together`. |
| 7 | `byteplus` | 🟢 | ✅ | 🟡 | BytePlus LLM (standard + coding plan) + video generation | Safe to disable — two chat providers (`byteplus`, `byteplus-plan`) plus `videoGenerationProviders` contract. Static catalog: Seed 1.8, Kimi K2.5, GLM 4.7 and coding-plan models. No downstream dependents. Alternatives: `alibaba` for video, multiple providers for overlapping model families. |
| 8 | `cerebras` | 🟢 | ✅ | 🟡 | Cerebras CS-3 high-speed inference | Safe to disable — single chat provider (`cerebras`) on `api.cerebras.ai` with static model catalog. No contracts, no downstream dependents. Models (GLM, GPT-OSS, Qwen, Llama) available through other providers (chutes, arcee, zai, together). |
| 9 | `chutes` | 🟢 | ✅ | 🟡 | Chutes.ai OAuth + 40+ open-source models | Safe to disable — single chat provider (`chutes`) with OAuth or API key auth, refreshable catalog of 40+ models. No contracts, no downstream dependents. Many models available through other providers individually, but the unified Chutes endpoint convenience is lost. |
| 10 | `cloudflare-ai-gateway` | 🟢 | ✅ | 🟡 | Cloudflare AI Gateway passthrough (caching, logging) | Safe to disable — single chat provider (`cloudflare-ai-gateway`) serving as a transparent passthrough proxy. No contracts. Underlying providers remain directly accessible. This is an optional intermediary layer, not a primary model provider. |
| 11 | `codex` | 🟢 | ✅ | 🔵 | Codex agent harness (GPT) + `/codex` command + media | **Deeply integrated** — disables with care. Registers: `codex` chat provider, Codex agent harness (`onAgentHarnesses: ["codex"]`), `mediaUnderstandingProviders` (image via GPT), `/codex` slash command, and event hooks (`inbound_claim`, `onConversationBindingResolved`). Disabling removes the Codex harness workflow, `/codex` command, GPT model catalog, and Codex media understanding. Agents using `codex` harness fail to activate. Startup unaffected (`onStartup: false`). No lightweight alternative within workspace. |
| 12 | `comfy` | 🟢 | ✅ | 🟡 | ComfyUI local/cloud media gen (image+music+video) | Safe to disable if ComfyUI unused. Registers `comfy` provider (auth surface) plus three contracts: `imageGenerationProviders`, `musicGenerationProviders`, `videoGenerationProviders`. Supports local/cloud modes with per-modality workflow config. No downstream dependents. Alternatives: limited — `alibaba`/`byteplus` for video, no bundled alternative for ComfyUI image/music generation workflows. |

### Batch B

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 13 | `copilot-proxy` | 🟢 | ✅ | 🟡 | GitHub Copilot via local VS Code proxy (no API key) | Safe to disable — single chat provider (`copilot-proxy`) requiring a running VS Code Copilot Proxy on the host, using a `custom` local auth method. Zero contracts, zero tools, zero CLI backends. If disabled, the Copilot Proxy option disappears from the model list. Alternative: `github-copilot` for direct Copilot OAuth access without a local proxy. |
| 14 | `deepinfra` | 🟢 | ✅ | 🟡 | DeepInfra LLM + 5 contracts (image/video/speech/media/embeddings) | **Multi-contract provider** — very broad but no dependents. Registers: `deepinfra` chat provider plus FIVE contracts — `mediaUnderstandingProviders` (image+audio, autoPriority 45), `memoryEmbeddingProviders`, `imageGenerationProviders`, `speechProviders`, `videoGenerationProviders`. If disabled, all six capabilities disappear. System features that auto-select `deepinfra` will fall back (e.g., image understanding to `google` at priority 30). No other plugin depends on it. |
| 15 | `deepseek` | 🟢 | ✅ | 🟡 | Native DeepSeek API (V4 Flash/Pro, Chat, Reasoner) | Safe to disable — single chat provider (`deepseek`) with custom V4 thinking profile (off/minimal/low/medium/high/xhigh/max). Zero contracts, zero CLI backends. No downstream dependents. DeepSeek models also available via `deepinfra` host. |
| 16 | `fal` | 🟢 | ✅ | 🟡 | fal.ai image + video generation (no LLM chat) | Safe to disable — single provider (`fal`) plus two contracts: `imageGenerationProviders` and `videoGenerationProviders`. Does NOT register a chat/text LLM provider. No dependents. Alternatives: `google`, `deepinfra` for image/video generation. |
| 17 | `fireworks` | 🟢 | ✅ | 🟡 | Fireworks AI LLM (dynamic model resolution) | Safe to disable — single chat provider (`fireworks`) with `resolveDynamicModel` support. Zero contracts. Dependent on `@mariozechner/pi-ai`. No downstream dependents. Alternative Kimi models available via `deepinfra`, native `kimi` plugin. |
| 18 | `github-copilot` | 🟢 | ✅ | 🟡 | GitHub Copilot OAuth (19 models) + embeddings | Safe to disable but commonly relied upon. Registers `github-copilot` chat provider (device OAuth, 20 cataloged models) plus `memoryEmbeddingProviders` contract. If disabled, all Copilot-hosted models and embeddings disappear — a very commonly used default. No other plugin depends on it. Alternative: `copilot-proxy` for VS Code proxy path. |
| 19 | `google` | 🟢 | ✅ | 🔵 | Google AI: 3 LLM providers + 8 capability contracts | **Most contract-heavy plugin** — disable very carefully. Registers THREE providers (`google`, `google-gemini-cli`, `google-vertex`) and EIGHT contracts: `mediaUnderstandingProviders` (image/audio/video, auto-priorities 30/40/10), `memoryEmbeddingProviders` (as `gemini`), `imageGenerationProviders`, `musicGenerationProviders`, `realtimeVoiceProviders`, `speechProviders`, `videoGenerationProviders`, `webSearchProviders` (as `gemini`). Also registers `google-gemini-cli` CLI backend. If disabled, Gemini disappears from ALL these surfaces simultaneously. Depends on `@google/genai` + `@mariozechner/pi-ai`. No single alternative covers all. |
| 20 | `gradium` | 🟢 | ✅ | 🟡 | Gradium TTS speech only (no LLM) | Safe to disable — slimmest plugin in the ecosystem. Registers only one `speechProviders` contract (`gradium`). No `providers` array in manifest, no chat, no CLI backends. If disabled, Gradium TTS disappears. Alternatives: `google`, `deepinfra` for speech. |
| 21 | `groq` | 🟢 | ✅ | 🟡 | Groq LLM + Whisper audio transcription | Safe to disable — registers `groq` chat provider (auth: empty list, auto-detected from env) and `mediaUnderstandingProviders` contract (audio via `whisper-large-v3-turbo`, auto-priority 20). Provides model compat for Qwen/GPT-OSS reasoning profiles. No dependents. Alternatives: `deepinfra`, `google` for audio media understanding. |
| 22 | `huggingface` | 🟢 | ✅ | 🟡 | Hugging Face inference API with model discovery | Safe to disable — single chat provider (`huggingface`) with dynamic model discovery toggled by `discovery.enabled`. Strips `huggingface/` prefix from model IDs. Zero contracts. No dependents. No bundled alternative directly wraps the HF API. |
| 23 | `kilocode` | 🟢 | ✅ | 🟡 | Kilo Gateway (OpenRouter-compatible) | Safe to disable — single chat provider (`kilocode`) with refreshable discovery, one static model (`kilo/auto`), Gemini replay hooks. Passthrough pricing to OpenRouter/LiteLLM. No contracts, no dependents. |
| 24 | `kimi` | 🟢 | ✅ | 🟡 | Kimi K2.6 coding API (Moonshot AI) | Safe to disable — two providers (`kimi`, `kimi-coding`) sharing auth family `moonshot`. Custom coding endpoint, two-level thinking (off/low), `KIMI_REPLAY_POLICY`. Zero contracts. Depends on `@mariozechner/pi-ai`. No dependents. Same underlying models available via `deepinfra`, `fireworks`. |

### Batch C

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 25 | `litellm` | 🟢 | ✅ | 🟡 | LiteLLM unified gateway (100+ providers) + image gen | Safe to disable — single chat provider (`litellm`) plus `imageGenerationProviders` contract. Uses simple-order catalog with `LITELLM_API_KEY`. No dependents. Alternative: `openrouter` provides similar multi-provider gateway access. |
| 26 | `lmstudio` | 🟢 | ✅ | 🟡 | Local LM Studio server for LLM + embeddings | Safe to disable — single chat provider (`lmstudio`) with `external: false` pricing, dynamic model discovery, and `memoryEmbeddingProviders` contract. Uses `LM_API_TOKEN`, synthetic local auth. No dependents. Alternative: `ollama` for local model serving. |
| 27 | `microsoft-foundry` | 🟢 | ✅ | 🟡 | Azure OpenAI via Microsoft Foundry (Entra ID or API key) | Safe to disable — single chat provider (`microsoft-foundry`) with dual auth (Entra ID via `az login` or `AZURE_OPENAI_API_KEY`). Zero contracts. No dependents. Alternative: native `openai` plugin for direct OpenAI access. |
| 28 | `minimax` | 🟢 | ✅ | 🔵 | MiniMax AI: LLM×2 + speech + image/music/video + media + web search | **Multi-contract** — not trivially safe. Registers two providers (`minimax` with API key, `minimax-portal` with OAuth) and SIX contracts: `speechProviders`, `mediaUnderstandingProviders` (image, both IDs, priority 40-50), `imageGenerationProviders`, `musicGenerationProviders`, `videoGenerationProviders`, `webSearchProviders` (as `minimax`). Has `autoEnableWhenConfiguredProviders` — re-enables itself if config references it. No dependents. |
| 29 | `mistral` | 🟢 | ✅ | 🟡 | Mistral AI: LLM + embeddings + audio media + transcription | Safe to disable — single chat provider (`mistral`) with static 7-model catalog plus three contracts: `memoryEmbeddingProviders`, `mediaUnderstandingProviders` (audio via `voxtral-mini-latest`, priority 50), `realtimeTranscriptionProviders`. No dependents. Alternatives: `google`, `deepinfra` cover some contracts. |
| 30 | `moonshot` | 🟢 | ✅ | 🟡 | Moonshot Kimi models + media (image/video) + Kimi web search | Safe to disable — single chat provider (`moonshot`) with 5 static models plus `mediaUnderstandingProviders` (image+video) and `webSearchProviders` (as `kimi` — distinct from the `kimi-coding` extension). The `kimi` web search contract ID is registered ONLY here. No dependents. |
| 31 | `nvidia` | 🟢 | ✅ | 🟡 | NVIDIA hosted open models (Nemotron, Kimi, GLM) | Safe to disable — single chat provider (`nvidia`) with static 4-model catalog. Zero contracts. `modelIdNormalization` prefixes bare IDs with `nvidia`. No dependents. Models also available via `together`, `volcengine`. |
| 32 | `ollama` | 🟢 | ✅ | 🟡 | Local/remote Ollama: LLM + embeddings + media + web search | Safe to disable — single chat provider (`ollama`) with `external: false` pricing, dynamic model discovery, WSL2 crash-loop check, and THREE contracts: `memoryEmbeddingProviders`, `mediaUnderstandingProviders`, `webSearchProviders`. No dependents. Alternative: `lmstudio` for local models. |
| 33 | `openai` | 🟢 | ✅ | ⚠️ | **Core infra**: GPT-4 to GPT-5.5 + Codex backend + 8 contracts | **DO NOT DISABLE** — the most critical plugin. Registers: two providers (`openai` with 40+ static models, `openai-codex` with OAuth + 3 Codex models), `codex-cli` CLI backend (via `api.registerCliBackend`), and EIGHT contracts covering speech, realtime transcription/voice, memory embeddings, media understanding (image+audio, priority 10), image generation, and video generation. Disabling removes 40+ models, breaks the Codex CLI backend, and loses all OpenAI contracts. The `codex` extension depends on the `codex-cli` backend registered here. System prompt personality mode contributions stop. |
| 34 | `opencode-go` | 🟢 | ✅ | 🟡 | OpenCode Go catalog (DeepSeek, Kimi, GLM coding models) | Safe to disable — single chat provider (`opencode-go`, family `opencode`) plus `mediaUnderstandingProviders` (image via `kimi-k2.6`). Shares `OPENCODE_API_KEY` credential profile with sibling `opencode`. Disabling only removes the Go catalog; Zen catalog in `opencode` remains unaffected. |
| 35 | `opencode` | 🟢 | ✅ | 🟡 | OpenCode Zen catalog (curated: Claude, GPT, Gemini, etc.) | Safe to disable — single chat provider (`opencode`, family `opencode`) plus `mediaUnderstandingProviders` (image). Dynamic models fetched from Zen API. Shares credential profiles with `opencode-go`. Disabling removes the Zen catalog; Go catalog continues independently. |
| 36 | `openrouter` | 🟢 | ✅ | 🟡 | OpenRouter multi-provider: LLM + image/video gen + speech + media | Safe to disable — single chat provider (`openrouter`) with simple-order dynamic catalog plus FOUR contracts: `mediaUnderstandingProviders` (image, model `auto`), `imageGenerationProviders`, `videoGenerationProviders`, `speechProviders`. Passthrough pricing, cache eligibility for anthropic/deepseek/moonshot/zai prefixes. No dependents. Alternative: `litellm` for similar gateway access. |

### Batch D

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 37 | `qianfan` | 🟢 | ✅ | 🟡 | Baidu Qianfan (Ernie/DeepSeek hosted models) | Safe to disable — single chat provider (`qianfan`) with static 2-model catalog on `qianfan.baidubce.com`. Zero contracts. No dependents. No alternative provides same Baidu-hosted models. |
| 38 | `qwen` | 🟢 | ✅ | 🟡 | Alibaba Qwen Cloud: 4 provider aliases + media + video gen | Safe to disable but note contract loss — four provider IDs (`qwen`, `qwencloud`, `modelstudio`, `dashscope`) all backing the `modelstudio` family plus `mediaUnderstandingProviders` (image+video) and `videoGenerationProviders`. 4 auth methods across China/Global × Standard/Coding Plan tiers. No dependents. |
| 39 | `sglang` | 🟢 | ✅ | 🟡 | Self-hosted SGLang inference server | Safe to disable — single chat provider (`sglang`) with `external: false` pricing, interactive custom auth (base URL + API key + model name). Late model discovery from server. Zero contracts. No dependents. Alternative: `vllm` for vLLM self-hosted servers. |
| 40 | `stepfun` | 🟢 | ✅ | 🟡 | StepFun LLM (standard + plan tiers) | Safe to disable — two providers (`stepfun`, `stepfun-plan`) with paired catalog resolution, 4 auth methods, `autoEnableWhenConfiguredProviders`. Zero contracts. No dependents. No alternative. |
| 41 | `synthetic` | 🟢 | ✅ | 🟡 | Synthetic AI (Anthropic-compatible multi-model) | Safe to disable — single chat provider (`synthetic`), Anthropic-compatible. Zero contracts. Environment variable: `SYNTHETIC_API_KEY`. No dependents. Alternative: `anthropic` for native Claude models. |
| 42 | `together` | 🟢 | ✅ | 🟡 | Together AI LLM (7 static models) + video generation | Safe to disable — single chat provider (`together`) with 7 static models plus `videoGenerationProviders` contract. Custom `classifyFailoverReason` for concurrency-limit detection. No dependents. |
| 43 | `venice` | 🟢 | ✅ | 🟡 | Venice AI (privacy-focused, uncensored) | Safe to disable — single chat provider (`venice`) with Grok model compat patches and DeepSeek V4 custom stream wrapper. Zero contracts. No dependents. No alternative provides same privacy-focused uncensored model access. |
| 44 | `vercel-ai-gateway` | 🟢 | ✅ | 🟡 | Vercel AI Gateway (multi-provider proxy) | Safe to disable — single chat provider (`vercel-ai-gateway`) with Claude model ID aliases (e.g., `opus-4.6` → `claude-opus-4-6`). Passthrough pricing. `resolveThinkingProfile`. Zero contracts. No dependents. Alternative: `cloudflare-ai-gateway`. |
| 45 | `vllm` | 🟢 | ✅ | 🟡 | Self-hosted vLLM inference server | Safe to disable — single chat provider (`vllm`) with `external: false` pricing, interactive custom auth, late model discovery, custom stream wrapper, `buildUnknownModelHint`. Zero contracts. No dependents. Alternative: `sglang` for similar self-hosted. |
| 46 | `volcengine` | 🟢 | ✅ | 🟡 | Volcano Engine Doubao LLM (standard + coding) + TTS | Safe to disable — two providers (`volcengine`, `volcengine-plan`) with 10+ static models plus `speechProviders` contract (TTS with separate `VOLCENGINE_TTS_*` env vars). No dependents. Alternative: `byteplus` for related ByteDance speech. |
| 47 | `vydra` | 🟢 | ✅ | 🟡 | Vydra media-only: image gen + video gen + speech | Safe to disable — single `vydra` provider plus three contracts: `speechProviders` (TTS), `imageGenerationProviders`, `videoGenerationProviders`. No text/LLM at all — pure media generation. No dependents. Alternatives: `xai` covers speech/image/video but different provider. |
| 48 | `xai` | 🟢 | ✅ | 🔵 | xAI Grok: LLM + 6 contracts + `x_search` + `code_execution` tools | **Richest single plugin by features** — disable only if xAI unused. Registers `xai` chat provider plus SIX contracts: `webSearchProviders` (as `grok`), `videoGenerationProviders`, `mediaUnderstandingProviders` (audio STT), `speechProviders`, `realtimeTranscriptionProviders`, `imageGenerationProviders`. Also TWO tools: `code_execution` (sandbox Python) and `x_search` (X post search), both lazy-loaded. Depends on `@mariozechner/pi-ai` + `typebox`. No dependents. No single alternative matches breadth. |
| 49 | `xiaomi` | 🟢 | ✅ | 🟡 | Xiaomi MiMo AI (flash/pro/omni) + TTS | Safe to disable — single chat provider (`xiaomi`) with static 3-model catalog plus `speechProviders` contract. Usage tracking via `resolveUsageAuth`/`fetchUsageSnapshot`. No dependents. No alternative. |
| 50 | `zai` | 🟢 | ✅ | 🟡 | Z.AI GLM series (4 endpoints) + media understanding | Safe to disable — single chat provider (`zai`, aliases `z-ai`/`z.ai`) with 5 auth methods, GLM-5 forward-compat resolution, plus `mediaUnderstandingProviders` (image via `glm-4.6v`, auto-priority 60). No dependents. Alternative: `together` and `volcengine` for GLM. |
| 51 | `tencent` | 🟢 | ✅ | 🟡 | Tencent Cloud TokenHub (Hy3 model) | Safe to disable — single chat provider (`tencent-tokenhub`) with 1 static model (`hy3-preview`, tiered pricing). Zero contracts. No dependents. No alternative. |

---

## 2. Messaging Channel Plugins (24 plugins)

### Batch A

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 52 | `bluebubbles` | 🟢 | ❌ | 🟡 | iMessage via BlueBubbles macOS REST API | Safe to disable — registers only the `bluebubbles` channel. Declares `preferOver: ["imessage"]` metadata (soft preference, not a hard dependency). Zero runtime deps. If disabled, the BlueBubbles channel disappears. `imessage` is the experimental alternative but BlueBubbles is preferred. |
| 53 | `discord` | 🟢 | ❌ | 🟡 | Full Discord bot (voice, slash commands, subagent hooks) | Safe to disable — the richest channel plugin. Registers `discord` channel with `nativeCommandsAutoEnabled`, `nativeSkillsAutoEnabled`, subagent hooks (`subagent_spawning`, `subagent_ended`, `subagent_delivery_target`), account inspection, doctor contracts, security audit, and channel directory resolution. Heavy deps avoided: `@discordjs/voice`, `opusscript` (audio codec), `discord-api-types`, `ws`. No alternative. |
| 54 | `feishu` | 🟢 | ❌ | 🟡 | Feishu/Lark enterprise chat + 6 tool suites | Safe to disable — registers `feishu` channel, six tool families (Doc, Chat, Wiki, Drive, Perm, Bitable), four bundled skills, and subagent hooks. Community-maintained. Heavy dep avoided: `@larksuiteoapi/node-sdk`. No alternative. |
| 55 | `googlechat` | 🟢 | ❌ | 🟡 | Google Workspace Chat via HTTP webhook + REST | Safe to disable — registers `googlechat` channel with Google Chat webhook receiver. `doctorCapabilities` for DM/group routing. Heavy deps avoided: `gaxios`, `google-auth-library`. No alternative. |
| 56 | `imessage` | 🟢 | ❌ | 🟡 | Native macOS iMessage (reads local chat.db) | Safe to disable — marked "work in progress" and preempted by BlueBubbles (`preferOver`). Zero runtime deps. No practical impact since BlueBubbles is preferred. Alternative: `bluebubbles`. |
| 57 | `irc` | 🟢 | ❌ | 🟡 | Classic IRC client (raw TCP/TLS, NickServ) | Safe to disable — registers `irc` channel. Pure Node.js `net`/`tls` implementation. Zero runtime deps. No alternative. |
| 58 | `line` | 🟢 | ❌ | 🟡 | LINE Messaging API + `/card` rich card command | Safe to disable — registers `line` channel and one command: `card` (rich card message sender via LINE Flex Messages). Dep avoided: `@line/bot-sdk`. No alternative. |
| 59 | `matrix` | 🟢 | ❌ | 🟡 | Matrix homeserver + E2E encryption (WASM crypto) | Safe to disable — registers `matrix` channel, three gateway methods (`matrix.verify.recoveryKey`, `matrix.verify.bootstrap`, `matrix.verify.status`), subagent hooks, and CLI metadata. **Heaviest dependency footprint**: WASM crypto, `fake-indexeddb`, full `matrix-js-sdk`. Disabling avoids all of this. No alternative. |
| 60 | `mattermost` | 🟢 | ❌ | 🟡 | Mattermost chat via WebSocket + slash commands | Safe to disable — registers `mattermost` channel and slash command HTTP route. Dep avoided: `ws` (very light). No alternative. |
| 61 | `msteams` | 🟢 | ❌ | 🟡 | Microsoft Teams bot (Azure AD, Express, JWT) | Safe to disable — registers `msteams` channel. **Second heaviest channel plugin**: `@azure/identity` (Azure auth), `@microsoft/teams.api` + `@microsoft/teams.apps`, `express`, `jsonwebtoken` + `jwks-rsa`. Disabling avoids the full Azure/Teams SDK stack. No alternative. |
| 62 | `nextcloud-talk` | 🟢 | ❌ | 🟡 | Nextcloud Talk via webhook bots | Safe to disable — registers `nextcloud-talk` channel. Only `zod` dependency. No alternative. |
| 63 | `nostr` | 🟢 | ❌ | 🟡 | Nostr NIP-04 encrypted DMs + profile HTTP route | Safe to disable — registers `nostr` channel and `POST /api/channels/nostr` HTTP route (gateway-auth, trusted-operator) for profile management. Deps avoided: `nostr-tools`. No alternative. |

### Batch B

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 64 | `qqbot` | 🟢 | ✅ | 🟡 | QQ Bot (Tencent) messaging + silk-wasm voice codec | Safe to disable — registers `qqbot` channel and skills from `./skills`. Deps avoided: `@tencent-connect/qqbot-connector`, `silk-wasm` (voice codec WASM), `mpg123-decoder`. Alternatives: Telegram, Discord for bot messaging. |
| 65 | `signal` | 🟢 | ❌ | 🟡 | Signal via signal-cli REST bridge | Safe to disable — registers `signal` channel. Zero runtime deps (requires external `signal-cli` process). No alternative. |
| 66 | `slack` | 🟢 | ❌ | 🟡 | Slack workspace bot (Socket Mode, HTTP routes) | Safe to disable — registers `slack` channel, `channelSecrets`, `inspectSlackReadOnlyAccount`, and HTTP routes via `registerFull`. Native commands/skills NOT auto-enabled. Deps avoided: `@slack/bolt`, `@slack/web-api`. Alternative: Discord. |
| 67 | `synology-chat` | 🟢 | ❌ | 🟡 | Synology Chat NAS-based messaging | Safe to disable — registers `synology-chat` channel. Only `zod` dependency. No alternative. |
| 68 | `telegram` | 🟢 | ❌ | 🟡 | Telegram bot (BotFather) with native commands/skills | Safe to disable — registers `telegram` channel with `nativeCommandsAutoEnabled`, `nativeSkillsAutoEnabled`, `channelSecrets`, and `inspectTelegramReadOnlyAccount`. Deps avoided: `grammy`, `@grammyjs/runner`, `@grammyjs/transformer-throttler`. Alternatives: WhatsApp, Discord, Slack. |
| 69 | `tlon` | 🟢 | ❌ | 🟡 | Tlon/Urbit decentralized messaging + `tlon` tool | Safe to disable — registers `tlon` channel, bundled skills, and a `tlon` tool (CLI wrapper with Urbit subcommands). **Heavy deps avoided**: `@aws-sdk/client-s3` (~60MB), `@aws-sdk/s3-request-presigner`, `@tloncorp/tlon-skill`, `@urbit/aura`. No alternative. |
| 70 | `twitch` | 🟢 | ❌ | 🟡 | Twitch chat via IRC-based Twurple | Safe to disable — registers `twitch` channel (alias `twitch-chat`). Deps avoided: `@twurple/api`, `@twurple/auth`, `@twurple/chat`. No alternative. |
| 71 | `whatsapp` | 🟢 | ❌ | 🟡 | WhatsApp via Baileys QR linking (real phone) | Safe to disable — registers `whatsapp` channel with plugin hooks (`messageReceived` broadcast) and persisted auth state. **Heavy deps avoided**: `@whiskeysockets/baileys` (~15MB+ with native builds), `jimp` (~5MB). Alternative: Telegram for bot messaging without QR pairing. |
| 72 | `zalo` | 🟢 | ❌ | 🟡 | Zalo Bot API (Vietnam) via webhook | Safe to disable — registers `zalo` channel (alias `zl`) and `channelSecrets`. Dep avoided: minimal (`undici`). Alternative: `zalouser` for personal account Zalo integration. |
| 73 | `zalouser` | 🟢 | ❌ | 🟡 | Zalo personal account via zca-js QR login | Safe to disable — registers `zalouser` channel (alias `zlu`) and `zalouser` tool. Deps avoided: `zca-js`, `typebox`. Alternative: `zalo` for official Bot API integration. |
| 74 | `google-meet` | 🔴 | ❌ | 🔵 | Google Meet AI participant (Chrome/Twilio/voice-call) + `/googlemeet` CLI | **Keep disabled by default** (already is). This is a tool/action plugin, NOT a channel plugin. Registers: `google_meet` tool (16 actions), 12 gateway methods, `googlemeet.chrome` node host command, and `googlemeet` CLI subcommand. `enabledByDefault: false` but loads on startup. If disabled, the `google_meet` tool returns "Google Meet plugin disabled" errors. Runtime requirements: Chrome browser, SoX audio, BlackHole driver, Google OAuth tokens, realtime model provider. No alternative for automated Meet participation. |

---

## 3. Memory Plugins (3 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 75 | `memory-core` | 🟢 | ❌ | ⚠️ | **Core memory infra**: `memory_search`, `memory_get`, local embedding, dreaming, flush, CLI | **DO NOT DISABLE** — hardcoded as default memory slot in `src/plugins/slots.ts`. Provides `kind: memory` with tools `memory_search` and `memory_get`, `registerMemoryCapability` pipeline (prompt builder, flush, runtime, public artifacts), built-in `local` embedding provider (conditionally requiring heavy optional `node-llama-cpp@3.18.1`), dreaming engine, and CLI `memory`. If disabled without an alternative memory slot plugin (`plugins.slots.memory = "memory-lancedb"`), ALL agents lose memory search/retrieval — tools vanish, flush fails, dreaming stops. `active-memory` sub-agent silently degrades (returns `NONE`). Alternative: `memory-lancedb` for vector-based memory with different tools (`memory_recall`, `memory_store`, `memory_forget`). |
| 76 | `memory-lancedb` | 🟢 | ❌ | 🔵 | LanceDB vector memory with auto-capture/recall | Disable only if another memory plugin fills the slot. Provides `kind: memory` with tools `memory_recall`, `memory_store`, `memory_forget`, CLI `ltm`, and hooks: `before_prompt_build` (auto-recall), `agent_end` (auto-capture), `session_end`. Depends on `@lancedb/lancedb` (heavy native vector DB) and `openai` (embeddings SDK). Self-disables if no embedding provider configured. If disabled, vector-based memory tools and auto-capture/recall stop; `active-memory` gracefully degrades. Disabling frees significant native binary footprint. Alternative: `memory-core` for file-backed memory. |
| 77 | `memory-wiki` | 🔴 | ❌ | 🟡 | Obsidian-friendly wiki vault + tools (`wiki_search`, `wiki_get`, etc.) | Safe to disable — provides memory prompt/corpus supplements, tools `wiki_status`, `wiki_lint`, `wiki_apply`, `wiki_search`, `wiki_get`, CLI `wiki`, and gateway methods. Does NOT occupy the `memory` slot — complements whatever memory plugin is active. Can optionally bridge from `memory-core` via `bridge.enabled` and `unsafeLocal.allowPrivateMemoryCoreAccess`. Loads at startup (`onStartup: true`) but only `typebox` + `yaml` deps. If disabled, wiki tools vanish but no other plugin breaks. No alternative. |

---

## 4. Speech/Audio/TTS Plugins (8 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 78 | `azure-speech` | 🟢 | ✅ | 🟡 | Azure AI Speech TTS (MP3, Ogg/Opus, PCM) | Safe to disable — pure `speechProviders` contract (`azure-speech`, `azure`). `enabledByDefault: true`, `onStartup: false`. Zero production deps. If disabled, any `talk.provider = "azure-speech"` fails but system falls back to any other registered speech provider. Alternatives: `microsoft` (free Edge TTS), `elevenlabs` (higher quality). |
| 79 | `deepgram` | 🟢 | ✅ | 🟡 | Deepgram audio transcription + realtime | Safe to disable — dual contract: `mediaUnderstandingProviders` (audio via `nova-3`, priority 30) and `realtimeTranscriptionProviders`. If disabled, audio transcription and realtime voice-call transcription lose this provider; elevenlabs covers both at higher priority (45). No deps. |
| 80 | `elevenlabs` | 🟢 | ✅ | 🟡 | ElevenLabs TTS + audio transcription + realtime | Safe to disable — **broadest speech plugin** with three contracts: `speechProviders`, `mediaUnderstandingProviders` (audio via `scribe_v2`, highest auto-priority 45), `realtimeTranscriptionProviders`. Also has legacy config migrations. If disabled, the highest-priority transcription provider disappears; `voice-call` may lose preferred streaming backend. Alternatives: `azure-speech`/`microsoft` for TTS; `deepgram` for transcription. |
| 81 | `inworld` | 🟢 | ✅ | 🟡 | Inworld AI streaming TTS | Safe to disable — pure `speechProviders` contract (`inworld`). Streaming TTS with configurable voice/model/temperature. If disabled, Inworld TTS disappears. Alternatives: `microsoft`, `azure-speech`, `elevenlabs`, `tts-local-cli`. |
| 82 | `microsoft` | 🟢 | ✅ | 🟡 | Microsoft Edge TTS (**free, no key**) | Safe to disable — pure `speechProviders` contract (`microsoft`) using `node-edge-tts`. The only zero-config free TTS fallback. If disabled, `talk.provider = "microsoft"` fails and `/voice` commands break if Microsoft is active. `voice-call` explicitly ignores Microsoft for telephony. Dep freed: `node-edge-tts`. Alternatives: `azure-speech` (paid, same vendor), `tts-local-cli` (free local CLI). |
| 83 | `senseaudio` | 🟢 | ✅ | 🟡 | SenseAudio audio transcription | Safe to disable — single `mediaUnderstandingProviders` contract (`senseaudio`). If disabled, audio transcription via SenseAudio fails; `deepgram` and `elevenlabs` provide same contract. |
| 84 | `talk-voice` | 🔴 | ✅ | 🟡 | `/voice` slash command for voice selection | Safe to disable — pure control-surface plugin. Registers `/voice` command (status/list/set) using `api.runtime.tts.listVoices` and `api.runtime.config.replaceConfigFile`. No speech providers registered — only commands. `onStartup: true` but zero deps (248 lines). If disabled, users lose the ability to list/change Talk voices via chat; must edit config manually. All speech providers continue working. |
| 85 | `tts-local-cli` | 🟢 | ✅ | 🟡 | Local CLI TTS (invokes `say`, `espeak`, `festival`) | Safe to disable — pure `speechProviders` contract (`tts-local-cli`, `cli`). Shells out to a system TTS command. Zero production deps. If disabled, local CLI TTS fails. Alternatives: `microsoft` (free edge TTS), `azure-speech`/`elevenlabs` (paid). |

---

## 5. Web Search / Fetch / Extraction Plugins (8 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 86 | `brave` | 🟢 | ❌ | 🟡 | Brave Search API web search | Safe to disable — single `webSearchProviders` contract (`brave`). Only `typebox` dependency. If disabled, Brave search is unavailable but switching to any other web search provider restores function. Alternatives: `duckduckgo` (free), `searxng` (self-hosted), `tavily`, `firecrawl`, `perplexity`, `exa`. |
| 87 | `duckduckgo` | 🟢 | ❌ | 🟡 | DuckDuckGo Instant Answer API (**free, no key**) | Safe to disable but **this is the only free/no-key web search**. Single `webSearchProviders` contract (`duckduckgo`). Zero deps, no API key. If disabled AND no other provider (especially if no API-keyed provider is configured), web search becomes completely unavailable. Consider keeping as last-resort fallback. Alternative: `searxng` (self-hosted, also free). |
| 88 | `exa` | 🟢 | ❌ | 🟡 | Exa Search API | Safe to disable — single `webSearchProviders` contract (`exa`). No deps. Alternatives: `duckduckgo`, `brave`, `searxng`, `tavily`, `firecrawl`, `perplexity`. |
| 89 | `firecrawl` | 🟢 | ❌ | 🔵 | Firecrawl web search + scraping + fetch + tools | More impactful than single-contract providers. Registers `webSearchProviders` contract, `webFetchProviders` contract (the ONLY bundled `webFetchProvider`), and two tools: `firecrawl_search`, `firecrawl_scrape`. If disabled, those two tools vanish, web search loses this provider, and web fetch falls back to core's native fetch (which is the default). No other plugin depends on it. Alternatives: many for search; core native fetch for web fetching. |
| 90 | `perplexity` | 🟢 | ❌ | 🟡 | Perplexity LLM-backed web search + OpenRouter fallback | Safe to disable — single `webSearchProviders` contract (`perplexity`). Routes via Perplexity or OpenRouter chat-completions API (Sonar models). No deps. Alternatives: many other search providers. |
| 91 | `searxng` | 🟢 | ❌ | 🟡 | Self-hosted SearXNG (privacy-respecting, no API key) | Safe to disable — single `webSearchProviders` contract (`searxng`). Zero deps, no API key needed. The only self-hosted/private search backend. If disabled, operators who want search without third-party APIs lose that capability. Alternatives: `duckduckgo` (free, no self-host needed). |
| 92 | `tavily` | 🟢 | ❌ | 🔵 | Tavily AI-optimized search + content extraction + tools | Registers `webSearchProviders` contract and two tools: `tavily_search`, `tavily_extract`, plus bundled skills. If disabled, those two tools and skills vanish. No dependents. Alternatives: `duckduckgo`, `brave`, `searxng`, `firecrawl`, `perplexity`, `exa`. |
| 93 | `voyage` | 🟢 | ✅ | 🔵 | Voyage AI memory embedding provider | Disable only if another `memoryEmbeddingProvider` is enabled. Registers `memoryEmbeddingProviders` contract (`voyage`, `autoSelectPriority: 40`). If disabled AND no other embedding provider available, the memory vector store fails — `memory_search`/`memory_recall`/dreaming break with missing embedding errors. 9+ alternatives: `openai`, `ollama`, `mistral`, `google` (gemini), `lmstudio`, `github-copilot`, `deepinfra`, `amazon-bedrock`, `memory-core` (local). |

---

## 6. Tool / Utility Plugins (14 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 94 | `browser` | 🔴 | ✅ | 🔵 | Full browser automation (Chrome/CDP) via Playwright | **One of the heaviest** — `onStartup: true`. Registers: `browser` tool, `browser` CLI subcommands, `browser.request` gateway method (operator.admin), `browser-control` service, `browser.proxy` node host command, security audit, and skills. Depends on `playwright-core@1.59.1` (downloads Chromium ~300MB), `express`, `ws`, `commander`, `@modelcontextprotocol/sdk`. If disabled, agents cannot navigate pages, take screenshots, or interact with web UIs. The `diffs` plugin does NOT depend on it (launches its own Chromium). Disabling frees significant resources. No alternative. |
| 95 | `device-pair` | 🔴 | ✅ | 🟡 | QR/paste-code device pairing + `/pair` command | Safe to disable — registers `/pair` slash command and pairing notifier service. Zero deps, `onStartup: true` but extremely lightweight. If disabled, users cannot generate setup codes or approve device pairing via commands; must use manual IP:port entry for mobile apps. The bootstrapping API itself is in core SDK — only the user-facing command disappears. No alternative. |
| 96 | `diffs` | 🔴 | ❌ | 🔵 | Read-only diff viewer + file-to-PNG/PDF renderer | `onStartup: true`, not enabled by default. Registers: `diffs` tool, HTTP route `/plugins/diffs`, and `before_prompt_build` hook (injects diff guidance into agent prompts). Depends on `@pierre/diffs`, `@pierre/theme`, `playwright-core` (separate Chromium launch). If disabled, agents cannot render or view diffs via this tool. Frees Playwright weight if `browser` also disabled. No alternative. |
| 97 | `document-extract` | 🟢 | ✅ | 🔵 | Extracts text + fallback images from PDF attachments | **Keep enabled** unless PDF ingestion is unused. Registers `documentExtractors` contract `pdf`. Core document extraction auto-discovers and calls this; when enabled AND `tools.web.fetch.readability: true`, every HTML web fetch goes through it first. If disabled, PDF text extraction stops, web fetch may get lower-quality raw HTML. Deps: `pdfjs-dist`, optional `@napi-rs/canvas`. No alternative. |
| 98 | `file-transfer` | 🔴 | ✅ | 🟡 | File fetch/list/write on paired nodes (base64, 16MB) | Safe to disable — registers 4 agent tools (`file_fetch`, `dir_list`, `dir_fetch`, `file_write`) and 4 node host commands. If disabled, agents fall back to `exec`/bash for file operations without error, but file writes >16KB may hit stdout truncation. Dep: `minimatch` (lightweight). No alternative. |
| 99 | `llm-task` | 🔴 | ❌ | 🟡 | Generic JSON-only LLM subtask tool | Safe to disable — single optional tool (`llm_task`) for structured JSON subtask execution via configurable provider/model with JSON Schema validation. If disabled, TaskFlow workflows that explicitly call it fail. No other agent workflows are affected. Deps: `ajv`, `typebox`. No alternative. |
| 100 | `lobster` | 🔴 | ❌ | 🟡 | Typed workflow tool with resumable approvals | Safe to disable — single optional tool (`lobster_task`) via `@clawdbot/lobster` for typed pipeline execution with approval gating and state persistence. If disabled, TaskFlow workflows that call it fail. Deps: `@clawdbot/lobster`, `ajv`, `typebox`. No alternative. |
| 101 | `open-prose` | 🟢 | ❌ | 🟡 | VM skill pack + `/prose` slash command | Safe to disable — empty `register()`, works entirely through manifest-declared skills and command routing. Zero runtime deps. If disabled, OpenProse skills and `/prose` command disappear. No dependents. No alternative. |
| 102 | `phone-control` | 🔴 | ✅ | 🔵 | Arm/disarm high-risk phone node commands with auto-expiry | Safe to disable if phone nodes not used. Registers `/phone` slash command and `phone-control-expiry` service (15s interval auto-disarm). Zero deps. If disabled, `/phone` command disappears and auto-expiry stops — phone commands stay at whatever allow/deny state was last configured. No alternative; fallback is manual config editing. |
| 103 | `skill-workshop` | 🔴 | ✅* | 🟡 | Auto-captures agent workflows as reusable skills | Safe to disable — registers `skill_workshop` tool, `before_prompt_build` hook (guidance injection), `agent_end` hook (post-turn heuristic/LLM review). Captures user corrections and workflow patterns, queues pending suggestions or auto-applies. If disabled, automatic skill capture stops but existing skills remain on disk. Dep: `typebox`. No alternative; manual skill editing is fallback. |
| 104 | `thread-ownership` | 🔴 | ❌ | 🟡 | Prevents multi-agent collision in same Slack thread | Safe to disable — registers `message_received` and `message_sending` hooks, active only when `channelId === "slack"`. Posts ownership claims to a `slack-forwarder` HTTP API. If disabled, multi-agent thread coordination stops — multiple agents may reply in same thread. Only relevant in multi-agent Slack deployments. Zero deps. No alternative. |
| 105 | `tokenjuice` | 🟢 | ❌ | 🟡 | Compacts exec/bash tool results for pi/codex runtimes | Safe to disable — registers agent tool result middleware targeting `["pi", "codex"]` runtimes. Uses `tokenjuice` reducers to shrink large outputs. If disabled, exec/bash results pass through uncompacted — may cause earlier context window exhaustion but no hard errors. Dep: `tokenjuice`. No alternative. |
| 106 | `web-readability` | 🟢 | ✅ | ⚠️ | Mozilla Readability HTML→article text extractor | **Do NOT disable** — this is the primary web content quality path. Registers `webContentExtractors` contract `readability` (loaded as public artifact, `autoDetectOrder: 10`). Core `web-fetch.ts` calls this first for every HTML web fetch when `tools.web.fetch.readability: true` (default). If disabled, web fetch delivers raw HTML or lower-quality content. Deps: `@mozilla/readability`, `linkedom` (lazily loaded). No alternative bundled. |
| 107 | `webhooks` | 🔴 | ❌ | 🔵 | Authenticated inbound HTTP webhooks → TaskFlows | Disable only if no external automation integration. Registers HTTP routes with plugin-level auth + HMAC secret verification per route. If disabled, all configured webhook endpoints return 404 — any CI, GitHub, or monitoring automation pushing events into OpenClaw silently breaks. Dep: `zod`. No alternative; fallback is gateway-level HTTP route configuration. |

---

## 7. Sandbox / Shell Plugins (2 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 108 | `acpx` | 🔴 | ✅ | ⚠️ | Embedded ACP runtime: Claude/Codex agent backends + MCP bridging | **DO NOT DISABLE** if any ACP agent is used. Sole provider of the `acpx` ACP runtime backend — registers the `acpx-runtime` deferred service providing the full ACP runtime interface (`ensureSession`, `runTurn`, `getCapabilities`, `getStatus`, `setMode`, `probeAvailability`, `isHealthy`, etc.), `reply_dispatch` hook, ACP router skills, and MCP server bridging (plugin-tools + core-tools). If disabled, ALL ACP sessions fail with: `runtime="acp" is unavailable in this session because no ACP runtime backend is loaded. Enable the acpx plugin or use runtime="subagent"`. Breaks Claude Code via ACP, Gemini CLI via ACP, Codex-ACP, and OpenCode ACP. Deps: `@agentclientprotocol/claude-agent-acp`, `@zed-industries/codex-acp`, `acpx`. Fallback: `runtime="subagent"` (built-in process spawner). |
| 109 | `openshell` | 🔴 | ❌ | 🔵 | OpenShell sandbox backend (mirrored workspaces + SSH) | Disable only if no agent uses the `openshell` sandbox backend. Registers `openshell` sandbox factory + manager via `registerSandboxBackend("openshell", ...)`. If disabled, agents configured with `agents.defaults.sandbox.backend=openshell` fail with: `Sandbox backend "openshell" is not registered. Load the plugin that provides it, or set agents.defaults.sandbox.backend=docker.` Built-in `docker` and `ssh` sandbox backends remain available. Dep: `openshell`. |

---

## 8. Migration Plugins (2 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 110 | `migrate-claude` | 🟢 | ❌ | 🟡 | Imports Claude Code/Desktop config, MCP servers, skills | Safe to disable — one-shot migration utility. Registers `migrationProviders` contract `claude`. Lazily activated (`onStartup: false`), zero runtime deps. If disabled, `openclaw migrate --from claude` fails with "no migration provider for claude." Manual config transfer is the alternative. |
| 111 | `migrate-hermes` | 🟢 | ❌ | 🟡 | Imports Hermes config, memories, skills, credentials | Safe to disable — one-shot migration utility. Registers `migrationProviders` contract `hermes`. Lazily activated, depends on `yaml`. If disabled, `openclaw migrate --from hermes` fails. Manual config transfer is the alternative. |

---

## 9. Diagnostics Plugins (2 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 112 | `diagnostics-otel` | 🔴 | ❌* | 🟡 | Export traces/metrics/logs to OpenTelemetry collector via OTLP | **Recommended to disable unless you have an OTel collector.** Activates at startup but only exports when explicitly enabled in config. Pulls in 11 `@opentelemetry/*` packages totaling dozens of MB. If disabled, all OTel export stops — no traces/metrics/logs sent, but system operates normally. Disabling frees substantial install size and memory. `diagnostics-prometheus` provides metrics alternative. |
| 113 | `diagnostics-prometheus` | 🔴 | ❌* | 🟡 | Prometheus text-format metrics at `/api/diagnostics/prometheus` | Safe to disable — registers a service + HTTP route. **Zero external deps** — implements Prometheus format in-process. If disabled, `/api/diagnostics/prometheus` returns 404, Prometheus scrapers fail. `diagnostics-otel` provides alternative export path. |

---

## 10. Infrastructure / Gateway Plugins (2 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 114 | `bonjour` | 🔴 | ✅ | 🟡 | Bonjour/mDNS gateway auto-discovery for mobile apps | Safe to disable — registers `gatewayDiscoveryService` `bonjour` advertising gateway over mDNS. Depends on `@homebridge/ciao`. If disabled, mobile apps cannot auto-discover the gateway on LAN; manual IP:port entry required. No other plugin depends on it. Disabling frees `@homebridge/ciao` dep. Alternative: Tailscale DNS or manual connection. |
| 115 | `voice-call` | 🔴 | ✅* | 🟡 | Phone calls via Twilio/Telnyx/Plivo + `voice_call` tool | Safe to disable — registers 12 gateway methods (`voicecall.*`), `voice_call` tool (6 actions), `voicecall` CLI, and voice-call runtime service with webhook server. Code defaults `enabled: true` with `mock` provider when unconfigured (low overhead). Deps: `commander`, `typebox`, `ws`. If disabled, all telephony features gone — gateway methods fail, `voice_call` tool errors. The `google-meet` plugin provides alternative voice conversation (via Google Meet). |

---

## 11. Agent Support Plugins (1 plugin)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 116 | `active-memory` | 🔴 | ✅* | 🟡 | Runs blocking memory sub-agent before replies, injects context | Safe to disable — registers `before_prompt_build` hook (runs blocking memory sub-agent, injects `🧩 Active Memory:` context), and `/active-memory` slash command (on/off/status per-session and global). Zero external deps. Loads at startup but only fires for allowed agents in direct messages (configurable). If disabled, memory context stops being auto-injected into prompts, conversations lose personalization. Does SOME depend on memory provider plugins (`memory-core`/`memory-lancedb`) to retrieve memories — without them, consistently returns `NONE`. `memory-wiki` provides alternative knowledge surface but different mechanics. |

---

## 12. Media Generation Plugins (1 plugin)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 117 | `runway` | 🟢 | ✅ | 🟡 | Runway API video generation | Safe to disable — single `videoGenerationProviders` contract (`runway`). Lazily activated, zero external deps beyond SDK. If disabled, Runway video generation fails. Alternative: `comfy` for ComfyUI-based video generation. |

---

## 13. QA / Testing Plugins (3 plugins)

| # | Plugin | Ld | Def | Rating | What It Does | Disable Notes |
|---|--------|-----|-----|--------|--------------|---------------|
| 118 | `qa-channel` | 🟢 | ❌ | 🟡 | Synthetic Slack-class channel for QA scenarios | Safe to disable — bundled channel entry providing synthetic transport for automated QA. Hidden from normal setup (`exposure: { configured: false, setup: false, docs: false }`). Dep: `typebox`. Only `qa-lab` depends on it indirectly. No production impact. |
| 119 | `qa-lab` | 🟢 | ❌ | 🟡 | QA harness with scenario runner + debugger UI | Safe to disable — registers `qa` CLI with subcommands. Deps: `@copilotkit/aimock`, `@modelcontextprotocol/sdk`, `playwright-core`, `yaml`, `zod`. Internal QA tooling only — no production features break. No dependents except `qa-channel` (as transport under test). |
| 120 | `qa-matrix` | 🟢 | ❌ | 🟡 | Docker-backed Matrix live QA lane (disposable Synapse) | Safe to disable — empty `register()`, manifest-declared `qaRunner` for Matrix QA. Deps: `undici`. Requires Docker for disposable Synapse homeserver. No production impact. |

---

## Summary Statistics

### By Load Behavior
| Load Type | Count |
|-----------|-------|
| 🔴 Startup (`onStartup: true`) | **22** |
| 🟢 Lazy (on demand) | **98** |

### By Default State
| Default | Count |
|---------|-------|
| ✅ Enabled by default | **67** |
| ❌ Not enabled by default | **53** |

### By Disable Safety Rating
| Rating | Count | Meaning |
|--------|-------|---------|
| 🟡 Safe to disable if unused | **90** | Entirely optional for most users |
| 🔵 Only if no config referencing it | **26** | Check your config before disabling |
| ⚠️ Do NOT disable | **4** | `openai`, `memory-core`, `acpx`, `web-readability` |

### Startup Plugins (22 — impact cold-boot latency)
`memory-wiki`, `google-meet`, `browser`, `device-pair`, `diffs`, `file-transfer`, `llm-task`, `lobster`, `phone-control`, `skill-workshop`, `thread-ownership`, `webhooks`, `acpx`, `openshell`, `diagnostics-otel`, `diagnostics-prometheus`, `bonjour`, `voice-call`, `active-memory`, `talk-voice`

### Heaviest npm Dependencies (by weight)
| Plugin | Heavy Deps |
|--------|------------|
| `matrix` | WASM crypto + fake-indexeddb + matrix-sdk |
| `msteams` | Azure Identity SDK + Express + JWT |
| `whatsapp` | @whiskeysockets/baileys (~15MB+) + jimp |
| `google` | @google/genai (large SDK) + @mariozechner/pi-ai |
| `diagnostics-otel` | 11 @opentelemetry/* packages |
| `tlon` | AWS S3 SDK (~60MB) |
| `browser` | playwright-core (downloads Chromium ~300MB) + express |
| `memory-lancedb` | @lancedb/lancedb (native vector DB binary) |

---

## Action Recommendations for ThirdClaw Operations

1. **Leave `hosted-minimal` as default** — `amazon-bedrock`, `amazon-bedrock-mantle`, `anthropic-vertex`, `codex` are the right 4 to disable.

2. **NEVER disable these 4**: `openai` (core model + CLI infra), `memory-core` (all memory ops), `acpx` (ACP agent backends), `web-readability` (primary web content quality path).

3. **Consider disabling these for faster startup**: `diagnostics-otel` (11 heavy OTel packages, only needed with collector), `google-meet` (Chrome/SoX/BlackHole requirements), `diffs` (playwright-core weight), `llm-task`/`lobster` (TaskFlow-specific), `thread-ownership` (multi-agent Slack only), `openshell` (if using built-in docker/ssh sandbox).

4. **Keep `duckduckgo` as web search fallback** — it's the only zero-config, zero-key, zero-auth web search provider.

5. **Monitor runtime dep disk usage** — `matrix`, `whatsapp`, `browser`, `google`, `tlon` pull in large dependency trees auto-installed at startup.

6. **The `voice-call` plugin** defaults to `enabled: true` with `mock` provider on startup — minimal overhead but verify it doesn't allocate resources in `hosted-minimal`.
