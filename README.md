# Claude Code Project Workflow

A reference guide and reusable template for all projects developed with Claude Code. Documents the complete development cycle from project setup through session management, coding, testing, deployment, and context optimization.

## Quick Start

1. Read this guide to understand the full workflow
2. Copy the templates from `templates/` into your new project
3. Configure global skills (one-time setup)
4. Start a session and follow the workflow

## Contents

```
README.md                          This file
workflow-guide.md                  Complete development cycle guide
templates/
  CLAUDE.md                        Project-level CLAUDE.md template
  ROADMAP.md                       Task tracking template
  .claude/
    references/
      documentation-standards.md   Documentation conventions (on-demand)
      git-conventions.md           Branch and commit conventions (on-demand)
      docker-services.md           Project docker config template (on-demand)
    agents/
      code-reviewer.md             Code review agent template
      security-reviewer.md         Security audit agent template
```

## Global Setup (one-time)

These files live outside any project and apply to all Claude Code sessions:

```
~/.claude/CLAUDE.md                Global Claude behavior rules
~/.claude/skills/
  project-resume/SKILL.md          Session resume and task tracking
  docker-dev/SKILL.md              Docker pre-flight and data protection
```
