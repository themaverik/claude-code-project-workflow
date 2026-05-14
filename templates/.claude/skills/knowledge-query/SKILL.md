---
name: knowledge-query
description: >
  Queries Knowledge-Base.md for past decisions, constraints, domain rules,
  or spec details. Triggers on "what did we decide about X", "any ADR for Y",
  "check past decisions on Z", "does the KB say anything about X",
  "any constraints on Y", "/knowledge-query [topic]". Reads only relevant
  KB sections — never the full file. Checks for contradictions before answering.
user-invocable: true
disable-model-invocation: false
allowed-tools: Read Bash(grep *) Bash(awk *) Bash(ls *)
---

# Knowledge Query

Targeted read of docs/Knowledge-Base.md. Never loads full file.

## Step 1 — Check KB and section map

```!
if [ -f "docs/Knowledge-Base.md" ]; then
  grep -n "^## " docs/Knowledge-Base.md 2>/dev/null
  echo "---"
  grep "Last synced:" docs/Knowledge-Base.md 2>/dev/null | head -1
else
  echo "KB not found — run /doc-sync to generate"
fi
```

## Step 2 — Targeted section read

If $ARGUMENTS empty: read sections 2, 5, 6 (ADRs, Current State, Open Decisions).
If $ARGUMENTS has a topic: grep for topic first, then read only matching sections.

```!
TOPIC="$ARGUMENTS"
KB="docs/Knowledge-Base.md"
[ ! -f "$KB" ] && exit 0
if [ -z "$TOPIC" ]; then
  awk '/<!-- adr-registry -->/{p=1} /<!-- domain-knowledge -->/{p=0} p{print}' "$KB" | head -50
  awk '/<!-- current-state -->/{p=1} /<!-- open-decisions -->/{p=0} p{print}' "$KB" | head -30
  awk '/<!-- open-decisions -->/{p=1} /<!-- tech-debt -->/{p=0} p{print}' "$KB" | head -20
else
  grep -n -i "$TOPIC" "$KB" | head -30
fi
```

## Step 3 — Answer format

```
Knowledge Query: <topic>
------------------------------------------------------------
Relevant ADRs    : ADR-NNN <title> [Accepted/Proposed]
Constraints      : <hard limits>
Domain rules     : <business rules from specs>
Open questions   : <unresolved items>
Source           : <ADR file | spec file | inferred>
------------------------------------------------------------
Confidence: high (ADR) | medium (spec) | low (inferred)
```

## Step 4 — Contradiction check

If query is "can we do X" or "should we use Y":
- Check ADR registry for conflicting **Accepted** decisions
- If conflict found: "⚠️ Conflicts with ADR-NNN: <title>"
- If none: "No prior decisions constrain this"

## Step 5 — Gap detection

If topic has no KB entry:
- Check `docs/adr/` for relevant unindexed files
- `grep -rl <topic> docs/` to find spec files not yet ingested
- Suggest: "Run /knowledge-ingest to extract from [file]" if found
