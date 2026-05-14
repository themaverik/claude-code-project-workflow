# Claude Code Project Workflow

Reference templates and guides for setting up and running Claude Code projects.

## Contents

| File / Directory | Purpose |
|-----------------|---------|
| [`project-setup-guide.md`](project-setup-guide.md) | One-time setup: global config, language rule sets, skill reference, context budget |
| [`workflow-guide.md`](workflow-guide.md) | Day-to-day: session start, feature dev, code review, git workflow, skill quick reference |
| [`templates/`](templates/) | Project scaffold — copy to a new project and delete what you don't need |

## Using the Template

```bash
cp -r templates/ /path/to/new-project/.claude
cp templates/CLAUDE.md /path/to/new-project/CLAUDE.md
cp templates/Roadmap.md /path/to/new-project/Roadmap.md
```

Then:
1. Delete language rule sets under `.claude/rules/` that don't apply
2. Update the `web/` path frontmatter in `.claude/rules/web/*.md` to match your frontend directory
3. Fill in the stack placeholders in `.claude/agents/security-reviewer.md`
4. Customise `.claude/references/docker-services.md` if the project uses Docker

See [`project-setup-guide.md`](project-setup-guide.md) for the full checklist.
