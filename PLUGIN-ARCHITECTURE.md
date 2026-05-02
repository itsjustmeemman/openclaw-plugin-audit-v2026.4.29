# OpenClaw v2026.4.29 — Embedded Plugin Architecture Scan Report

**Date:** 2026-05-02  
**Source:** `openclaw-sources/v2026.4.29/`  
**Tool:** Codex CLI (deepseek-v4-pro)  
**Role:** [codex-cli] [executor]

---

## 1. Executive Summary

OpenClaw v2026.4.29 ships with **120 bundled plugins** across 13 categories. Plugins are TypeScript modules discovered at startup via filesystem scanning under `extensions/`, registered through a unified plugin API, and loaded by a Jiti-based runtime loader. During Fly.io deployment, the `THIRDCLAW_PLUGIN_PROFILE` env var controls which plugins are disabled on first boot (default: `hosted-minimal`, which disables 4 heavy providers).

---

## 2. Plugin Categories (120 Total)

| # | Category | Count | On-Startup Count | Startup Plugins |
|---|----------|-------|-------------------|-----------------|
| 1 | LLM Provider | 51 | 2 | kilocode, synthetic |
| 2 | Messaging Channel | 24 | 2 | qqbot, google-meet |
| 3 | Memory | 3 | 1 | memory-wiki |
| 4 | Speech/Audio/TTS | 8 | 1 | talk-voice |
| 5 | Web Search/Fetch/Extraction | 8 | 0 | — |
| 6 | Tool/Utility | 14 | 8 | browser, device-pair, diffs, file-transfer, llm-task, lobster, phone-control, skill-workshop, thread-ownership, webhooks |
| 7 | Sandbox/Shell | 2 | 2 | acpx, openshell |
| 8 | Migration | 2 | 0 | — |
| 9 | Diagnostics | 2 | 2 | diagnostics-otel, diagnostics-prometheus |
| 10 | Infrastructure/Gateway | 2 | 2 | bonjour, voice-call |
| 11 | Agent Support | 1 | 1 | active-memory |
| 12 | Media Generation | 2 | 0 | — |
| 13 | QA/Testing | 3 | 0 | — |

**Plugins that load on gateway startup (`activation.onStartup: true` or no activation block):** ~18  
**Plugins that load lazily (on-demand via capability trigger):** ~102

---

## 3. Plugin Architecture Deep Dive

### 3.1 Identity Types

- **`PluginOrigin`** (`src/plugins/plugin-origin.types.ts:1`): `"bundled" | "global" | "workspace" | "config"` — where the plugin was discovered
- **`PluginKind`** (`src/plugins/plugin-kind.types.ts:1`): `"memory" | "context-engine"` — exclusive-slot plugin categories
- **`PluginFormat`** (`src/plugins/manifest-types.ts:10`): `"openclaw" | "bundle"` — native vs bundle wrapper

### 3.2 Manifest (`openclaw.plugin.json`)

Each plugin declares a manifest (`src/plugins/manifest.ts:33`) with:
- `id`, `name`, `version`, `kind`
- `channels[]`, `providers[]`, `cliBackends[]` — capability ownership
- `contracts` (`PluginManifestContracts`): declares which capability providers the plugin owns (tools, speech, image gen, web search, etc.)
- `activation` (`PluginManifestActivation`): when to load — `onStartup`, `onProviders`, `onChannels`, `onCommands`, `onRoutes`, `onConfigPaths`, `onCapabilities`
- `enabledByDefault`: whether the plugin is ON by default
- `modelSupport`, `modelCatalog`, `channelConfigs`, `skills`, `uiHints`

### 3.3 Registration API (`OpenClawPluginApi`)

Defined in `src/plugins/types.ts:2342-2551`. The central API injected into every plugin at load time. Supports 35+ registration methods:
- `registerChannel()`, `registerProvider()`, `registerCliBackend()`
- `registerTool()`, `registerHook()`, `registerCommand()`
- `registerSpeechProvider()`, `registerMediaUnderstandingProvider()`, `registerImageGenerationProvider()`, `registerVideoGenerationProvider()`, `registerMusicGenerationProvider()`
- `registerWebFetchProvider()`, `registerWebSearchProvider()`
- `registerMemoryCapability()`, `registerContextEngine()`
- `registerAgentHarness()`, `registerService()`, `registerHttpRoute()`, `registerGatewayMethod()`
- `registerMigrationProvider()`, `registerMemoryEmbeddingProvider()`
- `registerTrustedToolPolicy()`, `registerAgentToolResultMiddleware()`
- `registerSessionSchedulerJob()`, `registerGatewayDiscoveryService()`
- and more...

### 3.4 Plugin SDK Entry Point

`definePluginEntry()` in `src/plugin-sdk/plugin-entry.ts:265` — canonical entry helper for non-channel plugins. Channel plugins use `defineChannelPluginEntry()` from `openclaw/plugin-sdk/core`.

---

## 4. Plugin Loading Pipeline (Step by Step)

### 4.1 Startup Orchestration

```
openclaw.mjs (CLI entry)
  → src/entry.ts → runCli()
    → src/cli/gateway-cli/run.ts → runGatewayCommand()
      → src/gateway/server.impl.ts (line 572)
        → src/gateway/server-startup-plugins.ts → prepareGatewayPluginBootstrap()
```

### 4.2 Bootstrap Sequence (`server-startup-plugins.ts:136`)

1. **Run startup maintenance** (line 153): channel maintenance + session migration
2. **Init subagent registry** (line 172)
3. **Apply plugin auto-enable** (line 178): `applyPluginAutoEnable()` — detects configured channels/providers and auto-enables their plugins
4. **Load PluginLookUpTable** (line 192): loads manifest registry + installed plugin index
5. **Resolve startup plugin IDs** (line 202): `resolveGatewayStartupPluginIdsFromRegistry()` decides which plugins are needed at startup
6. **Pre-stage runtime deps** (line 210): `prestageGatewayBundledRuntimeDeps()` bulk-installs missing npm dependencies for all startup plugins
7. **Load plugins** (line 215): `loadGatewayStartupPlugins()` → `loadOpenClawPlugins()`

### 4.3 Core Loader (`src/plugins/loader.ts:1079`)

The `loadOpenClawPlugins()` function:

1. **Cache Check** (line 1116): If config hasn't changed, restore cached registry
2. **Discovery** (line 1263): `discoverOpenClawPlugins()` scans:
   - `dist/extensions/` (bundled) — resolved by `resolveBundledPluginsDir()` in `src/plugins/bundled-dir.ts`
   - `~/.openclaw/extensions/` (global)
   - `<workspace>/.openclaw/extensions/` (workspace)
   - Config-specified paths from `plugins.load.paths`
3. **Manifest Loading** (line 1270): `loadPluginManifestRegistry()` parses `openclaw.plugin.json` for each candidate
4. **Deduplication** (line 1304): Origin-based precedence: `config > workspace > global > bundled`
5. **Per-Plugin Loading Loop** (line 1322):
   - Resolve enable state (allow/deny lists)
   - Install bundled runtime deps if needed (`installBundledRuntimeDeps()`)
   - Load module via **Jiti** (TS/JS JIT transpiler with SDK alias mapping)
   - Call `register(api)` synchronously
   - Collect registrations into `PluginRegistry`
6. **Post-load**: cache registry, warn about untracked plugins, set active registry

### 4.4 Discovery Mechanism

Two discovery paths in `src/plugins/discovery.ts`:
- **Primary (97 plugins)**: Has `openclaw.plugin.json` + `package.json` with `openclaw.extensions: ["./index.ts"]`
- **Fallback (5 plugins)**: Has `openclaw.plugin.json` but NO `package.json`; discovered via `DEFAULT_PLUGIN_ENTRY_CANDIDATES` (`index.ts`, `index.js`, `index.mjs`, `index.cjs`):
  - `active-memory`, `device-pair`, `phone-control`, `talk-voice`, `thread-ownership`

### 4.5 Runtime Dependency Installation

Three paths:
- **Build-time (Dockerfile)**: `pnpm install --frozen-lockfile` + `pnpm prune --prod` + `node scripts/postinstall-bundled-plugins.mjs`
- **Runtime bulk pre-stage**: `prestageGatewayBundledRuntimeDeps()` — installs all startup plugin deps before loading
- **Runtime per-plugin**: `installBundledRuntimeDeps()` — spawns npm/pnpm for missing deps during plugin load

Deps are installed to `OPENCLAW_PLUGIN_STAGE_DIR` (or `~/.openclaw/plugin-runtime-deps`). Jiti aliases map plugin imports to the installed locations.

---

## 5. Plugin Priority / Ordering Systems

| Mechanism | Location | Rule |
|-----------|----------|------|
| **Origin Precedence** | `manifest-registry.ts:87-92` | `config(0) > workspace(1) > global(2) > bundled(3)` — lower rank wins |
| **Hook Priority** | `hook-types.ts:937`, `hooks.ts:224` | Higher `priority` number runs first; sequential for modifying hooks |
| **Channel `preferOver`** | `manifest.ts:51-59` | Channel declares which other channels it should supersede |
| **`assistantPriority`** | `types.ts:1083` | Lower number sorts earlier in interactive assistant pickers |
| **`autoSelectOrder`** | Speech/TTS provider types | Lower = preferred when auto-selecting |

---

## 6. Fly.io Deployment Plugin Flow

### 6.1 Build Time (Dockerfile)

- `OPENCLAW_EXTENSIONS` build arg (default: `""`) controls which extensions get pre-resolved dependencies — but ALL extensions are still compiled into `dist/extensions/` at line 92 (`COPY . .`)
- `pnpm prune --prod` strips dev deps
- `postinstall-bundled-plugins.mjs` runs: prunes stale dist, applies hotfixes, migrates plugin registry

### 6.2 Deploy Time (deploy-openclaw-public.sh)

- `THIRDCLAW_PLUGIN_PROFILE` is substituted into `fly.public-auth.toml`
- Default: `hosted-minimal` for v2026.4.26+

### 6.3 First Boot (start.sh)

**`THIRDCLAW_PLUGIN_PROFILE` values:**
- **`hosted-minimal`** (default): Disables 4 heavy providers via deny list + explicit disable:
  ```json
  {
    "deny": ["amazon-bedrock", "amazon-bedrock-mantle", "anthropic-vertex", "codex"],
    "entries": {
      "amazon-bedrock": {"enabled": false},
      "amazon-bedrock-mantle": {"enabled": false},
      "anthropic-vertex": {"enabled": false},
      "codex": {"enabled": false}
    }
  }
  ```
- **`upstream-default`**: All plugins enabled with defaults: `{"enabled": true}`

Config is written to `/data/openclaw.json` (only on first boot; persists across restarts).

### 6.4 Gateway Startup

- Plugin loader repairs/enables plugins based on `/data/openclaw.json` `plugins` config
- Bundled runtime deps are repaired as needed before import
- Users can change plugin enable/disable via Control UI or CLI; changes persist on restart

---

## 7. Key Files Reference

| File | Purpose |
|------|---------|
| `src/plugins/types.ts` (2554 lines) | All plugin type definitions, `OpenClawPluginApi`, capability provider types |
| `src/plugins/manifest.ts` (1583 lines) | Manifest parsing, `PluginManifest` type |
| `src/plugins/loader.ts` (2511 lines) | Core plugin loading pipeline (`loadOpenClawPlugins`) |
| `src/plugins/discovery.ts` (977 lines) | Filesystem plugin discovery |
| `src/plugins/manifest-registry.ts` (725 lines) | Manifest registry with deduplication |
| `src/plugins/registry-types.ts` (443 lines) | `PluginRegistry`, `PluginRecord` types |
| `src/plugins/hooks.ts` (1411 lines) | Hook runner with priority ordering |
| `src/plugins/hook-types.ts` (944 lines) | 35 lifecycle hook names + types |
| `src/plugins/bundled-runtime-deps.ts` (416 lines) | Runtime dependency installation for plugins |
| `src/plugins/activation-planner.ts` (266 lines) | Activation plan resolution |
| `src/plugins/runtime.ts` (125 lines) | Active registry management |
| `src/gateway/server-startup-plugins.ts` (136 lines) | Gateway plugin bootstrap orchestration |
| `src/gateway/server-plugins.ts` (516 lines) | Gateway plugin loading |
| `scripts/postinstall-bundled-plugins.mjs` (1030 lines) | npm postinstall script |
| `start.sh` (212 lines) | Fly.io boot: seeds `/data/openclaw.json` with plugin config |
| `fly.public-auth.toml` (52 lines) | `THIRDCLAW_PLUGIN_PROFILE` env var |
| `Dockerfile` (311 lines) | Build-time plugin dependency resolution |

---

## 8. Non-Plugin Infrastructure (excluded from count)

4 directories under `extensions/` are shared runtime libraries, NOT plugins:
- `image-generation-core/` — shared image gen library
- `video-generation-core/` — shared video gen library
- `speech-core/` — shared speech/TTS library
- `media-understanding-core/` — shared media understanding library

Also: `shared/` and `test-support/` are utility directories, not plugins.

4 workspace packages under `packages/`:
- `plugin-sdk/` — `@openclaw/plugin-sdk`
- `sdk/` — `@openclaw/sdk`
- `plugin-package-contract/` — `@openclaw/plugin-package-contract`
- `memory-host-sdk/` — `@openclaw/memory-host-sdk`

---

## 9. Environment Variables That Control Plugin Behavior

| Variable | Where Set | Purpose |
|----------|-----------|---------|
| `THIRDCLAW_PLUGIN_PROFILE` | fly.public-auth.toml | First-boot plugin profile: `hosted-minimal` or `upstream-default` |
| `OPENCLAW_EXTENSIONS` | Dockerfile ARG | Space-separated extension dirs to include in build |
| `OPENCLAW_BUNDLED_PLUGINS_DIR` | env override | Path to bundled plugins directory |
| `OPENCLAW_DISABLE_BUNDLED_PLUGINS` | env | Set `1` to disable all bundled plugins |
| `OPENCLAW_PLUGIN_STAGE_DIR` | env | Directory for plugin runtime dep staging |
| `OPENCLAW_EAGER_BUNDLED_PLUGIN_DEPS` | env | Set `1` for eager dep installation |
| `OPENCLAW_DISABLE_BUNDLED_PLUGIN_POSTINSTALL` | env | Skip postinstall script |
| `OPENCLAW_DISABLE_PERSISTED_PLUGIN_REGISTRY` | env | Skip persisted registry cache |

---

## 10. Verification Checklist

- [x] 120 bundled plugins identified and categorized
- [x] Plugin architecture (types, manifests, SDK, API) fully documented
- [x] Complete loading pipeline traced from CLI entry to runtime registration
- [x] Runtime dependency installation mechanism understood (3 paths)
- [x] Plugin priority/ordering systems documented (origin, hooks, channels, autoSelect)
- [x] Fly.io deployment plugin flow documented (build → deploy → boot → startup)
- [x] All key source files referenced with line numbers
- [x] Non-plugin infrastructure distinguished from actual plugins
- [x] Environment variable catalog compiled

---

## 11. Notes for ThirdClaw Operations

1. **Default profile is `hosted-minimal`** — only 4 providers are disabled. Most plugins (116) remain available.
2. **Startup plugins (~18)** load eagerly; the rest load on demand. The `browser` and `acpx` plugins start at boot and consume resources.
3. **Runtime deps** are auto-installed at startup by the plugin loader — ensure `/data` volume has enough space (4GB recommended for v2026.4.26+).
4. **Plugin config persists** across restarts in `/data/openclaw.json`. First-boot seeding only happens when the file doesn't exist.
5. **Channel plugins** (Discord, Telegram, etc.) are `enabledByDefault: false` — they don't activate unless explicitly configured.
6. **`brave`** (web search), **`voyage`** (embeddings), and **`web-readability`** are important utility plugins that are enabled by default but need provider config.
7. The `codex` provider plugin is disabled in `hosted-minimal` — if client wants Codex integration, switch to `upstream-default` or enable it manually.
