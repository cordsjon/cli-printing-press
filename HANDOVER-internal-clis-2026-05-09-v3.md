# HANDOVER — Internal CLIs Generation — 2026-05-09 v3

## Status at Handover

**Chunk 1 (Consigliere) — DONE**
**Chunk 2 (Hermes) — NOT STARTED**
**Chunk 3 (Poster Engine) — NOT STARTED**
**Chunk 4 (MCP Registration + Wiki Overview) — PARTIAL**

---

## What Was Completed This Session

### Go Installation
- Go 1.26.3 installed via `brew install go` at `/opt/homebrew/bin/go`
- Path must be set: `export PATH="/opt/homebrew/bin:$PATH"`

### Consigliere CLI (Chunk 1) — COMPLETE

**Source generated at:**
`~/printing-press/.runstate/projects-18d1ce8a/runs/20260509-193442/working/consigliere-pp-cli/`

**Binaries built at:**
- `~/printing-press/library/consigliere/consigliere-pp-cli` (18.9 MB)
- `~/printing-press/library/consigliere/consigliere-pp-mcp` (20.9 MB)

**Skill installed:**
`~/.claude/skills/consigliere/SKILL.md` (426 lines)

**MCP registered in `~/.claude/settings.json`:**
```json
"mcpServers": {
  "consigliere": {
    "command": "/Users/jc-folder/printing-press/library/consigliere/consigliere-pp-mcp",
    "args": [],
    "env": {}
  }
}
```

**MCP smoke test:** 128 tools confirmed via stdin/stdout JSON-RPC test.

**Wiki page created:** `cli-tools/consigliere` (page ID 104)

---

## Next Steps — Jump Directly Here

### Chunk 2: Hermes CLI (port 9109)

Run this exact command to generate:

```bash
export PATH="/opt/homebrew/bin:$PATH"
RUNSTATE=~/printing-press/.runstate/projects-18d1ce8a
RUN_ID=$(date +%Y%m%d-%H%M%S)
mkdir -p "$RUNSTATE/runs/$RUN_ID/working"

printing-press generate \
  --spec http://localhost:9109/openapi.json \
  --output "$RUNSTATE/runs/$RUN_ID/working/hermes-pp-cli" \
  --name hermes \
  --force --lenient
```

Then build:
```bash
cd "$RUNSTATE/runs/$RUN_ID/working/hermes-pp-cli"
go mod tidy
mkdir -p ~/printing-press/library/hermes
go build -o ~/printing-press/library/hermes/hermes-pp-cli ./cmd/hermes-pp-cli
go build -o ~/printing-press/library/hermes/hermes-pp-mcp ./cmd/hermes-pp-mcp
```

Then install skill, register MCP, verify tools, doc wiki — same pattern as Consigliere.

### Chunk 3: Poster Engine CLI (port 9120)

Same pattern, `--name poster-engine`, output to `~/printing-press/library/poster-engine/`.

### Chunk 4: Wiki Overview Page

After Chunks 2 & 3 done, create `cli-tools/overview` wiki page listing all 3 CLIs with their binary paths, MCP tool counts, and skill invocations.

---

## Key Facts

- `printing-press` binary: `/usr/local/bin/printing-press` v4.2.0
- Go binary: `/opt/homebrew/bin/go` v1.26.3 (installed this session)
- Press scope: `projects-18d1ce8a`
- Generation warnings to ignore: resource renames (health/search prefixed), `spec.OwnerName` empty
- MCP tool count > spec op count is normal (compound workflows add synthetic tools)
- `--validate` flag causes failure if Go not in PATH — don't use it; use `--lenient` instead
- Two binaries per API: `*-pp-cli` (human CLI) + `*-pp-mcp` (MCP stdio server)

## Consigliere API Context
- 116 operations, 8 tags, no auth required
- Interesting renames: `search` → `consigliere-search`, `health` → `consigliere-health`

