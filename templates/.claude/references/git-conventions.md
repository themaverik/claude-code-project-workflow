# Git Conventions

Reference file for branch naming and commit message conventions.
Read this file when creating branches or composing commit messages.

## Branch Naming Conventions

Follow these patterns for branch names:

- **Feature branches**: `feature/<short-description>`
  - Example: `feature/user-authentication`
- **Bug fixes**: `fix/<issue-description>`
  - Example: `fix/login-timeout`
- **Hotfixes**: `hotfix/<critical-issue>`
  - Example: `hotfix/security-patch`
- **Refactoring**: `refactor/<scope>`
  - Example: `refactor/database-layer`
- **Documentation**: `docs/<topic>`
  - Example: `docs/api-endpoints`
- **Experimental/Research**: `experiment/<feature-name>`
  - Example: `experiment/new-algorithm`

**Rules**:
- The default main branch name will be `main` unless specified
- Use lowercase letters and hyphens (kebab-case)
- Keep names concise but descriptive
- Avoid special characters except hyphens and forward slashes

## Commit Message Style

Follow **Conventional Commits** specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring without changing functionality
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `build`: Changes to build system or dependencies
- `ci`: Changes to CI/CD configuration
- `chore`: Other changes that don't modify src or test files
- `revert`: Reverting a previous commit

**Examples**:
```
feat(auth): add OAuth2 login support

fix(api): resolve timeout issue in user endpoint

docs(readme): update installation instructions

refactor(database): optimize query performance
```

**Rules**:
- Use lowercase for type and scope
- Subject line max 50 characters
- Use imperative mood ("add" not "added" or "adds")
- Don't end subject with a period
- Separate subject from body with blank line
- Body should explain what and why, not how
- Reference issue numbers in footer when applicable
- For multiple commits, keep point wise short descriptions for clarity

## Task Status Markers

Use these markers consistently in `ROADMAP.md` for all tasks:

| Marker | Status | Rule |
|--------|--------|------|
| `[ ]` | Pending | Not yet started |
| `[~]` | In Progress | Started but incomplete -- always note what remains |
| `[x]` | Completed | Done and committed -- always note the commit hash |
| `[!]` | Blocked | Cannot proceed -- always note the reason and what resolves it |

Example:
```markdown
- [x] Set up project structure *(commit: abc1234)*
- [~] Implement user auth *(login endpoint done, signup pending)*
- [ ] Create dashboard UI
- [!] Deploy to staging *(blocked: waiting for AWS credentials)*
```
