# OpenClaw — AI Assistant Guide

> **Note:** `AGENTS.md` is the canonical reference for development conventions, git ops, and agent-specific notes.
> This file provides a structural and architectural overview specifically for Claude Code and similar AI coding assistants.

---

## Project Overview

**OpenClaw** is a personal AI gateway and assistant that connects to messaging channels you already use. It routes conversations through a central "gateway" process and dispatches to AI providers (Anthropic Claude, OpenAI, etc.).

- **Runtime:** Node ≥ 22.12.0 (TypeScript/ESM throughout)
- **Package manager:** pnpm (v10.23.0) — always use `pnpm`, not `npm` or `yarn`
- **Build tool:** tsdown (rolldown-based)
- **Test framework:** Vitest with V8 coverage
- **Linter/formatter:** Oxlint + Oxfmt (not ESLint/Prettier)

---

## Repository Layout

```
openclaw/
├── src/                    # Core TypeScript source (the gateway + CLI)
├── extensions/             # Extension channel packages (workspace packages)
├── apps/                   # Native platform apps
│   ├── macos/              # macOS Swift app
│   ├── ios/                # iOS Swift app
│   ├── android/            # Android Kotlin/Gradle app
│   └── shared/             # Shared OpenClawKit Swift library
├── packages/               # Internal workspace packages
│   ├── clawdbot/           # Bot identity/branding package
│   └── moltbot/            # Secondary bot package
├── ui/                     # Control UI (Lit web components, Vite-built)
├── docs/                   # Documentation (Mintlify-hosted at docs.openclaw.ai)
├── skills/                 # Bundled skills/agent capabilities
├── scripts/                # Build, test, release automation scripts
├── test/                   # Shared test helpers
├── dist/                   # Build output (gitignored)
└── git-hooks/              # Git hook scripts (pre-commit, etc.)
```

---

## `src/` Structure — Core Source

```
src/
├── entry.ts                # Node process entry (handles respawn for NODE_OPTIONS)
├── index.ts                # Main gateway/CLI bootstrap
├── runtime.ts              # Runtime initialization helpers
├── globals.ts              # Global type declarations
│
├── cli/                    # CLI argument parsing, profile management, respawn logic
├── commands/               # All CLI subcommand implementations
│   ├── agent.ts            # `openclaw agent` — run AI agent
│   ├── gateway-status.ts   # `openclaw gateway status`
│   ├── onboard*.ts         # `openclaw onboard` wizard (many files)
│   ├── configure*.ts       # `openclaw configure`
│   ├── channels*.ts        # `openclaw channels`
│   ├── doctor*.ts          # `openclaw doctor` diagnostics
│   ├── sessions.ts         # `openclaw sessions`
│   └── ...                 # Other commands
│
├── gateway/                # Gateway HTTP/WebSocket server
│   ├── server.ts           # Main gateway server
│   ├── server-http.ts      # HTTP endpoint registration
│   ├── server-chat.ts      # Chat message handling
│   ├── server-channels.ts  # Channel management
│   ├── server-plugins.ts   # Plugin lifecycle
│   ├── server-cron.ts      # Cron/scheduled task handling
│   ├── hooks.ts            # Hook runner (before/after agent, etc.)
│   ├── auth.ts             # Gateway authentication
│   ├── session-utils.ts    # Session storage/retrieval
│   └── ...
│
├── channels/               # Channel-agnostic message routing helpers
│   ├── registry.ts         # Channel registry
│   ├── allowlists/         # Allowlist matching logic
│   ├── command-gating.ts   # Per-channel command gating
│   ├── mention-gating.ts   # Mention-based gating
│   └── ...
│
├── routing/                # Message routing (resolve-route, session keys)
│   ├── resolve-route.ts
│   └── session-key.ts
│
├── agents/                 # AI agent engine (subagent support, auth profiles)
│   ├── auth-profiles/      # Auth profile rotation + failover
│   └── ...
│
├── plugins/                # Plugin system (bundled plugins, hooks, discovery)
│
├── sessions/               # Session-level overrides (model, send policy, etc.)
│
├── providers/              # AI provider adapters and auth helpers
│   ├── github-copilot-*.ts
│   ├── google-shared.*.ts
│   ├── qwen-portal-oauth.ts
│   └── ...
│
├── config/                 # Config loading, sessions store, schema
├── infra/                  # Infrastructure helpers (env, ports, binaries, errors)
├── logging/                # Structured logging internals
├── memory/                 # Memory/recall subsystem
├── media/                  # Media pipeline (images, PDFs, etc.)
├── media-understanding/    # Media content analysis
├── link-understanding/     # URL/link content extraction
├── security/               # Security utilities
├── daemon/                 # Gateway daemon management (launchd/systemd)
├── cron/                   # Cron scheduling
├── tts/                    # Text-to-speech
├── tui/                    # Terminal UI
├── canvas-host/            # Canvas rendering host
├── node-host/              # Node execution host
├── process/                # Child process management
├── markdown/               # Markdown rendering
│
│ # Built-in channel integrations (core):
├── telegram/               # Telegram (grammY)
├── discord/                # Discord
├── slack/                  # Slack (Bolt)
├── signal/                 # Signal
├── imessage/               # iMessage (macOS)
├── whatsapp/               # WhatsApp (Baileys)
├── web/                    # Web/WebChat channel
├── line/                   # LINE messaging
│
├── acp/                    # Agent Client Protocol (ACP/SDK integration)
├── pairing/                # Device pairing
├── wizard/                 # Onboarding wizard engine
├── plugin-sdk/             # Public Plugin SDK (exported as `openclaw/plugin-sdk`)
└── shared/                 # Shared types/utilities
```

---

## `extensions/` — Extension Channels

Each extension is a pnpm workspace package with its own `package.json`. Extensions **must not** add deps to the root `package.json`.

| Extension | Description |
|---|---|
| `bluebubbles` | BlueBubbles (iMessage bridge) |
| `discord` | Discord (extension variant) |
| `feishu` | Feishu/Lark |
| `googlechat` | Google Chat |
| `irc` | IRC |
| `line` | LINE |
| `matrix` | Matrix protocol |
| `msteams` | Microsoft Teams |
| `signal` | Signal (extension variant) |
| `slack` | Slack (extension variant) |
| `telegram` | Telegram (extension variant) |
| `voice-call` | Voice call support |
| `whatsapp` | WhatsApp (extension variant) |
| `zalo` / `zalouser` | Zalo (Vietnamese platform) |
| `talk-voice` | Talk/voice mode |
| `nostr` | Nostr protocol |
| `tlon` | Tlon/Urbit |
| `twitch` | Twitch chat |
| `llm-task` | LLM task runner extension |
| `memory-core` | Memory subsystem (core) |
| `memory-lancedb` | Memory subsystem (LanceDB vector store) |
| `shared` | Shared extension utilities |
| `copilot-proxy` | GitHub Copilot proxy |
| `lobster` | Lobster (internal tool) |
| `open-prose` | Open Prose |
| `phone-control` | Phone control |
| `device-pair` | Device pairing extension |
| `diagnostics-otel` | OpenTelemetry diagnostics |
| `thread-ownership` | Thread ownership management |

---

## Key Architectural Concepts

### Gateway
The **Gateway** is a long-running HTTP/WebSocket server (default port 18789) that:
- Accepts connections from channel adapters (Telegram, WhatsApp, etc.)
- Maintains sessions and routes messages to AI providers
- Exposes a Control UI (Lit web components) and REST/WS API
- Can run as a daemon (launchd on macOS, systemd on Linux)

### Channels
Channels are message source/sink adapters. Each channel has:
- A transport layer (webhook, polling, WebSocket)
- An allowlist for controlling who can interact
- Routing rules (session keys, reply prefixes, etc.)

Core channels live in `src/<channel-name>/`.
Extension channels live in `extensions/<channel-name>/`.

### Sessions
Sessions track conversation context. Session keys determine routing continuity (same session = same conversation thread). Overrides (model, send policy) are applied per-session.

### Plugins
Plugins extend gateway behavior. They register via `src/plugins/` and can add commands, hooks, routes, and scheduled tasks.

### Plugin SDK
External plugins import from `openclaw/plugin-sdk` (path alias resolved via jiti at runtime). The SDK is built separately with `build:plugin-sdk:dts`.

### Hooks
Hooks fire at lifecycle points (before agent start, after response, etc.). Configured in gateway config; the runner is `src/gateway/hooks.ts`.

### Auth Profiles
Multiple auth credentials can be configured (Anthropic, OpenAI, etc.) with automatic rotation and failover. Profile logic lives in `src/agents/auth-profiles/`.

---

## Build & Development Commands

```bash
# Install dependencies
pnpm install

# Full build (tsdown + scripts)
pnpm build

# Type-check only (fast, using native TypeScript)
pnpm tsgo

# Lint + format check
pnpm check

# Auto-fix linting + formatting
pnpm lint:fix

# Run the CLI in dev mode (no build needed)
pnpm openclaw <command>
# or:
pnpm dev

# Run the gateway in dev mode (skips channels)
pnpm gateway:dev

# Run all unit tests
pnpm test:fast

# Run unit + gateway tests (CI default)
pnpm test

# Run with coverage
pnpm test:coverage

# Run e2e tests
pnpm test:e2e

# Build the Control UI
pnpm ui:build

# Dev UI with HMR
pnpm ui:dev
```

---

## Testing Conventions

- Tests co-locate with source: `src/foo.ts` → `src/foo.test.ts`
- E2E tests: `*.e2e.test.ts`
- Live tests (require real credentials): `*.live.test.ts`
- Vitest configs:
  - `vitest.unit.config.ts` — unit tests only (excludes `src/gateway/`, extensions)
  - `vitest.e2e.config.ts` — e2e tests
  - `vitest.gateway.config.ts` — gateway tests
  - `vitest.live.config.ts` — live/integration tests
  - `vitest.extensions.config.ts` — extension tests
- Coverage threshold: **70%** lines/branches/functions/statements
- Do not set test workers above 16

Run live tests:
```bash
CLAWDBOT_LIVE_TEST=1 pnpm test:live
```

---

## Code Style Conventions

- **Language:** TypeScript strict mode, ESM only
- **Formatting:** Oxfmt (`pnpm format` to check, `pnpm format:fix` to fix)
- **Linting:** Oxlint with type-aware rules (`pnpm lint`)
- **Never use:** `@ts-nocheck`, `no-explicit-any` suppression, prototype mutation
- **Class composition:** use `extends` chains or explicit composition, never `applyPrototypeMixins`
- **File length:** aim for ≤ 700 LOC; refactor when it improves clarity
- **UI decorators:** Lit uses legacy decorators (`@state()`, `@property()`) — do not switch to standard decorators
- **Naming product:** "OpenClaw" in docs/headings; `openclaw` for CLI, paths, config keys

---

## Commit & PR Conventions

- Use `scripts/committer "<msg>" <file...>` to scope commits precisely
- Commit message style: `Scope: action-oriented description` (e.g., `CLI: add verbose flag`)
- Keep PRs focused (one concern per PR)
- AI-assisted PRs are welcome — mark them in the PR title/description
- Full PR workflow: see `.agents/skills/PR_WORKFLOW.md`

---

## Configuration & Secrets

- User config: `~/.openclaw/` (credentials, sessions, profile)
- Never commit real phone numbers, credentials, or live config values
- Use `openclaw config set <key> <value>` for runtime config changes
- `openclaw doctor` diagnoses config/migration issues

---

## Docs

- Hosted on Mintlify at `https://docs.openclaw.ai`
- Source: `docs/` directory
- Internal doc links: root-relative, no `.md`/`.mdx` extension (e.g. `[Config](/configuration)`)
- Do not edit `docs/zh-CN/` directly — it is generated via i18n pipeline
- Do not add em dashes or apostrophes to headings (breaks Mintlify anchors)

---

## Mobile & Desktop Apps

| Platform | Location | Build tool |
|---|---|---|
| macOS | `apps/macos/` | Xcode / Swift |
| iOS | `apps/ios/` | Xcode / XcodeGen |
| Android | `apps/android/` | Gradle |
| Shared Swift lib | `apps/shared/` | Swift Package Manager |

Mobile build commands in `package.json`: `ios:build`, `ios:run`, `android:assemble`, `android:run`.

---

## Key Dependencies

| Package | Role |
|---|---|
| `grammy` | Telegram Bot API |
| `@slack/bolt` | Slack Bolt framework |
| `@whiskeysockets/baileys` | WhatsApp Web protocol |
| `discord-api-types` | Discord types |
| `express` v5 | Gateway HTTP server |
| `ws` | WebSocket server |
| `vitest` | Test runner |
| `tsdown` | Build bundler (rolldown-based) |
| `oxlint` + `oxfmt` | Linting and formatting |
| `zod` v4 | Schema validation |
| `@sinclair/typebox` | JSON Schema types |
| `jiti` | Runtime TypeScript/alias resolution for plugins |
| `chokidar` | File watching |
| `playwright-core` | Browser automation |
| `sharp` | Image processing |
| `pdfjs-dist` | PDF parsing |
| `croner` | Cron scheduling |
| `@mariozechner/pi-*` | Pi agent core / TUI |
| `@agentclientprotocol/sdk` | ACP (Agent Client Protocol) SDK |

---

## CI / Release

- Versions follow calendar versioning: `vYYYY.M.D`
- Release channels: `stable` (latest), `beta` (prerelease), `dev` (main HEAD)
- Before any release: read `docs/reference/RELEASING.md` and `docs/platforms/mac/release.md`
- Release check: `pnpm release:check`

---

## Agent/AI Assistant Notes

- See `AGENTS.md` for the canonical agent operations guide (git ops, shorthand commands, VM ops, GHSA)
- When working on GitHub Issues or PRs, print the full URL at the end of the task
- Verify claims in source code before answering; do not guess
- When touching docs, end replies with `https://docs.openclaw.ai/...` URLs for referenced pages
- "makeup" is shorthand for "mac app" in maintainer vocabulary
- When adding a new `AGENTS.md` in a subdirectory, also create `CLAUDE.md` as a symlink: `ln -s AGENTS.md CLAUDE.md`
