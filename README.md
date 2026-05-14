# Claude Code Project Workflow

A baseline project scaffold and reference guides for running a full development workflow with Claude Code.

The setup centres on three plugin stacks:

- **[ECC (everything-claude-code)](https://github.com/affaan-m/everything-claude-code)** — language-specific code review, build, test, planning, and session skills across Java, TypeScript, Python, Go, Kotlin, Rust, Dart, Swift, and more
- **[Superpowers](https://github.com/anthropics/superpowers)** — architectural and critical-path skills: brainstorming, planning, TDD enforcement, systematic debugging, security review, and branch completion
- **[Karpathy Guidelines](https://github.com/forrestchang/andrej-karpathy-skills)** — coding discipline: think before coding, surgical changes, minimum implementation, goal-driven execution

Additional project-specific skills are layered on top as needed (e.g. Playwright for E2E, docker-dev for container workflows, pr-description for pull requests).

---

## Contents

| File / Directory | Purpose |
|-----------------|---------|
| [`project-setup-guide.md`](project-setup-guide.md) | One-time setup: global config, language rule sets, full skill and plugin reference, context budget |
| [`workflow-guide.md`](workflow-guide.md) | Day-to-day: session start, feature dev, code review, git workflow, skill quick reference |
| [`templates/`](templates/) | Project scaffold — copy to a new project and delete what you don't need |

---

## Using the Template

```bash
cp -r templates/.claude /path/to/new-project/
cp templates/CLAUDE.md  /path/to/new-project/CLAUDE.md
cp templates/Roadmap.md /path/to/new-project/Roadmap.md
```

Then:
1. Delete language rule sets under `.claude/rules/` that don't apply to your stack
2. Update `web/` path frontmatter in `.claude/rules/web/*.md` to match your frontend directory
3. Fill in the stack placeholders in `.claude/agents/security-reviewer.md`
4. Customise `.claude/references/docker-services.md` if the project uses Docker

See [`project-setup-guide.md`](project-setup-guide.md) for the full checklist.

---

## Project Knowledge Base — Graphify

For larger codebases, use **graphify** to generate a structural knowledge graph before searching raw files. This significantly reduces token usage on architecture and codebase questions.

The template `settings.json` includes a `PreToolUse` hook that fires on every `Glob` or `Grep` call: if `graphify-out/graph.json` exists, it injects a context hint pointing Claude to `graphify-out/GRAPH_REPORT.md` instead of scanning source files directly.

Usage pattern in `CLAUDE.md`:
```
## Graphify
Knowledge graph at graphify-out/.
- Before architecture or codebase questions: read graphify-out/GRAPH_REPORT.md
- If graphify-out/wiki/index.md exists: navigate it instead of reading raw files
- After modifying code files: rebuild graph via graphify watch
```

---

## Additional Cost and Token Optimization

The baseline setup is intentionally lean. For further token efficiency and cost reduction, consider these additional plugins and skills:

### [RTK](https://github.com/rtk-ai/rtk)

Retrieval-Thinking-Knowledge framework. Structures Claude's retrieval and reasoning steps to reduce wasted context on large, complex tasks. Useful on projects where Claude regularly scans many files or reasons across long histories.

### [Caveman](https://github.com/juliusbrussee/caveman)

Minimal-execution skill that handles boilerplate, find-and-replace, and simple mechanical tasks without triggering full planning or review cycles. Keeps the cheap tasks cheap. Pairs with ECC — use caveman for the trivial 30%, ECC for the complex 70%.

Both are referenced in the primary workflow (`CLAUDE.md` directive: "Defer to ecc skills and caveman for execution") but require separate installation.
