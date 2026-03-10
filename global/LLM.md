# Global Development Guidelines

Universal instructions for AI-assisted development across all LLM tools.

> Project-level `LLM.md` or tool-specific files (e.g., `CLAUDE.md`) override these guidelines.

---

## Documentation Standards

- **No icons/emojis**: Avoid in all documentation. Exception: status indicators (✅ done, 🚧 in progress, ⏳ not started, ❌ declined).
- **File naming**: Title-Case for markdown files (e.g., `Api-Reference.md`). Exception: `README.md`.
- **Format**: Markdown unless stated otherwise.

---

## 1. Repository Etiquette

### Branch Naming

kebab-case with prefix — default main branch is `main`:

| Type | Pattern |
|------|---------|
| Feature | `feature/<description>` |
| Bug fix | `fix/<description>` |
| Hotfix | `hotfix/<description>` |
| Refactor | `refactor/<scope>` |
| Docs | `docs/<topic>` |
| Release | `release/<version>` |

### Gitignore

Create `.gitignore` before first commit if missing:

```gitignore
# LLM configurations
.claude/
.chatgpt/
.grok/
.deepseek/
.gemini/
LLM.md
CLAUDE.md
CHATGPT.md
GROK.md
DEEPSEEK.md
GEMINI.md

# Dependencies & build
node_modules/
dist/
build/
venv/
.venv/
site/
target/
vendor/

# IDE
.idea/
.vscode/
*.iml

# OS
.DS_Store
Thumbs.db

# Vim
*.swp
*.swo

# Environment & logs
.env*
!.env.example
*.log
/logs/

# Backups & patches
*.backup
*.bkp
*.orig
*.rej
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

Common types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

- Subject: max 50 chars, imperative mood, no period
- Body: explain what and why (not how); use bullet points for multiple changes
- Footer: reference issues (e.g., `Closes #123`)

### Pull Requests

- Title follows commit message format (e.g., `feat(payments): integrate Stripe gateway`)
- Rebase on latest `main`, resolve conflicts, and self-review before opening
- Keep PRs focused and small; request at least one reviewer

**Description template**:

```markdown
## Summary
Brief description of changes

## Changes
- Change 1
- Change 2

## Testing
How to test and what cases are covered

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

### Git Workflow

1. `git pull --rebase` before pushing
2. Small, logical commits frequently
3. Never force push to `main` or `develop`
4. Delete merged branches

---

## 2. Code Style

### Principles

1. **Readability over cleverness**
2. **Consistency**: Follow existing codebase patterns
3. **DRY**: Extract common logic into reusable functions
4. **SOLID**: Apply when designing classes/modules
5. **Avoid premature optimization**: Profile before optimizing; consider Big O only where it matters

### Naming

| Target | Convention | Notes |
|--------|-----------|-------|
| Variables | Language convention (camelCase / snake_case) | Booleans: prefix `is`, `has`, `should`, `can` |
| Constants | `UPPER_SNAKE_CASE` | |
| Functions | Verb-first: `fetchUser()`, `validateInput()` | Single responsibility, minimal side effects |
| Classes | `PascalCase` | Name after the responsibility |
| Files | Match primary export/class | Follow language conventions |

### Organization

- Group related functionality; separate concerns (models, views, controllers)
- Target under 300 lines per file
- Import order: standard library → third-party → local (alphabetize within groups)
- Method order: public → private; important functions near top

### Comments

- Explain **why**, not what
- Document complex algorithms and business logic
- Use `TODO`/`FIXME` with context
- Remove commented-out code before committing

### Error Handling

- Never fail silently
- Use specific error types; include context in messages
- Log at appropriate level: `debug`, `info`, `warn`, `error`

### Testing

- Write tests for all new features, bug fixes (regression), and critical business logic
- Use descriptive names: `test_user_cannot_login_with_invalid_password`
- Structure: Arrange → Act → Assert

### Security

1. Never commit secrets; use environment variables
2. Validate and sanitize all user input
3. Use parameterized queries (prevent SQL injection)
4. New projects: use latest stable dependency versions
5. Existing projects: ask before updating dependencies with known vulnerabilities
6. Follow principle of least privilege

---

## 3. API Standards

- URL path versioning: `/api/v1/resource`
- Consistent JSON response structure with meaningful error codes
- Follow REST conventions for HTTP methods and status codes
- Document breaking changes in changelog; maintain backwards compatibility where possible

---

## 4. Database Conventions

- Migration naming: `V<version>__<description>.sql` (e.g., `V1.0.1__add_email_index.sql`)
- Always create migrations for schema changes; never modify existing migrations in production
- Include rollback scripts when possible

---

## 5. Logging

Use structured logging (JSON preferred):

- `timestamp`: ISO 8601
- `level`: `debug` | `info` | `warn` | `error`
- `message`: Human-readable description
- `correlationId`: Request tracing ID
- `context`: Additional metadata

---

## 6. Dependency Management

- Pin exact versions in lock files; use version ranges cautiously in manifests
- Document major version upgrade decisions
- Prioritize patching security vulnerabilities

---

## 7. Language-Specific Guidelines

**JavaScript/TypeScript**: `const` by default, `let` when needed, no `var`; async/await over promises; TS strict mode, avoid `any`.

**Python**: PEP 8; type hints on function signatures; virtual environments always.

**Java/Kotlin**: Google Java Style or Kotlin conventions; prefer immutability; use dependency injection.

**All languages**: Use linters and formatters (ESLint/Prettier, Black, Spotless); configure format-on-save.
