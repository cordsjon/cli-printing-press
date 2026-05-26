# Internal CLIs via Printing Press — Implementation Plan

> **For agentic workers:** REQUIRED: Use `/sh:execute` to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Generate three agent-native Go CLIs (consigliere, hermes, poster-engine) targeting local FastAPI services, then register them as MCP servers and document them in the wiki.

**Architecture:** Each CLI is produced by the Printing Press fast-path (`/printing-press --spec <url> --name <slug>`), which — given an OpenAPI spec directly — skips discovery and runs analysis → generation → shipcheck. The generated binary lives at `~/printing-press/library/<slug>/<slug>-pp-cli`; its `SKILL.md` is then installed to `~/.claude/skills/<slug>/` for auto-load in all sessions. MCP servers are registered in `~/.claude/settings.json` after all three CLIs pass shipcheck.

**Tech Stack:** Go 1.26+, Printing Press binary, wiki.py, Claude Code settings.json

---

## Chunk 0: Pre-flight

**Goal:** Confirm every prerequisite is satisfied before touching any CLI.

- [ ] **Step 1: Confirm printing-press binary exists**

```bash
printing-press --version
```

Expected: prints version string. If `command not found`, build it first:

```bash
cd /Users/jcords-macmini/projects/cli-printing-press
go build -o ./printing-press ./cmd/printing-press
# Then ensure it's on PATH — add to ~/.zshrc or copy to /usr/local/bin
cp ./printing-press /usr/local/bin/printing-press
printing-press --version
```

- [ ] **Step 2: Check all three services are reachable**

Run in parallel:

```bash
curl -sf http://localhost:9104/openapi.json | python3 -c "import sys,json; d=json.load(sys.stdin); print('consigliere OK:', d.get('info',{}).get('title','?'))"
curl -sf http://localhost:9109/openapi.json | python3 -c "import sys,json; d=json.load(sys.stdin); print('hermes OK:', d.get('info',{}).get('title','?'))"
curl -sf http://localhost:9120/openapi.json | python3 -c "import sys,json; d=json.load(sys.stdin); print('poster-engine OK:', d.get('info',{}).get('title','?'))"
```

Expected: all three print `OK` lines. If any service is down, skip its CLI chunk, note it, and proceed with the others. Return to the skipped CLI after the others complete.

- [ ] **Step 3: Confirm wiki.py is accessible**

```bash
python3 /Users/jcords-macmini/projects/00_Governance/wiki/scripts/wiki.py --help 2>&1 | head -5
```

Expected: prints usage. If not found, stop — wiki documentation is a required deliverable.

---

## Chunk 1: Consigliere CLI

**Services:** `http://localhost:9104/openapi.json`
**Destination:** `~/printing-press/library/consigliere/consigliere-pp-cli`
**Skill install:** `~/.claude/skills/consigliere/SKILL.md`

- [ ] **Step 1: Run Printing Press fast-path**

Invoke the `printing-press` skill with:

```
args: --spec http://localhost:9104/openapi.json --name consigliere
```

The skill will:
1. Read the spec directly (skips discovery, browser-sniff, crowd-sniff)
2. Write a research brief
3. Generate the CLI
4. Run shipcheck (go build, --help, doctor, verify, scorecard)

**Wait for the skill to reach shipcheck verdict before proceeding.** If shipcheck fails, follow the skill's one-fix-loop guidance. If still failing, document the gap and proceed to Chunk 2.

- [ ] **Step 2: Verify binary exists and responds**

```bash
ls -lh ~/printing-press/library/consigliere/consigliere-pp-cli
~/printing-press/library/consigliere/consigliere-pp-cli --version
~/printing-press/library/consigliere/consigliere-pp-cli --help | head -20
```

Expected: binary exists, version printed, help output shows namespaces (osint, items, sources, runs, ai, research, skills, market, network, tasks).

- [ ] **Step 3: Install skill**

```bash
mkdir -p ~/.claude/skills/consigliere
cp ~/printing-press/library/consigliere/SKILL.md ~/.claude/skills/consigliere/SKILL.md
echo "Skill installed:"
head -5 ~/.claude/skills/consigliere/SKILL.md
```

- [ ] **Step 4: Smoke-test MCP server startup**

```bash
timeout 5 ~/printing-press/library/consigliere/consigliere-pp-cli mcp 2>&1 | head -30 || true
```

Expected: output shows the MCP server initializing and listing tools. A timeout exit is expected (server runs until killed) — check that tool names appear in the output before it times out. If no tool names appear, run `consigliere-pp-cli tools-audit` to inspect the tools manifest.

- [ ] **Step 5: Verify tools-manifest.json**

```bash
cat ~/printing-press/library/consigliere/tools-manifest.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Tools: {len(d.get(\"tools\",[]))}')"
```

Expected: `Tools: N` where N > 0.

- [ ] **Step 6: Document in wiki**

```bash
python3 /Users/jcords-macmini/projects/00_Governance/wiki/scripts/wiki.py set-section "CLI Tools/Consigliere" "$(cat <<'EOF'
## Consigliere CLI

**Binary:** `~/printing-press/library/consigliere/consigliere-pp-cli`
**Base URL:** `http://localhost:9104`
**Generated:** 2026-05-09

### Install

```bash
# Binary is already installed locally
~/printing-press/library/consigliere/consigliere-pp-cli --version
```

### Environment

No API key required. Base URL is local:

```bash
export CONSIGLIERE_BASE_URL=http://localhost:9104  # override if port changes
```

### MCP Server

```bash
~/printing-press/library/consigliere/consigliere-pp-cli mcp
```

Register in `~/.claude/settings.json` under `mcpServers.consigliere`.

### Command Examples

```bash
# List OSINT items
consigliere-pp-cli items list

# Run OSINT research task
consigliere-pp-cli research run --query "example query"

# List sources
consigliere-pp-cli sources list

# Check active runs
consigliere-pp-cli runs list

# Search skills catalog
consigliere-pp-cli skills list

# Market intelligence
consigliere-pp-cli market list

# Network nodes
consigliere-pp-cli network list
```
EOF
)"
```

---

## Chunk 2: Hermes CLI

**Services:** `http://localhost:9109/openapi.json` (hermes-adapter; covers both Hermes and Paperclip namespaces)
**Destination:** `~/printing-press/library/hermes/hermes-pp-cli`
**Skill install:** `~/.claude/skills/hermes/SKILL.md`

- [ ] **Step 1: Run Printing Press fast-path**

Invoke the `printing-press` skill with:

```
args: --spec http://localhost:9109/openapi.json --name hermes
```

Note: This spec covers the unified hermes-adapter which exposes both Hermes (backlog, dagu, memories, files, chat, a2a) and Paperclip (agents) namespaces. The generated CLI should have subcommands for all of them.

**Wait for shipcheck verdict before proceeding.**

- [ ] **Step 2: Verify binary exists and responds**

```bash
ls -lh ~/printing-press/library/hermes/hermes-pp-cli
~/printing-press/library/hermes/hermes-pp-cli --version
~/printing-press/library/hermes/hermes-pp-cli --help | head -20
```

Expected: binary exists, help output shows namespaces (backlog, dagu, paperclip, memories, files, chat, a2a).

- [ ] **Step 3: Install skill**

```bash
mkdir -p ~/.claude/skills/hermes
cp ~/printing-press/library/hermes/SKILL.md ~/.claude/skills/hermes/SKILL.md
echo "Skill installed:"
head -5 ~/.claude/skills/hermes/SKILL.md
```

- [ ] **Step 4: Smoke-test MCP server startup**

```bash
timeout 5 ~/printing-press/library/hermes/hermes-pp-cli mcp 2>&1 | head -30 || true
```

Expected: tool names appear before timeout.

- [ ] **Step 5: Verify tools-manifest.json**

```bash
cat ~/printing-press/library/hermes/tools-manifest.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Tools: {len(d.get(\"tools\",[]))}')"
```

- [ ] **Step 6: Document in wiki**

```bash
python3 /Users/jcords-macmini/projects/00_Governance/wiki/scripts/wiki.py set-section "CLI Tools/Hermes" "$(cat <<'EOF'
## Hermes CLI

**Binary:** `~/printing-press/library/hermes/hermes-pp-cli`
**Base URL:** `http://localhost:9109` (hermes-adapter; covers Hermes + Paperclip)
**Generated:** 2026-05-09

### Install

```bash
~/printing-press/library/hermes/hermes-pp-cli --version
```

### Environment

No API key required. Base URL is local.

### MCP Server

```bash
~/printing-press/library/hermes/hermes-pp-cli mcp
```

### Command Examples

```bash
# List backlog items
hermes-pp-cli backlog list

# Add a backlog item
hermes-pp-cli backlog add --title "My task"

# List Paperclip agents
hermes-pp-cli paperclip list

# Start a Dagu DAG
hermes-pp-cli dagu start --dag my-dag

# List memories
hermes-pp-cli memories list

# Chat
hermes-pp-cli chat --message "hello"

# A2A tasks
hermes-pp-cli a2a list
```
EOF
)"
```

---

## Chunk 3: Poster Engine CLI

**Services:** `http://localhost:9120/openapi.json`
**Destination:** `~/printing-press/library/poster-engine/poster-engine-pp-cli`
**Skill install:** `~/.claude/skills/poster-engine/SKILL.md`

- [ ] **Step 1: Run Printing Press fast-path**

Invoke the `printing-press` skill with:

```
args: --spec http://localhost:9120/openapi.json --name poster-engine
```

**Wait for shipcheck verdict before proceeding.**

- [ ] **Step 2: Verify binary exists and responds**

```bash
ls -lh ~/printing-press/library/poster-engine/poster-engine-pp-cli
~/printing-press/library/poster-engine/poster-engine-pp-cli --version
~/printing-press/library/poster-engine/poster-engine-pp-cli --help | head -20
```

Expected: binary exists, help output covers poster generation, layouts, export namespaces.

- [ ] **Step 3: Install skill**

```bash
mkdir -p ~/.claude/skills/poster-engine
cp ~/printing-press/library/poster-engine/SKILL.md ~/.claude/skills/poster-engine/SKILL.md
echo "Skill installed:"
head -5 ~/.claude/skills/poster-engine/SKILL.md
```

- [ ] **Step 4: Smoke-test MCP server startup**

```bash
timeout 5 ~/printing-press/library/poster-engine/poster-engine-pp-cli mcp 2>&1 | head -30 || true
```

- [ ] **Step 5: Verify tools-manifest.json**

```bash
cat ~/printing-press/library/poster-engine/tools-manifest.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Tools: {len(d.get(\"tools\",[]))}')"
```

- [ ] **Step 6: Document in wiki**

```bash
python3 /Users/jcords-macmini/projects/00_Governance/wiki/scripts/wiki.py set-section "CLI Tools/Poster Engine" "$(cat <<'EOF'
## Poster Engine CLI

**Binary:** `~/printing-press/library/poster-engine/poster-engine-pp-cli`
**Base URL:** `http://localhost:9120`
**Generated:** 2026-05-09

### Install

```bash
~/printing-press/library/poster-engine/poster-engine-pp-cli --version
```

### Environment

No API key required. Base URL is local.

### MCP Server

```bash
~/printing-press/library/poster-engine/poster-engine-pp-cli mcp
```

### Command Examples

```bash
# Generate a poster
poster-engine-pp-cli generate --topic "My Poster" --layout infographic

# List available layouts
poster-engine-pp-cli layouts list

# Export a poster
poster-engine-pp-cli export --id <poster-id> --format pdf

# Batch generation
poster-engine-pp-cli generate batch --input topics.json

# List past posters
poster-engine-pp-cli list
```
EOF
)"
```

---

## Chunk 4: MCP Registration + Wiki Overview

**Goal:** Wire all three CLIs into Claude Code as MCP servers and create the wiki overview page.

- [ ] **Step 1: Register MCP servers in `~/.claude/settings.json`**

Open `~/.claude/settings.json` and add to the `mcpServers` block (create the block if it doesn't exist):

```json
"mcpServers": {
  "consigliere":   { "command": "/Users/jcords-macmini/printing-press/library/consigliere/consigliere-pp-cli",     "args": ["mcp"] },
  "hermes":        { "command": "/Users/jcords-macmini/printing-press/library/hermes/hermes-pp-cli",               "args": ["mcp"] },
  "poster-engine": { "command": "/Users/jcords-macmini/printing-press/library/poster-engine/poster-engine-pp-cli", "args": ["mcp"] }
}
```

> **Warning:** If `mcpServers` already has entries, merge rather than replace — do not clobber existing servers.

Verify JSON is valid after editing:

```bash
python3 -m json.tool ~/.claude/settings.json > /dev/null && echo "JSON valid"
```

- [ ] **Step 2: Confirm MCP servers appear in Claude Code**

Restart Claude Code (or run `/mcp` in a new session) and confirm all three servers appear with status `connected`. If any server fails to start, check the binary path:

```bash
ls -lh ~/printing-press/library/consigliere/consigliere-pp-cli
ls -lh ~/printing-press/library/hermes/hermes-pp-cli
ls -lh ~/printing-press/library/poster-engine/poster-engine-pp-cli
```

- [ ] **Step 3: Create wiki overview page**

```bash
python3 /Users/jcords-macmini/projects/00_Governance/wiki/scripts/wiki.py set-section "CLI Tools/Overview" "$(cat <<'EOF'
## Internal CLIs — Overview

Three agent-native Go CLIs generated by the Printing Press (fast-path) targeting local FastAPI services. Each CLI exposes both a human-ergonomic CLI surface and an MCP server for agent use.

### Common Pattern

```bash
# Get help for any CLI
<slug>-pp-cli --help
<slug>-pp-cli <command> --help

# Start MCP server (auto-started by Claude Code via settings.json)
<slug>-pp-cli mcp

# Check binary version
<slug>-pp-cli --version
```

### CLIs

| CLI | Binary | Port | Namespaces |
|-----|--------|------|------------|
| Consigliere | `consigliere-pp-cli` | 9104 | osint, items, sources, runs, ai, research, skills, market, network, tasks |
| Hermes | `hermes-pp-cli` | 9109 | backlog, dagu, paperclip, memories, files, chat, a2a |
| Poster Engine | `poster-engine-pp-cli` | 9120 | generate, layouts, export |

### MCP Registration

All three are registered in `~/.claude/settings.json` under `mcpServers` and auto-start when Claude Code launches.

### Skills

Generated skills are installed at `~/.claude/skills/<slug>/SKILL.md` and auto-loaded in every Claude Code session.

### Regenerating a CLI

If the underlying API changes:

```bash
/printing-press --spec http://localhost:<port>/openapi.json --name <slug>
```

Then re-copy the new `SKILL.md` to `~/.claude/skills/<slug>/SKILL.md`.
EOF
)"
```

- [ ] **Step 4: Final verification summary**

Run this checklist and report results:

```bash
echo "=== Binary check ==="
ls ~/printing-press/library/consigliere/consigliere-pp-cli 2>/dev/null && echo "consigliere OK" || echo "consigliere MISSING"
ls ~/printing-press/library/hermes/hermes-pp-cli 2>/dev/null && echo "hermes OK" || echo "hermes MISSING"
ls ~/printing-press/library/poster-engine/poster-engine-pp-cli 2>/dev/null && echo "poster-engine OK" || echo "poster-engine MISSING"

echo "=== Skill check ==="
ls ~/.claude/skills/consigliere/SKILL.md 2>/dev/null && echo "consigliere skill OK" || echo "consigliere skill MISSING"
ls ~/.claude/skills/hermes/SKILL.md 2>/dev/null && echo "hermes skill OK" || echo "hermes skill MISSING"
ls ~/.claude/skills/poster-engine/SKILL.md 2>/dev/null && echo "poster-engine skill OK" || echo "poster-engine skill MISSING"

echo "=== MCP registration check ==="
python3 -c "
import json, os
s = json.load(open(os.path.expanduser('~/.claude/settings.json')))
mcp = s.get('mcpServers', {})
for k in ['consigliere', 'hermes', 'poster-engine']:
    print(k, 'registered' if k in mcp else 'MISSING')
"
```

Expected: all nine lines show OK or `registered`.

---

## Failure Reference

| Failure | Action |
|---------|--------|
| `go build` fails inside skill | Fix before proceeding — never ship broken binary |
| Shipcheck `verify` gate fails | Follow skill's one-fix-loop; if still failing, document gap and continue to next CLI |
| Service not running | Skip that CLI's chunk, note it, return after others complete |
| `tools-manifest.json` has 0 tools | Run `printing-press verify --dir ~/printing-press/library/<slug>` for diagnosis |
| MCP server fails to connect | Verify binary path in settings.json matches actual binary location |
| wiki.py `set-section` destructive warning | Expected — `set-section` replaces the section content. Run only once per CLI |
