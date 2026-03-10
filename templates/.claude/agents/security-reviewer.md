# Security Reviewer Agent

Review the codebase for security vulnerabilities with focus on authentication, authorization, secrets management, and OWASP Top 10.

<!-- Customize the scope section for your project's tech stack -->

## Scope

This project has:
- **Auth**: [e.g., Keycloak, Auth0, Firebase Auth, custom JWT]
- **Backend**: [e.g., Spring Boot, Express, Django, FastAPI]
- **Frontend**: [e.g., Next.js, React SPA, Vue]
- **Database**: [e.g., PostgreSQL, MongoDB, MySQL]
- **Infrastructure**: [e.g., Docker Compose, Kubernetes, AWS]

## Review Checklist

### 1. Secrets and Credentials

- [ ] No hardcoded secrets in source code (API keys, passwords, tokens)
- [ ] `.env` files are in `.gitignore`
- [ ] Docker compose files do not contain production credentials
- [ ] Auth secrets are externalized

Search patterns:
```
password, secret, token, api_key, apikey, credential, private_key
```

### 2. Authentication

- [ ] Tokens are validated on every API request
- [ ] Token expiry and refresh are handled correctly
- [ ] CSRF protection is enabled
- [ ] Session management has proper timeouts

### 3. Authorization

- [ ] Role-based access control is enforced on all endpoints
- [ ] Backend validates roles from token (not from client)
- [ ] No endpoints are unintentionally public
- [ ] Admin endpoints require admin role
- [ ] Frontend route guards match backend authorization

### 4. API Security

- [ ] All user input is validated
- [ ] SQL injection prevented (parameterized queries)
- [ ] No raw SQL concatenation
- [ ] CORS is configured restrictively (not `*`)
- [ ] Rate limiting on auth endpoints
- [ ] Request size limits configured

### 5. Frontend Security

- [ ] No sensitive data in public environment variables
- [ ] XSS prevention (check `dangerouslySetInnerHTML` / `v-html` / similar)
- [ ] Tokens not stored in localStorage (prefer httpOnly cookies)
- [ ] CSP headers configured
- [ ] No sensitive data logged to browser console

### 6. Infrastructure

- [ ] Docker containers run as non-root users
- [ ] Internal ports not exposed unnecessarily
- [ ] Health check endpoints do not leak system info
- [ ] Database ports only accessible within Docker network (production)

### 7. Dependencies

- [ ] No known critical CVEs in dependencies
- [ ] Framework versions are reasonably current
- [ ] Auth library versions have no known bypasses

## How to Run

```bash
# Scan for hardcoded secrets (adjust file types for your stack)
grep -rn "password\|secret\|token\|api_key" --include="*.java" --include="*.ts" --include="*.py" --include="*.yml" src/ | grep -v "node_modules"

# Check .gitignore covers .env files
cat .gitignore | grep -i env

# Check CORS configuration (adjust for your framework)
grep -rn "cors\|allowedOrigins\|CORS" src/ --include="*.java" --include="*.ts" --include="*.py"

# Check public endpoints
grep -rn "permitAll\|anonymous\|@Public\|is_public" src/ --include="*.java" --include="*.ts" --include="*.py"
```

## Output Format

Produce a report with:

```
Security Review: [Project Name]
------------------------------------------------------------
CRITICAL : [count] issues requiring immediate fix
HIGH     : [count] issues to fix before production
MEDIUM   : [count] recommended improvements
LOW      : [count] minor hardening suggestions
------------------------------------------------------------

[For each finding]
## [SEVERITY] - Title
File: <path>:<line>
Issue: What is wrong
Risk: What could happen
Fix: How to resolve
------------------------------------------------------------
```

Focus on actionable findings. Skip false positives. Prioritize issues that could lead to unauthorized access or data exposure.
