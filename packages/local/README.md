# @roam-research/roam-tools-local

Local Roam Desktop API transport for [`@roam-research/roam-tools-core`](https://www.npmjs.com/package/@roam-research/roam-tools-core). Wraps core with a `RoamClient` that talks to the Roam Desktop app on `127.0.0.1`, plus the `~/.roam-tools.json` config reader and the interactive `connect` setup flow.

> **This package is not meant to be used directly.** It's an internal dependency of [`@roam-research/roam-mcp`](https://www.npmjs.com/package/@roam-research/roam-mcp) (MCP server) and [`@roam-research/roam-cli`](https://www.npmjs.com/package/@roam-research/roam-cli) (CLI). If you want to connect an AI assistant to Roam, install `@roam-research/roam-mcp`. If you want a command-line interface, install `@roam-research/roam-cli`.

## What's in here

- `RoamClient` — authenticated HTTP client for Roam's local API (port discovery via `~/.roam-local-api.json`, deep-link retry, token-info endpoint).
- Graph resolution & config I/O: `getPort`, `resolveGraph`, `getMcpConfig`, `saveGraphToConfig`, `removeGraphFromConfig`, `updateGraphTokenStatus`, `getConfiguredGraphs[Safe]`, `findGraphConfig`, `getOpenGraphs`. Reads/writes `~/.roam-tools.json` statelessly (no in-memory cache).
- Roam Desktop HTTP helpers: `fetchAvailableGraphs`, `requestToken`, `openRoamApp`, `slugify`.
- `connect()` interactive setup — exposed as a separate subpath export (`@roam-research/roam-tools-local/connect`) so consumers can lazy-load `@inquirer/prompts` only when needed.
- `graphManagementTools` — the two local-only standalone tools: `list_graphs`, `setup_new_graph`.
- Local-defaults `routeToolCall(name, args, options?)` — wraps core's `routeToolCall`, baking in `resolveGraph` (reads `~/.roam-tools.json`), `createClient` (constructs `RoamClient`), `tokenInfoMode: "local-sync"`, and `onTokenStatusUpdate` (writes `~/.roam-tools.json`). Also dispatches local standalone tools directly.
- Re-exports the full surface of `@roam-research/roam-tools-core` so consumers need only one import.

## How it relates to core

Core is transport-agnostic; local provides the local Roam Desktop transport. `@roam-research/roam-mcp` and `@roam-research/roam-cli` import from this package; a hosted MCP transport imports from `@roam-research/roam-tools-core` directly and supplies its own `resolveGraph` + `createClient`.

## Documentation

See the [main repository](https://github.com/Roam-Research/roam-tools) for full documentation.
