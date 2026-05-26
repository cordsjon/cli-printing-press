# Handover: Internal CLIs via Printing Press

**Date:** 2026-05-09
**Session stopped at:** Chunk 0 pre-flight (CRITICAL session guard — 40 tool calls)
**Plan:** `docs/plans/2026-05-09-internal-clis-generate-plan.md`

## What's Done

### Pre-flight status

| Check | Status | Notes |
|-------|--------|-------|
| printing-press binary | **BLOCKED** — needs install | Downloaded to `/tmp/printing-press` (darwin/arm64 v4.2.0) |
| consigliere (9104) | ✅ Running | Title: "Consigliere" |
| hermes (9109) | ✅ Running | Title: "FastAPI" |
| poster-engine (9120) | ✅ Running | Title: "Poster Engine" |
| wiki.py | Not checked yet | Path: `00_Governance/wiki/scripts/wiki.py` |

## First Task in New Session

Install the printing-press binary before anything else:

```bash
# Binary already downloaded to /tmp/printing-press
sudo cp /tmp/printing-press /usr/local/bin/printing-press
sudo chmod +x /usr/local/bin/printing-press
printing-press --version
# Expected: 4.2.0
```

If `/tmp/printing-press` is gone (tmp cleared), re-download:

```bash
curl -sL https://github.com/mvanhorn/cli-printing-press/releases/download/v4.2.0/cli-printing-press_darwin_arm64.tar.gz | tar xz -C /tmp
sudo cp /tmp/printing-press /usr/local/bin/printing-press
sudo chmod +x /usr/local/bin/printing-press
```

Then resume pre-flight step 3 (wiki.py check) and proceed to Chunk 1.

## Remaining Work

Follow the plan verbatim. For each CLI chunk (1, 2, 3):
1. Invoke `Skill("printing-press", "--spec http://localhost:<port>/openapi.json --name <slug>")`
2. After skill completes: copy SKILL.md, smoke-test MCP, wiki doc
3. Then Chunk 4: MCP registration in `~/.claude/settings.json` + wiki overview

## Key Facts

- All three services confirmed running on ports 9104 / 9109 / 9120
- Binary version matches local repo exactly (both v4.2.0) — no version skew
- Go is NOT installed on this machine; always use the pre-built darwin/arm64 release binary
- The `printing-press` skill is the primary interface — invoke via `Skill` tool, not as a bash command
