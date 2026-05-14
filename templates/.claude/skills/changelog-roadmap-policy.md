---
name: changelog-roadmap-policy
description: Policy for tracking changes based on project maturity (pre-MVP/release vs post-MVP/release)
type: policy
---

# Changelog and Roadmap Tracking Policy

## Rule Summary

**This is a strict rule that applies to all projects.**

Change tracking methodology depends on project maturity status:

### Pre-MVP / Pre-Release (Not Published)
- **Use:** ROADMAP.md only
- **Tracks:** All changes (features, bugfixes, refactoring, infrastructure, etc.)
- **Purpose:** Single source of truth for all development progress
- **Format:** Free-form session notes, task checklists, phase markers

### Post-MVP / Post-Release (Published/Public)
- **Use:** ROADMAP.md + CHANGELOG.md
- **ROADMAP.md:** Feature roadmap ONLY (future planned features and phases)
- **CHANGELOG.md:** ALL changes (features, bugfixes, updates, patches)
- **Purpose:** Separate planning from release notes; public-facing changelog
- **Format:** Standard CHANGELOG format (follows Keep a Changelog specification)

## Standard Changelog Format

When CHANGELOG.md is required (post-MVP/release), follow this structure:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New features

### Changed
- Changes to existing functionality

### Fixed
- Bug fixes

### Removed
- Removed features or deprecated items

---

## [1.0.0] - 2026-04-03

### Added
- Initial public release
- Feature X
- Feature Y

### Fixed
- Bug fix A
- Bug fix B
```

## When to Switch

**Switching trigger:** When the project is marked as MVP-ready or released for public use.

**At switch time:**
1. Mark project status in ROADMAP.md
2. Create CHANGELOG.md with all historical changes formatted to standard
3. Archive old session notes from ROADMAP.md if desired
4. Update ROADMAP.md to show only future planned features
5. All new changes go to CHANGELOG.md under [Unreleased] section

## Examples

### Pre-MVP (Use ROADMAP only)
```markdown
## Current Session State
**Active Phase**: S18 — Gateway Condition Builder
**Stopped At**: Tasks 1–2 complete, tasks 3–8 pending
```

### Post-MVP/Release (Use both)
```markdown
# ROADMAP.md
## Planned Features (S20+)
- Business Service Decoupling
- Advanced reporting

# CHANGELOG.md
## [Unreleased]

### Added
- Custody mappings CRUD
- Form datasource reference panel

### Fixed
- DOA level catalog fallback for empty arrays
```

## Why This Matters

- **Pre-MVP:** Single source of truth prevents confusion during rapid iteration
- **Post-MVP:** Changelog provides public release notes; Roadmap shows future direction
- **Clarity:** Users and contributors know what's planned (ROADMAP) vs what's shipped (CHANGELOG)
- **Professional:** Standard changelog format is expected for published projects

## Action Items

When this rule applies:
1. Before making changes, verify project maturity status
2. Update ROADMAP (pre-MVP) or CHANGELOG (post-MVP)
3. If switching status mid-session, follow "When to Switch" steps above
4. Commit ROADMAP/CHANGELOG updates with the code changes

---

---

## Detecting Project Maturity Status

### Industry Standard: Semantic Versioning (semver.org)

**Primary indicator:** Check the project version number in `package.json`, `pom.xml`, or similar.

| Version Pattern | Status | Tracking Method |
|---|---|---|
| `0.x.x` (e.g., 0.1.0, 0.9.0) | **Pre-MVP/Pre-Release** | Use ROADMAP.md only |
| `1.0.0` or higher (e.g., 1.0.0, 2.1.3) | **Post-MVP/Stable Release** | Use ROADMAP.md + CHANGELOG.md |
| `X.Y.Z-alpha`, `X.Y.Z-beta` | Pre-Release (alpha/beta testing) | Use ROADMAP.md only (pre-stable) |
| `X.Y.Z-rc.1` | Release Candidate | Use ROADMAP.md only (pre-stable) |

**Key Rule:** Major version zero (0.y.z) = development only, API unstable.

### Secondary Verification Methods

1. **CHANGELOG.md existence**
   - Present → Post-MVP/Released
   - Absent → Pre-MVP/Pre-Release

2. **Git tags**
   ```bash
   git tag -l  # Look for release tags like v1.0.0, v2.1.3
   ```
   - Release tags (v1.0.0+) → Post-MVP
   - No tags or beta/rc tags only → Pre-MVP

3. **README.md status indicators**
   - "Beta", "In development", "Pre-release" → Pre-MVP
   - "Stable", "Production-ready", "Active maintenance" → Post-MVP

4. **Repository visibility**
   - Private/internal → Usually Pre-MVP
   - Public with releases → Post-MVP

### At Session Start

When using `project-resume` skill:
```bash
# 1. Check version
cat package.json | grep '"version"'  # or pom.xml, build.gradle, etc.

# 2. Check for CHANGELOG
ls -la CHANGELOG.md

# 3. If still unclear, ask user
```

---

**Applies to:** All projects
**Enforced by:** Claude in all sessions unless explicitly overridden by project CLAUDE.md
**Standard Reference:** [Semantic Versioning 2.0.0](https://semver.org/)
