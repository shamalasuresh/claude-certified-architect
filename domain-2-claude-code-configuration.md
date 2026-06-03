# Domain 2: Claude Code Configuration — 20%

> Covers how Claude Code is configured, scoped, and extended through CLAUDE.md files, permissions, slash commands, and MCP integration.

---

## What This Domain Covers

- `CLAUDE.md` file structure and best practices
- Project-level vs user-level configuration
- Permission models (allow/deny)
- Slash commands
- Integrating MCP servers into Claude Code workflows
- Configuration inheritance and override rules

---

## 1. What is Claude Code?

Claude Code is an agentic coding assistant that operates in your terminal. Unlike chat-based Claude, Claude Code:

- Can read, write, and execute files directly
- Has access to shell commands and tools
- Persists context through a project session
- Is configured through `CLAUDE.md` files at different scopes

---

## 2. CLAUDE.md — The Configuration File

`CLAUDE.md` is a Markdown file that provides Claude Code with:
- Project context (what is this codebase?)
- Behavioral instructions (how should it act?)
- Tool permissions (what is it allowed to do?)
- Custom slash commands (shortcuts for repeated tasks)

### File Locations and Scope

There are **three scope levels**, applied in this order (lower overrides higher):

```
~/.claude/CLAUDE.md          ← User-level (global defaults)
      │
      ▼ inherits + overrides
{project-root}/CLAUDE.md     ← Project-level (team config)
      │
      ▼ inherits + overrides
{subdirectory}/CLAUDE.md     ← Sub-project level (package/service)
```

**Key rule:** More specific scope wins. A project `CLAUDE.md` overrides user-level. A subdirectory `CLAUDE.md` overrides project-level for files within that subdirectory.

### User-Level vs Project-Level — When to Use Each

| Configuration | User-Level (`~/.claude/`) | Project-Level (`{repo}/`) |
|---------------|--------------------------|--------------------------|
| Personal preferences | ✓ | ✗ |
| Editor shortcuts | ✓ | ✗ |
| Global safe defaults | ✓ | ✗ |
| Project tech stack info | ✗ | ✓ |
| Team code conventions | ✗ | ✓ |
| Project-specific permissions | ✗ | ✓ |
| Sensitive credentials/config | ✗ | ✗ (never in CLAUDE.md) |

**Critical exam point:** Sensitive data (API keys, passwords, secrets) should NEVER be placed in `CLAUDE.md`. It is committed to version control and read by the model.

---

## 3. CLAUDE.md Structure — Best Practices

A well-structured `CLAUDE.md` has these sections:

```markdown
# Project Name

## Overview
Brief description of what this codebase does and its tech stack.

## Architecture
Key architectural decisions and patterns in use.

## Development Guidelines
- Code style rules
- Testing requirements
- Commit message format

## Common Tasks
How to run tests, build, deploy.

## Allowed Operations
What Claude Code is permitted to do in this project.

## Forbidden Operations
Explicit list of things Claude Code must not do.
```

### What to Include

**Good content for CLAUDE.md:**
- Tech stack (language, framework, key libraries)
- Coding conventions (naming, file structure)
- Test patterns (how to write and run tests)
- Common build/run commands
- Architecture decisions that affect how code is written
- Which files/directories are sensitive (don't modify)

**Bad content for CLAUDE.md:**
- Secrets, tokens, credentials
- Information that changes frequently (current sprint tasks)
- Opinions without actionable guidance
- Contradictory instructions

---

## 4. Permission Model

Claude Code uses an allow/deny permission system to control what operations it can perform.

### Permission Tiers

| Tier | Examples | Risk Level |
|------|----------|------------|
| Read | Read files, view directory structure | Low |
| Write | Create files, edit files | Medium |
| Execute | Run shell commands, run tests | High |
| Network | Make HTTP requests, access external services | High |
| Destructive | Delete files, drop databases | Very High |

### Permission Configuration Syntax

```markdown
## Permissions

### Allowed
- Read all source files
- Write to src/ and tests/ directories
- Execute: npm test, npm run build, git status, git diff
- Execute: docker compose up (read-only operations)

### Forbidden
- Delete any files
- Modify .env files
- Push to git remote
- Access production environment
- Execute: rm -rf, DROP TABLE, or similar destructive commands
```

### Permission Inheritance Rules

1. User-level permissions are the **baseline**
2. Project-level can **restrict** user-level (more restrictive wins for safety-sensitive operations)
3. Project-level can **expand** user-level for specific operations defined in that project
4. Subdirectory-level can further restrict for sensitive sub-packages

**Exam insight:** When a question asks about restricting access to sensitive directories, the answer involves project-level `CLAUDE.md` with explicit forbidden rules, not just relying on user-level config.

---

## 5. Slash Commands

Slash commands are shortcuts defined in `CLAUDE.md` that expand into longer instructions.

### Defining Slash Commands

```markdown
## Slash Commands

### /review
Perform a code review of the staged changes. Check for:
- Security vulnerabilities
- Missing tests
- Performance issues
- Code style violations per our ESLint config

### /test-coverage
Run the full test suite and generate a coverage report. 
Output the coverage percentage and list files below 80%.

### /changelog
Generate a changelog entry from the git log since the last tag.
Format: ## [version] - YYYY-MM-DD followed by bullet points.
```

### Slash Command Best Practices

| Practice | Reason |
|----------|--------|
| Give slash commands clear, action-oriented names | Reduces ambiguity |
| Include output format in the command definition | Gets consistent results |
| Define the scope (which files, which tests) | Prevents over-broad actions |
| Keep commands focused on one task | Easier to debug when they fail |

### User-Defined vs Project-Defined Slash Commands

- **User-level** (`~/.claude/CLAUDE.md`): Available in every project (e.g., personal code review style)
- **Project-level** (`{repo}/CLAUDE.md`): Available only in this project (e.g., project-specific deploy commands)

---

## 6. Integrating MCP Servers into Claude Code

MCP (Model Context Protocol) servers extend Claude Code's capabilities by providing additional tools, resources, and prompts.

### MCP Configuration for Claude Code

MCP servers are configured in a separate config file (not in `CLAUDE.md`):

```json
// .claude/mcp_config.json or ~/.claude/mcp_config.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

**Key point:** Environment variables in MCP config should reference shell env vars (`${VAR_NAME}`), not hardcoded values.

### Project-Level vs User-Level MCP

| Configuration | Where |
|---------------|-------|
| Personal MCP servers (always available) | `~/.claude/mcp_config.json` |
| Project-specific MCP servers | `{project}/.claude/mcp_config.json` |
| Team-shared MCP server config | Committed to repo, uses env var refs |

### Referencing MCP in CLAUDE.md

Once an MCP server is configured, you can reference it in `CLAUDE.md`:

```markdown
## Available Tools

This project uses the following MCP tools:
- `github`: Access to issues, PRs, and repository data
- `postgres`: Direct database query access (read-only in dev)

When working on issues, always check the linked GitHub issue context 
before implementing changes.
```

---

## 7. Configuration Troubleshooting Patterns

### Problem: Claude Code ignores project instructions
**Root cause:** Instructions are in user-level CLAUDE.md but project-level has conflicting content.
**Fix:** Ensure project-level CLAUDE.md is explicit. More specific scope wins.

### Problem: Claude Code modifies files it shouldn't
**Root cause:** No explicit forbidden list, or forbidden list is too vague.
**Fix:** Use explicit path patterns in the Forbidden section: `Forbidden: Modify any file in /config/production/`

### Problem: Team members have different Claude Code behavior
**Root cause:** Relying on user-level config for team conventions.
**Fix:** Move team conventions to project-level CLAUDE.md, committed to version control.

### Problem: MCP tool available but Claude Code doesn't use it
**Root cause:** CLAUDE.md doesn't mention the tool, or the tool isn't relevant to the current task.
**Fix:** Explicitly document in CLAUDE.md when and how to use available MCP tools.

---

## 8. Security Best Practices in CLAUDE.md

1. **Never store credentials** — Use env var references, never actual values
2. **Explicit deny > implicit allow** — List what's forbidden explicitly
3. **Restrict destructive operations** — Always in the Forbidden section
4. **Commit CLAUDE.md to version control** — It's team documentation, not a secret
5. **Review CLAUDE.md like code** — It shapes agent behavior; treat it seriously
6. **Separate prod and dev permissions** — CLAUDE.md should explicitly call out production boundaries

---

## 9. Exam Scenarios & Right Answers

### Scenario: "Developer wants Claude Code to always follow their personal code style"
**Right answer:** User-level `~/.claude/CLAUDE.md` with style preferences. Project-level should not override this.

### Scenario: "Team wants consistent Claude Code behavior across all developers"
**Right answer:** Project-level `CLAUDE.md` committed to the repository. This ensures all team members get the same configuration.

### Scenario: "Claude Code is accessing production database in a dev session"
**Right answer:** Project-level `CLAUDE.md` should explicitly forbid production environment access. MCP config should point to dev DB only.

### Scenario: "Frequently repeated task needs to be a quick shortcut"
**Right answer:** Define a slash command in `CLAUDE.md` with explicit output format and scope.

### Scenario: "API key needs to be available to Claude Code's MCP server"
**Right answer:** Store in environment variable, reference as `${ENV_VAR}` in mcp_config.json. Never put in CLAUDE.md.

### Scenario: "Sub-package in monorepo has stricter security requirements"
**Right answer:** Add a subdirectory-level `CLAUDE.md` in that package with additional restrictions. Subdirectory scope overrides project scope.

---

## 10. Quick Reference Card

### Scope Hierarchy (specific wins)
```
User-level      (~/.claude/CLAUDE.md)
   ↓ overridden by
Project-level   ({repo}/CLAUDE.md)
   ↓ overridden by
Sub-directory   ({repo}/packages/x/CLAUDE.md)
```

### CLAUDE.md Sections
```
1. Overview (what is this?)
2. Architecture (key patterns)
3. Development Guidelines (conventions)
4. Common Tasks (commands)
5. Allowed Operations
6. Forbidden Operations
7. Slash Commands
8. Available MCP Tools
```

### MCP Config Location
```
~/.claude/mcp_config.json     ← personal tools
{project}/.claude/mcp_config.json ← project tools (use env vars)
```
