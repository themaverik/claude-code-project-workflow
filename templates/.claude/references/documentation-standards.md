# Documentation Standards

Reference file for documentation conventions across the project.
Read this file when creating or editing documentation, README files, or code comments.

## General Rules

- All documentation files used for code documentation must be in markdown format unless stated otherwise
- Use kebab-case for documentation file names (e.g. `git-conventions.md`, `docker-services.md`)
- No icons or emojis in any documentation or code unless explicitly stated otherwise

## File Naming

- Use title case (Title-Case or Title Case) for document titles inside the file
- Use kebab-case for the file name itself
- Exception: `README.md` keeps its standard name

## Code Comments

- Explain WHY, not WHAT (code should be self-documenting)
- Document complex algorithms or business logic
- Add TODO/FIXME comments with context and a brief description of what needs to happen
- Remove commented-out code before committing
- Keep comments up-to-date with code changes
- Use clear, concise language

## README Files

- Every project should have a `README.md` at the root
- Include: project description, setup instructions, usage, and contribution guidelines
- No emojis or icons unless the project explicitly requires them
- Keep it concise -- link to detailed docs rather than duplicating content

## API Documentation

- Document all public endpoints with method, path, request/response format
- Include example requests and responses
- Note required authentication and authorization
- Document error responses and status codes

## Inline Code Documentation

- Use language-appropriate doc formats (JSDoc for JS/TS, Javadoc for Java, docstrings for Python)
- Document public APIs, interfaces, and exported functions
- Skip documentation for obvious getters/setters and self-explanatory methods
- Include parameter descriptions, return types, and thrown exceptions where non-obvious
