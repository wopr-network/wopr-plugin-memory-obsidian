# wopr-plugin-memory-obsidian

Auto-store and recall conversation memories in an Obsidian vault.

**Depends on:** `@wopr-network/wopr-plugin-obsidian` — uses `ctx.getExtension("obsidian")`
for all vault I/O. Both plugins must be installed.

## Features

- **Auto-store** — listens to `session:afterInject`, extracts memorable content, writes to vault
- **Auto-recall** — context provider searches vault and injects relevant memories before each message
- **Memory augmentation** — listens to `memory:search` and adds vault results to the mutable payload
- **A2A tools** — `memory.store`, `memory.search`, `memory.recall`, `memory.forget`, `memory.list`
- **Extension API** — `ctx.getExtension("memory-obsidian")` for plugin-to-plugin access
- **WebMCP tools** — browser-callable search/store/list for the web dashboard

## Build commands

```bash
bun install          # install deps
bun run build        # tsc → dist/
bun run check        # biome + tsc --noEmit
bun run test         # vitest run
bun run lint:fix     # auto-fix lint issues
```

## Architecture

```
src/
  index.ts                       # WOPRPlugin default export — orchestration only
  extractor.ts                   # Memory extraction strategies (heuristic, full-exchange)
  a2a-tools.ts                   # A2A tool definitions
  webmcp.ts                      # WebMCP tool declarations and registration
  memory-obsidian-schema.ts      # PluginSchema (Zod) for local SQLite index
  memory-obsidian-repository.ts  # Repository wrapper functions
  types.ts                       # Config, VaultClient, MemoryObsidianExtension interfaces
tests/
  extractor.test.ts
  a2a-tools.test.ts
```

## Key details

- **No direct Obsidian HTTP calls** — all vault I/O goes through `ctx.getExtension("obsidian")`
- **Local SQLite index** via PluginSchema/Repository — stores metadata, enables fast lookups
- **Imports only from `@wopr-network/plugin-types`** — VaultClient interface is defined locally
  (duck-typed against the obsidian extension at runtime)
- **Vault note format**: YAML frontmatter (id, session, tags, created) + markdown body
- **Vault path format**: `{vaultFolder}/{YYYY-MM}/{sessionPrefix}-{idPrefix}.md`
- `catch (error: unknown)` everywhere — never `catch (error: any)`
- `shutdown()` is idempotent — safe to call twice
- `randomUUID()` from `node:crypto` — no extra uuid dependency

## Config options

| Field | Default | Description |
|-------|---------|-------------|
| `autoStore` | `"heuristic"` | `"always"` / `"heuristic"` / `"never"` |
| `extractionMode` | `"heuristic"` | `"heuristic"` / `"full-exchange"` |
| `autoRecall` | `"always"` | `"always"` / `"on-demand"` / `"never"` |
| `maxRecallNotes` | `5` | Max memories injected per message |
| `vaultFolder` | `"WOPR/memories"` | Vault folder for memory notes |
| `minExchangeLength` | `300` | Min combined length to trigger heuristic auto-store |

## Issue tracking

All issues in Linear (team: WOPR).
Descriptions must start with `**Repo:** wopr-network/wopr-plugin-memory-obsidian`
