---
name: Council Memory Hygiene
description: Audit and clean OpenClaw memory files. Removes stale facts, contradicted entries, and outdated context from council_memory.json, MEMORY.md, and daily memory files. Use when memory has accumulated junk, agents are citing old numbers, or before a major project phase change. Run by Samael on Sunday cron.
---

# Council Memory Hygiene

Keeps agent memory lean, accurate, and contradiction-free.

## What Gets Cleaned

1. **Shared Council Memory** (`council_memory.json`) — stale facts, outdated numbers
2. **Daily memory files** (`memory/YYYY-MM-DD.md`) — files older than 30 days get archived
3. **Self-improving HOT tier** (`~/self-improving/memory.md`) — demote unused patterns
4. **Vault facts** — contradict flags from Samael's contradiction detector

## Hygiene Commands

### Audit (read-only, safe)
```bash
# Show all facts in shared memory
curl -s https://64-227-110-70.sslip.io/council-api/council-memory \
  -H "x-council-token: TOKEN" | python3 -m json.tool

# Show all unresolved contradictions
curl -s https://64-227-110-70.sslip.io/council-api/contradictions/all/unresolved \
  -H "x-council-token: TOKEN" | python3 -m json.tool
```

### Prune stale council memory facts
Samael reviews facts older than 14 days and asks Lucy to confirm or archive them.
Tag for removal: `[MEMORY_ARCHIVE: entity_name]` in any agent response.

### Archive old daily memory files
```bash
# Files older than 30 days go to memory/archive/
find ~/.openclaw/workspace/memory/ -name "*.md" -mtime +30 \
  -exec mv {} ~/.openclaw/workspace/memory/archive/ \;
```

### Self-improving tier decay
Review `~/self-improving/memory.md` — entries unused 30 days move to `domains/` or `archive/`.

## Samael's Sunday Hygiene Protocol

Every Sunday 20:00 UTC, Samael runs:
1. Read all facts in `council_memory.json`
2. Flag facts older than 7 days that haven't been referenced
3. Check against contradiction detector — remove contradicted facts
4. Send hygiene report to Telegram: what was kept, what was archived, what's stale

## Rules

- Never delete without logging what was removed and why
- Contradicted facts → archive with both versions + Samael's resolution note  
- Preferences and rules (not facts) never expire unless Lucy explicitly removes them
- Always preserve: OCR accuracy history, spend history, agent correction logs

## What NOT to Clean

- Correction logs (permanent learning record)
- Strategic decisions (permanent decision registry)
- Client/bookkeeper names and contact info
- Any entry tagged `[PERMANENT]`
