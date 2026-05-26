# Graph Resolution

How the MCP server determines which Roam graph to use.

## Configuration File

The MCP server reads graph configuration from `~/.roam-tools.json`:

```json
{
  "graphs": [
    {
      "name": "my-graph-name",
      "type": "hosted",
      "token": "roam-graph-local-token-...",
      "nickname": "my-graph"
    }
  ]
}
```

Each graph requires:

- `name`: The actual graph name in Roam
- `type`: `"hosted"` (cloud) or `"offline"` (local-only)
- `token`: Local API token from Roam settings
- `nickname`: Slug identifier for the graph (lowercase, hyphens, no spaces). Must match `[a-z0-9]+(-[a-z0-9]+)*`

## Resolution Order

Graph resolution is stateless — every tool call resolves the graph independently:

1. **Explicit graph parameter** — If the tool call includes a `graph` param, look it up by nickname (or name as fallback)
2. **Auto-select** — If exactly one graph is configured, use it automatically
3. **Error** — If multiple graphs are configured and no `graph` param is provided, return error with `available_graphs` inline

## Nickname Resolution

Graphs are referenced by nickname (case-insensitive) with a fallback to the canonical name:

- `--graph "my-graph"` → matches nickname "my-graph"
- `--graph "my-actual-graph"` → matches by canonical name as fallback

Nicknames are constrained to slugs (`[a-z0-9]+(-[a-z0-9]+)*`). The `connect` CLI auto-slugifies user input.

## Response Format

All client tool responses are prefixed with `Roam graph: {nickname}` so the agent always knows which graph it's operating on.

## Port Discovery

The API port is read from `~/.roam-local-api.json` (written by Roam):

```json
{
  "port": 3333
}
```

If the file doesn't exist, defaults to port 3333.

## Error Cases

| Scenario                          | Result                                                                    |
| --------------------------------- | ------------------------------------------------------------------------- |
| Config file not found             | Error: "Roam MCP config not found" with setup instructions                |
| Graph not in config               | Error listing available graphs                                            |
| Multiple graphs, no `graph` param | Error with `available_graphs` inline — no extra `list_graphs` call needed |
| Invalid token                     | Authentication error with guidance                                        |
| Roam not running                  | Launches Roam via deep link and retries                                   |

## RoamClient Error Mapping

What the local `RoamClient` throws for each Roam Desktop API response. Source of truth: `handleApiError` in `packages/local/src/client.ts`.

| Desktop API response                                                       | `RoamClient` throws                                                                                                                                                                                 |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `success:false` with `code:"VERSION_MISMATCH"` (any status; checked first) | `VERSION_MISMATCH` — message advises which side (Roam vs MCP server) to update, based on server version vs `EXPECTED_API_VERSION`                                                                   |
| HTTP 401                                                                   | `RoamError(authGuidance, code)` — **passes the server's `code` through** (commonly `MISSING_TOKEN` / `INVALID_TOKEN_FORMAT` / `WRONG_GRAPH_TYPE` / `TOKEN_NOT_FOUND`); guidance text varies by code |
| HTTP 403                                                                   | `RoamError(permissionGuidance, code)` — **passes the server's `code` through** (commonly `INSUFFICIENT_SCOPE` / `SCOPE_EXCEEDS_PERMISSION`)                                                         |
| HTTP 404                                                                   | `UNKNOWN_ACTION`                                                                                                                                                                                    |
| HTTP ≥ 500                                                                 | `INTERNAL_ERROR` (adds an encrypted-graph hint when the message mentions a promise error)                                                                                                           |
| Other non-success                                                          | `RoamError(message, code)` — passes the server's `code` through                                                                                                                                     |
| Network error / timeout / connection refused (after retries)               | `CONNECTION_FAILED` — on connection-refused, first launches Roam via deep link and retries                                                                                                          |

This is the **local** transport's mapping. A hosted consumer maps its own backend's responses independently — see `docs/architecture.md` §2c, where `ErrorCodes` is described as a recommended vocabulary, not a hard contract.

## Graph Type Handling

- **Hosted graphs** (default): Standard cloud-synced Roam graphs
- **Offline graphs**: Local-only graphs, requires `?type=offline` query param

The MCP server handles this automatically based on the `type` field in config.

## Same-Name Collision

If both a hosted and offline graph have the same name:

- The hosted graph takes precedence
- The offline graph is ignored
- A warning is logged

Use unique nicknames to avoid confusion.
