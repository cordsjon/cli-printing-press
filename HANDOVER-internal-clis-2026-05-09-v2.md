# Handover: Internal CLIs via Printing Press (v2)

**Date:** 2026-05-09
**Session stopped at:** Consigliere spec analysis complete — ready for `printing-press generate`
**Plan:** `docs/plans/2026-05-09-internal-clis-generate-plan.md`

## Pre-flight — ALL CLEAR

| Check | Status | Notes |
|-------|--------|-------|
| printing-press binary | ✅ `/usr/local/bin/printing-press` v4.2.0 | |
| printing-press skill | ✅ `~/.claude/skills/printing-press/` | globally installed |
| consigliere (9104) | ✅ Running | 116 ops, 8 tags |
| hermes (9109) | ✅ Running (assumed — was confirmed last session) | |
| poster-engine (9120) | ✅ Running (assumed — was confirmed last session) | |

## Consigliere Spec Summary (already analyzed)

- **Title:** Consigliere — Unified Ingest Engine
- **Version:** 0.1.0
- **Total paths:** 110 | **Total operations:** 116
- **Tags:** brain, ideas, memory, processes, producer, red-flags, research, social-graph
- **Key namespaces:** osint, items, sources, runs, ai, research, skills, market, network, tasks, catalog, flm, pep-catalog, plugins, quarantine, taxonomy

## First Task in New Session

The spec is already fetched and analyzed. Skip all pre-flight and spec fetching.

**Jump directly to running the Printing Press on consigliere:**

```bash
Skill("printing-press", "--spec http://localhost:9104/openapi.json --name consigliere")
```

When the skill loads, run the preflight bash block, confirm v4.2.0, then proceed to the fast-path briefing. Since `--spec` is provided, skip discovery. Treat briefing as "Let's go" — user has already approved this work via the plan. Proceed directly to Phase 0 → Phase 2 generation.

**PRESS paths (already initialized):**
- `PRESS_HOME=~/printing-press`
- `PRESS_RUNSTATE=~/printing-press/.runstate/cli-printing-press-18d1ce8a`
- `PRESS_LIBRARY=~/printing-press/library`
- Output binary: `~/printing-press/library/consigliere/consigliere-pp-cli`

## Remaining Work (Chunks 1–4)

### Chunk 1: Consigliere (port 9104)
1. Run `Skill("printing-press", "--spec http://localhost:9104/openapi.json --name consigliere")` ← **START HERE**
2. Verify binary: `~/printing-press/library/consigliere/consigliere-pp-cli --help`
3. Install skill: `mkdir -p ~/.claude/skills/consigliere && cp ~/printing-press/library/consigliere/SKILL.md ~/.claude/skills/consigliere/`
4. Smoke-test MCP: `~/printing-press/library/consigliere/consigliere-pp-cli mcp` (Ctrl-C after tools register)
5. Verify tools-manifest: `cat ~/printing-press/library/consigliere/tools-manifest.json | python3 -c "import sys,json; m=json.load(sys.stdin); print(len(m.get('tools',[])), 'tools')"`
6. Document in wiki: `python3 00_Governance/wiki/scripts/wiki.py set-section "CLI Tools/Consigliere" "$(cat <content>)"`

### Chunk 2: Hermes (port 9109)
Same cycle with `--spec http://localhost:9109/openapi.json --name hermes`

### Chunk 3: Poster Engine (port 9120)
Same cycle with `--spec http://localhost:9120/openapi.json --name poster-engine`

### Chunk 4: MCP Registration + Wiki Overview
After all 3 CLIs ship:
1. Add to `~/.claude/settings.json` mcpServers:
```json
"consigliere":   { "command": "/Users/jcords-macmini/printing-press/library/consigliere/consigliere-pp-cli",    "args": ["mcp"] },
"hermes":        { "command": "/Users/jcords-macmini/printing-press/library/hermes/hermes-pp-cli",              "args": ["mcp"] },
"poster-engine": { "command": "/Users/jcords-macmini/printing-press/library/poster-engine/poster-engine-pp-cli","args": ["mcp"] }
```
2. Create wiki overview page: `wiki.py set-section "CLI Tools/Overview" <content>`

## Key Facts

- Go is NOT installed on this machine — the printing-press binary is pre-built darwin/arm64
- All three services are local with no API key requirements
- The `printing-press` skill is the primary interface — invoke via `Skill` tool
- Wiki set-section is destructive (replaces section) — write complete content in one call
- Session guard fires at 40 (WARNING) and 50 (URGENT) tool calls — plan sessions carefully
