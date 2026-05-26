# @roam-research/roam-tools-core

The official transport-agnostic core for Roam Research's MCP and CLI tools. Provides the tool registry, schemas, operation functions, and dispatch — but no transport.

> **Most users don't want this package directly.** If you want to connect an AI assistant to Roam, install [`@roam-research/roam-mcp`](https://www.npmjs.com/package/@roam-research/roam-mcp). For the command line, install [`@roam-research/roam-cli`](https://www.npmjs.com/package/@roam-research/roam-cli). Both wrap the local Roam Desktop transport ([`@roam-research/roam-tools-local`](https://www.npmjs.com/package/@roam-research/roam-tools-local)) which depends on this package.

## Who is this for?

This package is for **hosted MCP transports** that talk to Roam through a different backend (e.g., an authenticated proxy) and want to reuse the same tool registry, Zod schemas, and operation functions without dragging in the local Roam Desktop client or the `~/.roam-tools.json` config reader.

## What's in here

- `RoamActionClient` — structural client interface (`call()` + optional `getTokenInfo()`). Bring your own implementation.
- `routeToolCall(name, args, options)` — central dispatcher. **Requires** `options.resolveGraph` and `options.createClient`. Optional: `tokenInfoMode`, `onTokenStatusUpdate`.
- `ToolGraph` / `ResolvedGraph` — cross-transport graph identity types.
- Tool registry: `dataTools` (graph content, reusable across transports), `desktopUiTools` (file ops + window/selection — local-only), `contentTools` (the union), `tools` (alias of contentTools at this layer).
- Helpers: `defineTool`, `defineStandaloneTool`, `findTool`.
- Operations: page, block, search, query, datalog, navigation, files (all transport-agnostic — they only call `client.call(...)`).
- Types and schemas: `GraphConfigSchema`, `RoamMcpConfigSchema`, `RoamError`, `ErrorCodes`, `EXPECTED_API_VERSION`, etc.

## What's NOT in here

- `RoamClient` — the local Roam Desktop transport, in `@roam-research/roam-tools-local`.
- `~/.roam-tools.json` reader (`getPort`, `resolveGraph`, `getMcpConfig`, etc.) — also in `@roam-research/roam-tools-local`.
- `connect` interactive setup — also in `@roam-research/roam-tools-local`.
- The `list_graphs` and `setup_new_graph` standalone tools — also local.

Hosted consumers reimplement these (or substitute their own equivalents — e.g., reading grants from a remote store) and inject them via `routeToolCall`'s options.

## Documentation

See the [main repository](https://github.com/Roam-Research/roam-tools) for full documentation, including the architecture rationale and the integration pattern.
