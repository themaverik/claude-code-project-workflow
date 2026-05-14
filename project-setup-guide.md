# Claude Project Setup Guide

Reference for configuring Claude Code on a new project. Most of the stack lives at user level and requires no per-project work. Only the language-specific rules and a project CLAUDE.md need to be added.

---

## User-Level Setup (Pre-configured — No Action Needed)

Everything below is already installed globally at `~/.claude/` and applies to every project automatically.

### CLAUDE.md (`~/.claude/CLAUDE.md`)

Always loaded. Contains:
- Primary workflow: ECC + Caveman + Graphify + Claude-mem
- Interaction style and coding behavior (Karpathy principles)
- Always-do rules (no `git push` without confirmation, commit freely, etc.)
- Token hygiene reminders
- Skill trigger table
- Code review strategy (ECC 80% / Superpowers 20%)

### Always-On Rules (`~/.claude/rules/common/`)

Loaded for every session regardless of file type:

| File | Purpose |
|---|---|
| `coding-style.md` | Immutability, KISS/DRY/YAGNI, naming, file size limits |
| `code-review.md` | Review process, severity levels, approval criteria |
| `security.md` | Pre-commit security checklist, secret management |
| `testing.md` | 80% coverage, TDD workflow, AAA pattern |
| `hooks.md` | Hook types, TodoWrite best practices |
| `patterns.md` | Repository pattern, API response format |
| `git-workflow.md` | Commit format, PR workflow |

Path-filtered (only load on source code files):

| File | Paths |
|---|---|
| `development-workflow.md` | `**/*.java`, `**/*.ts`, `**/*.tsx`, `**/*.js`, `**/*.py`, `**/*.go`, `**/*.rs` |
| `performance.md` | Same as above |

### Path-Triggered Rules (`~/.claude/rules/`)

| File | Triggers on |
|---|---|
| `doc-standards.md` | `**/*.md`, `docs/**` |
| `docker.md` | `infrastructure/**`, `docker-compose*.yml`, `**/Dockerfile` |
| `git-policy.md` | `.gitignore`, `.git/**`, `**/*.sh` |

### User-Level Skills (`~/.claude/skills/`)

Auto-triggered by description match:

| Skill | Trigger phrases |
|---|---|
| `project-resume` | "resume", "continue", "where were we", session start |
| `adr-manager` | "log this as ADR", "record this decision", "why did we choose X", tech stack choices |
| `doc-sync` | "sync docs", "consolidate docs", "update knowledge base" |
| `docker-dev` | "docker", "compose", "spin up", "bring up", "tear down" |
| `knowledge-ingest` | "ingest specs", "scan plans", "what does the spec say about X" |
| `knowledge-query` | "what did we decide about X", "any ADR for Y", "check past decisions" |
| `karpathy-guidelines` | Any coding, reviewing, or refactoring task |
| `pr-description` | "create a PR", "write a PR", "summarize changes for a pull request" |

### Global Hooks (`~/.claude/settings.json`)

Fires on every project:

| Hook | Trigger | Condition | Action |
|---|---|---|---|
| Post-edit checks | `Edit` or `Write` | `file_path` contains `werkflow/frontends/` | `npm run lint --silent` from portal root |
| Post-edit checks | `Edit` or `Write` | `file_path` matches `.env` | Blocks edit with error |
| Post-edit checks | `Edit` or `Write` | `file_path` matches `(services\|shared)/.*\.java` | Walks up to `pom.xml`, runs `mvn checkstyle:check -q` |
| Commit reminder | `Bash` | Command contains `git commit` | Reminds to update ROADMAP.md markers |

> Note: The lint and checkstyle hooks are werkflow-specific paths. Update these per project.

---

## New Project Setup

### 1. Create Project CLAUDE.md

Place at `<project-root>/CLAUDE.md`. Minimum required sections:

```markdown
# Claude Code Project Configuration

## Core Directives
1. **Token Efficiency First:** Use claude-mem for history, graphify for structure.
2. **Modular Execution:** Defer to ecc skills and caveman for execution.
3. **Highest Industry Standards:** Strict typing, clean architecture, modularity.
4. Read and update master Roadmap from/to `docs/Roadmap.md`. Do not commit this file.
5. [Define default code directories — e.g. `./services` for backend, `./frontend` for UI]

## Coding Behavior (Karpathy Principles)
> Full guidelines in `karpathy-guidelines` skill.
* **Think before coding:** State assumptions explicitly. Ask before guessing.
* **Simplicity first:** Minimum code. No speculative features or abstractions.
* **Surgical changes:** Touch only what the task requires. Match existing style.
* **Goal-driven execution:** Define verifiable success criteria before starting.

## Code Review Strategy
- **Always first:** `ecc:<language>-review` (~5% cost)
- **Only when needed:** `/superpowers:requesting-code-review`, `/superpowers:security-auditor` (~15% cost)
- **When to use Superpowers:** Auth, payments, data handling, major refactors, pre-production

## Token Management
* Suggest `/compact` after 10 turns or phase completion.
```

### 2. Create `.claude/` Directory Structure

```
<project-root>/
  .claude/
    rules/          ← language rule sets (see below)
    skills/         ← copy relevant skills from ~/.claude/skills/
```

### 3. Copy Skills

Copy the full user-level skills set as a starting point:

```bash
cp -r ~/.claude/skills/* .claude/skills/
```

Remove any skills not relevant to the project (e.g. remove `docker-dev` for non-containerised projects).

### 4. Install Language Rule Sets

Copy from the ECC plugin cache. Always include `common/` at user level (already done). Add language-specific sets at project level:

**ECC cache location:**
```
~/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.10.0/rules/
```

```bash
BASE=~/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.10.0/rules
# Copy whichever sets apply — mix and match
cp -r "$BASE/<language>" .claude/rules/<language>
```

---

## Language Rule Sets Reference

All sets contain: `coding-style.md`, `hooks.md`, `patterns.md`, `security.md`, `testing.md`

### Java / Spring Boot

```bash
cp -r "$BASE/java" .claude/rules/java
```

Path trigger: `**/*.java`, `**/pom.xml`, `**/build.gradle`  
Pair with: `java-services.md` custom rule for your specific service paths

### TypeScript + Web (React / Next.js)

```bash
cp -r "$BASE/typescript" .claude/rules/typescript
cp -r "$BASE/web"        .claude/rules/web
```

Path triggers:
- `typescript/`: `**/*.ts`, `**/*.tsx`, `**/*.js`, `**/*.jsx`
- `web/`: no default — **must add path frontmatter** pointing to your frontend dir:

```bash
for f in .claude/rules/web/*.md; do
  head -1 "$f" | grep -q "^---" || \
  { tmp=$(mktemp)
    printf -- "---\npaths:\n  - \"<your-frontend-dir>/**/*.tsx\"\n  - \"<your-frontend-dir>/**/*.ts\"\n  - \"<your-frontend-dir>/**/*.css\"\n---\n" | cat - "$f" > "$tmp" && mv "$tmp" "$f"; }
done
```

### Python

```bash
cp -r "$BASE/python" .claude/rules/python
```

Path trigger: `**/*.py`, `**/*.pyi`

### Go

```bash
cp -r "$BASE/golang" .claude/rules/golang
```

Path trigger: `**/*.go`, `**/go.mod`, `**/go.sum`

### Rust

```bash
cp -r "$BASE/rust" .claude/rules/rust
```

Path trigger: `**/*.rs`

### Kotlin / Android

```bash
cp -r "$BASE/kotlin" .claude/rules/kotlin
```

Path trigger: `**/*.kt`, `**/*.kts`

### Swift / iOS

```bash
cp -r "$BASE/swift" .claude/rules/swift
```

Path trigger: `**/*.swift`

### C++

```bash
cp -r "$BASE/cpp" .claude/rules/cpp
```

Path trigger: `**/*.cpp`, `**/*.h`, `**/*.cc`, `**/*.hpp`

### C#

```bash
cp -r "$BASE/csharp" .claude/rules/csharp
```

Path trigger: `**/*.cs`, `**/*.csproj`

### PHP / Laravel

```bash
cp -r "$BASE/php" .claude/rules/php
```

Path trigger: `**/*.php`

### Dart / Flutter

```bash
cp -r "$BASE/dart" .claude/rules/dart
```

Path trigger: `**/*.dart`

---

## On-Demand Skills (Not Always Loaded — Invoke Manually)

These are intentionally excluded from auto-loading. Invoke via `/skill-name` or the Skill tool only when needed.

### Superpowers (Token-Hungry — Use Surgically)

Installed at: `~/.claude/plugins/cache/claude-plugins-official/superpowers/`  
Cost: ~10–15% of a session budget per invocation. Use at most 1–2 times per session.

| Skill | Invoke as | When to use |
|---|---|---|
| `brainstorming` | `/superpowers:brainstorming` | Before building any new feature — explores intent before touching code |
| `writing-plans` | `/superpowers:writing-plans` | When you have a spec and need a structured multi-step plan |
| `requesting-code-review` | `/superpowers:requesting-code-review` | Auth, payments, sensitive data handling, pre-production |
| `systematic-debugging` | `/superpowers:systematic-debugging` | Any bug or test failure before proposing a fix |
| `verification-before-completion` | `/superpowers:verification-before-completion` | Before claiming work is done or creating a PR |
| `finishing-a-development-branch` | `/superpowers:finishing-a-development-branch` | When implementation is complete — guides merge/PR/cleanup decision |
| `test-driven-development` | `/superpowers:test-driven-development` | Enforces write-tests-first on complex features |
| `writing-skills` | `/superpowers:writing-skills` | When creating a new skill |

> Default code review strategy: ECC reviewers first (`ecc:java-review`, `ecc:typescript-review` etc.), Superpowers only for security-critical or pre-production code.

### Agents Vault (Non-Coding — Load When Needed)

Archived at: `~/.claude/agents-vault/`  
Load with: `claude --add-dir ~/.claude/agents-vault`

| Agent | Purpose |
|---|---|
| `product-strategy-advisor` | Product direction, feature prioritisation |
| `research-synthesizer` | Synthesising research from multiple sources |
| `market-research-analyst` | Competitor analysis, market sizing |
| `competitive-intelligence-analyst` | Deep competitor research |
| `audio-mixer` | Audio production tasks |
| `audio-quality-controller` | Audio QA |
| `video-editor` | Video editing workflows |

---

## Skill Trigger Examples

Skills activate in two ways: **auto-triggered** (Claude detects the intent from natural language) or **manual invoke** (explicit `/skill-name` command).

---

### Auto-Triggered Skills

These fire from natural conversation — no slash command needed.

#### `project-resume`
```
"Let's continue where we left off"
"Where were we?"
"Resume the session"
"Pick up from last time"
"What's next on the roadmap?"
```

#### `adr-manager`
```
"Log this as an ADR"
"Record this decision"
"Why did we choose Postgres over MySQL?"
"Document this trade-off"
"Save this architecture decision"
"We decided to use Kafka for event streaming"   ← silent auto-capture on tech stack choices
```

#### `doc-sync`
```
"Sync the docs"
"Update the knowledge base"
"Consolidate all docs"
"Regenerate Knowledge-Base.md"
```
Also fires automatically at session end when ADRs were written or `knowledge-ingest` was run.

#### `docker-dev`
```
"Spin up the containers"
"Bring up the stack"
"Tear down docker"
"Check if the services are running"
"Restart the admin service"
"Show me the logs for the engine container"
"Rebuild and restart business service"
```

#### `knowledge-ingest`
```
"Ingest the connector spec"
"Scan the feature plan for decisions"
"What does the spec say about DOA?"
"Load the ADR docs"
"Read the superpowers plan and extract constraints"
```

#### `knowledge-query`
```
"What did we decide about tenant isolation?"
"Any ADR for the auth approach?"
"Check past decisions on the database schema"
"Does the KB say anything about rate limiting?"
"Any constraints on the connector design?"
"What patterns did we establish for Flyway migrations?"
```

#### `karpathy-guidelines`
Fires silently on any coding task — no explicit trigger needed. Active whenever writing, reviewing, or refactoring code.

#### `pr-description`
```
"Write the PR description"
"Create a PR for this branch"
"Summarise the changes for a pull request"
"Draft a pull request"
```

#### `adr-manager` (spec scan mode)
```
"/adr-ingest"
"Scan the specs for decisions"
"Extract decisions from the roadmap"
```

---

### Manual Invoke — ECC Skills

Use `/ecc:<skill>` or invoke via the Skill tool. These are the daily execution skills.

#### Code Review
```
/ecc:java-review          ← after writing Java services
/ecc:typescript-review    ← after writing frontend or API code
/ecc:python-review        ← after writing Python scripts
/ecc:go-review            ← after writing Go services
/ecc:kotlin-review        ← after writing Kotlin code
/ecc:rust-review          ← after writing Rust code
```

#### Build and Test
```
/ecc:java-build           ← when Maven/Gradle build fails
/ecc:kotlin-build
/ecc:go-build
/ecc:rust-build
/ecc:flutter-build
/ecc:gradle-build

/ecc:go-test
/ecc:kotlin-test
/ecc:rust-test
/ecc:flutter-test
```

#### Planning and Feature Dev
```
/ecc:plan                 ← structured implementation plan for a feature
/ecc:feature-dev          ← full feature development lifecycle
/ecc:tdd                  ← enforce write-tests-first
/ecc:prp-prd              ← generate a PRD from a prompt
/ecc:prp-plan             ← turn a PRD into a phased plan
/ecc:prp-implement        ← execute a plan phase by phase
/ecc:prp-commit           ← commit at phase completion
```

#### Session and State
```
/ecc:save-session         ← save session state before compacting
/ecc:resume-session       ← restore session state after compacting
/ecc:sessions             ← list saved sessions
/ecc:context-budget       ← check remaining context budget
```

#### Code Quality
```
/ecc:code-review          ← general code review
/ecc:security-review      ← security-focused review
/ecc:verify               ← verify implementation against requirements
/ecc:docs                 ← generate or update documentation
```

---

### Manual Invoke — Superpowers (Token-Hungry)

Use only for security-critical, architectural, or pre-production work. Each invocation costs ~10–15% of session budget.

#### Before Building
```
/superpowers:brainstorming
→ "I want to build a tenant isolation system, let's brainstorm first"
→ "Before we implement the DOA approval flow, let's explore the design"

/superpowers:writing-plans
→ "I have the connector spec, write a plan before we touch any code"
→ "Turn these requirements into a phased implementation plan"
```

#### During Development
```
/superpowers:test-driven-development
→ "Let's build the new assignment panel TDD"
→ "Enforce write-tests-first for this auth refactor"

/superpowers:systematic-debugging
→ "The Keycloak token claim is missing realm roles, debug this properly"
→ "Flyway checksum repair keeps failing, walk through it systematically"
```

#### Before Committing or Merging
```
/superpowers:verification-before-completion
→ "Before I mark this done, verify everything actually works"
→ "Run verification before we create the PR"

/superpowers:requesting-code-review
→ "Review the new auth middleware before we ship"
→ "Security review on the JWT token extraction logic"
→ "Review the payment connector — it touches financial data"

/superpowers:finishing-a-development-branch
→ "Implementation is done and tests pass, how do we wrap up this branch?"
→ "Guide me through closing out this feature branch"
```

#### Architecture and Git Workflows
```
/superpowers:writing-skills
→ "Create a new skill for our Flyway migration workflow"

/superpowers:using-git-worktrees
→ "Set up a worktree for this parallel feature"

/superpowers:dispatching-parallel-agents
→ "Run security analysis and performance review in parallel"
```

---

### Manual Invoke — Claude-mem

```
/claude-mem:do             ← general memory operation ("remember that...")
/claude-mem:mem-search     ← search memory for a topic
/claude-mem:smart-explore  ← deep explore memory around a theme
/claude-mem:knowledge-agent ← answer a question using memory as context
/claude-mem:make-plan      ← create a plan stored in memory
/claude-mem:timeline-report ← chronological report of project activity
```

---

### Manual Invoke — Archived Agents (Non-Coding)

Load the vault first: `claude --add-dir ~/.claude/agents-vault`

```
"Analyse our competitors in the workflow automation space"   → competitive-intelligence-analyst
"Research the BPM market sizing"                             → market-research-analyst
"Help me think through the product roadmap priorities"       → product-strategy-advisor
"Synthesise these five research papers into a summary"       → research-synthesizer
```

---

## Quick-Start Checklist for a New Project

```
[ ] Create CLAUDE.md at project root (use template above)
[ ] mkdir -p .claude/rules .claude/skills
[ ] cp -r ~/.claude/skills/* .claude/skills/
[ ] Copy relevant ECC rule sets from cache into .claude/rules/
[ ] Add path frontmatter to web/ rules if using a frontend (no default paths)
[ ] Add project-specific custom rules (e.g. service paths, naming conventions)
[ ] Update hooks in ~/.claude/settings.json if lint/checkstyle paths differ
[ ] Run /project-resume to orient the first session
```

---

## Context Budget Reference

Approximate token cost at session start (before any file is opened):

| Scenario | ~Tokens |
|---|---|
| Idle baseline (user + project CLAUDE.md + memory index) | ~9,300 |
| Editing a Java file (+ java rules + flat rules) | ~14,300 |
| Editing a frontend file (+ ts + web + flat rules) | ~16,000 |

Keep `common/` rules lean — they load on every session. Language rules are path-triggered and only load when editing matching files.
