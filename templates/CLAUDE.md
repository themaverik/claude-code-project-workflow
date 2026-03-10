# Project Development Guidelines

This file contains project-level instructions that apply to every session in this project.

## Documentation Standards

For full documentation conventions, read `.claude/references/documentation-standards.md`.

**Critical rules (always apply):**
- No icons or emojis in documentation or code unless explicitly stated
- Kebab-case for documentation file names (e.g. `git-conventions.md`)
- Markdown format for all code documentation

---

## 1. Git Policy

For branch naming conventions, commit message format, and task status markers, read `.claude/references/git-conventions.md`.

**Critical rules (always apply):**
- No Claude references in commits. Use `themaverick` as the commit author name.
- **NEVER run `git push` automatically.** Always ask the user before pushing to any remote.
- Follow the official style guide for your language
- Use linters and formatters appropriate for the tech stack

---

## 2. Code Style Guidelines

### General Principles

1. **Readability over cleverness**: Write code that is easy to understand
2. **Consistency**: Follow existing patterns in the codebase
3. **DRY (Don't Repeat Yourself)**: Extract common logic into reusable functions
4. **SOLID principles**: Apply when designing classes and modules
5. **Meaningful names**: Use descriptive variable, function, and class names

### Naming Conventions

**Variables**:
- Follow language-specific conventions (camelCase for Java/JS/TS, snake_case for Python, etc.)
- Use UPPER_SNAKE_CASE for constants: `MAX_RETRY_COUNT`, `API_BASE_URL`
- Boolean variables should be prefixed with `is`, `has`, `should`: `isValid`, `hasPermission`

**Functions/Methods**:
- Use verbs for function names: `calculateTotal()`, `validateInput()`
- Keep functions small and focused on a single responsibility
- Avoid side effects when possible

**Classes**:
- Use PascalCase
- Name classes after nouns that represent their responsibility

**Files**:
- Follow language conventions (camelCase for JS/TS, snake_case for Python, etc.)
- Match filename to primary export/class name

### Code Organization

1. **File structure**: Group related functionality, separate concerns, keep files under 300 lines
2. **Import order**: Standard library first, third-party second, local last. Alphabetize within groups.
3. **Method order**: Public before private, related functions grouped, important functions near top

### Comments and Documentation

- Explain WHY, not WHAT (code should be self-documenting)
- Document complex algorithms or business logic
- Add TODO/FIXME comments with context
- Remove commented-out code before committing

### Error Handling

1. **Always handle errors**: Don't let errors fail silently
2. **Use specific error types**: Create custom error classes when needed
3. **Provide context**: Include relevant information in error messages
4. **Log appropriately**: Use proper logging levels (debug, info, warn, error)

### Testing Standards

1. Write minimal unit tests for new features, bug fixes, and critical business logic
2. Follow testing conventions appropriate for the tech stack

### Security Best Practices

1. **Never commit secrets**: Use environment variables for sensitive data
2. **Validate input**: Always sanitize user input
3. **Use parameterized queries**: Prevent SQL injection
4. **Use latest dependencies for new projects**: DON'T auto-update existing projects. ASK the user for high/critical security issues.
5. **Follow principle of least privilege**: Grant minimum necessary permissions

---

## 3. On-Demand Skills and Agents

These are loaded only when their trigger conditions are met, not on every message.

<!-- Customize this table for your project -->

| Trigger | What to Load |
|---------|-------------|
| New feature task | Brainstorm before coding (project-resume skill R3) |
| Docker / container task | Global docker-dev skill + `.claude/references/docker-services.md` for project-specific services |
| Browser automation task | `~/.agents/skills/browser-use/SKILL.md` |
| Code review | `.claude/agents/code-reviewer.md` or `/review` or `/code-review:code-review` |
| Security audit | `.claude/agents/security-reviewer.md` |
| Session start / resume | project-resume skill -- handles session brief, branch check, ROADMAP tracking |
| Git commit / branching | `.claude/references/git-conventions.md` |
| Creating / editing docs | `.claude/references/documentation-standards.md` |
