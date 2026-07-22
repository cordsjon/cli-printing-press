# Design: Internal CLIs via Printing Press

**Date:** 2026-05-09
**Status:** Approved
**Author:** Jonas Cords

## Summary

Generate three Go CLIs + MCP servers + Claude Code skills using the Printing Press fast-path, targeting internal FastAPI services. All artifacts are agent-native (MCP) and human-CLI ergonomic. Document each in the wiki with a how-to guide.

## Projects

| # | CLI slug | OpenAPI source | Primary namespaces |
|---|----------|----------------|--------------------|
| 1 | `consigliere` | `http://localhost:9104/openapi.json` | osint, items, sources, runs, ai, research, skills, market, network, tasks |
| 2 | `hermes` | `http://localhost:9109/openapi.json` | backlog, dagu, paperclip, memories, files, chat, a2a |
| 3 | `poster-engine` | `http://localhost:9120/openapi.json` | poster generation, layouts, export |

> Hermes and Paperclip share the hermes-adapter process (port 9109) and are generated as one unified CLI.

## Artifact Set (per CLI)

Each press run produces:

1. **Go binary** — `~/printing-press/library/<slug>/<slug>-pp-cli`
2. **Claude Code skill** — `~/.claude/skills/<slug>/` (auto-loaded in all sessions)
3. **MCP server** — served by the binary via `<slug>-pp-cli mcp`
4. **`tools-manifest.json`** — MCP tool catalog
5. **Wiki page** — `CLI Tools / <Name>` with install, env var, and 5+ command examples

## Execution Order

Sequential. Each CLI is fully verified before starting the next.

```
consigliere  →  hermes  →  poster-engine
```

### Per-CLI Steps

```
1. Confirm service running:   curl http://localhost:<port>/openapi.json
2. Run press fast-path:       /printing-press --spec <url> --name <slug>
3. Build + verify:            go build, --help, doctor  (press runs internally)
4. Install skill:             copy generated skill → ~/.claude/skills/<slug>/
5. Smoke-test MCP:            <slug>-pp-cli mcp  (verify tools register)
6. Document in wiki:          wiki.py set-section "CLI Tools/<Name>" <content>
```

## MCP Registration

After all three CLIs ship, register them in `~/.claude/settings.json`:

```json
"mcpServers": {
  "consigliere":  { "command": "/Users/jc-folder/printing-press/library/consigliere/consigliere-pp-cli",   "args": ["mcp"] },
  "hermes":       { "command": "/Users/jc-folder/printing-press/library/hermes/hermes-pp-cli",             "args": ["mcp"] },
  "poster-engine":{ "command": "/Users/jc-folder/printing-press/library/poster-engine/poster-engine-pp-cli","args": ["mcp"] }
}
```

## Auth / Config

All three services are local with no API key requirements. Base URL is pinned via env var at binary install time. No credentials in generated artifacts.

## Wiki Structure

New top-level section `CLI Tools` with four pages:

- **Overview** — common pattern (`--help`, `mcp`, env var setup), links to all three CLIs
- **Consigliere** — install, env var, 5+ command examples (osint run, research, items list…)
- **Hermes** — install, env var, 5+ command examples (backlog list, paperclip agents, dagu start…)
- **Poster Engine** — install, env var, 5+ command examples (generate, layouts, export…)

## Failure Handling

| Failure | Action |
|---------|--------|
| `go build` fails | Fix before proceeding — never ship broken binary |
| `verify` gate fails | One fix loop; if still failing, document gap and continue |
| Service not running at start | Skip that CLI, note it, return after others complete |
| Ordering failure | Strict: consigliere → hermes → poster-engine; failure in one does not cascade |

## Pre-requisites

- [ ] Printing Press binary built (`go build -o ./printing-press ./cmd/printing-press`)
- [ ] Consigliere running on port 9104
- [ ] Hermes-adapter running on port 9109
- [ ] PosterEngine running on port 9120
- [ ] `wiki.py` accessible at `00_Governance/wiki/scripts/wiki.py`
