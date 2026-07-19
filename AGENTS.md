# Repository Guidelines

## Project Overview

OpenClaw is a multi-channel AI gateway and extensible messaging integration platform. It routes inbound messaging channels (Slack, Discord, Telegram, Google Chat, Matrix, etc.) to isolated, stateful developer agents and custom tools, facilitating robust automation and natural language execution.

## Architecture & Data Flow

- **WebSocket Gateway**: Serves as the central orchestration server (`src/gateway/server.ts`, `src/gateway/http-server.ts`) coordinating RPC, active sessions, and plugin runtimes.
- **Agent Runtimes**: Pluggable models managed under `src/plugins/` and dispatched dynamically.
- **State Store**: Uses a SQLite state database (default path `~/.openclaw/state/openclaw.sqlite`) to track active sessions, audit ledgers, and paired device configurations.
- **Model Context Protocol (MCP)**: Adapts client connections and forwards tool invocations dynamically to registered servers.
- **Inference & Dispatch**: Incoming turns resolve matching agent workspaces (default `~/.openclaw/workspace`), resolve model configurations (main model vs fallbacks), evaluate tool policies, run preflight hooks, dispatch execution turns, and return normalized payloads to the channel.

## Key Directories

- `src/gateway/`: Core WebSocket Gateway server, HTTP server, and connection handling.
- `src/channels/`: Inbound and outbound channel adapters (Telegram, Slack, Discord, SMS).
- `src/plugins/`: Plugins loader, discovery mechanism, and built-in extensions.
- `src/security/`: Security auditing checks, ACL verification, and SecretRef masking.
- `packages/gateway-protocol/`: Shared RPC schemas, API request formats, and validation logic.
- `packages/ai/`: Unified transport layers and model routing interfaces.
- `packages/llm-core/`: Provider catalog entries, reasoning token decoders, and API adapters.
- `packages/agent-core/`: Core database migrations, sessions, and workspace storage structures.
- `apps/`: UI components and platform-native client applications (macOS, iOS, Android).

## Development Commands

OpenClaw is a pnpm monorepo. Use Node.js >= 24.

- **Install dependencies**: `pnpm install`
- **Bootstrap workspace**: `pnpm openclaw setup` or `pnpm openclaw onboard --non-interactive --accept-risk --auth-choice skip`
- **Build assets**: `pnpm build` or `pnpm ui:build` (builds control UI)
- **Start gateway in foreground**: `pnpm openclaw gateway run`
- **Run linter**: `pnpm lint` or `pnpm lint:fix`
- **Run tests**: `pnpm vitest run <path_to_test_file>` or `pnpm test`

## Code Conventions & Common Patterns

- **Language**: TypeScript (ES2022 / ESNext modules).
- **Naming**:
  - PascalCase for interfaces/types/schemas.
  - camelCase for functions, variables, and modules.
  - snake_case for SQLite tables and columns.
- **Error Handling**: Use structured results (`Result` from `@openclaw/normalization-core/result`). Never throw uncaught exceptions in async paths or background hooks.
- **Async Patterns**: High concurrency using `Promise.all` and structured event pools.
- **State Management**: Persisted to SQLite, read via Kysely query builders. Runtime configuration pinned in memory and refreshed on reload.
- **Dependency Injection**: Parameter passing and registry objects (e.g. `ResolvedPlugins`, `GatewayAuthConfig`).

## Important Files

- `openclaw.mjs`: Core CLI entry point.
- `package.json`: Monorepo scripts, engines configuration, and workspace tools.
- `~/.openclaw/openclaw.json`: Main gateway, model, and tool configuration file.
- `src/security/audit-extra.sync.ts`: Core security auditing logic for config properties.

## Runtime/Tooling Preferences

- **Runtime**: Node.js (version 22.x/24.x).
- **Package Manager**: pnpm.
- **Tooling Constraints**: No absolute paths or raw shell execution without sandboxing. Secret keys must be loaded via `SecretRef` configs rather than plaintext strings.

## Testing & QA

- **Framework**: Vitest (v4.1.10).
- **Executing Tests**: Run `pnpm vitest run <test-file>` to test isolated unit/integration files.
- **Coverage**: Expected to maintain high unit coverage on core algorithms, protocols, and security extra checks.
