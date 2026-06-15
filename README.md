# agentkit

> AI Engineering Platform CLI — install governed agents, skills, hooks, and coding standards into any project in seconds.

Works with **GitHub Copilot** (VS Code) and **Claude** (Claude Code / Claude Desktop).

**Latest: [v0.2.1](https://github.com/JkhatriInfobox/agentkit/releases/tag/v0.2.1)** — SSL self-update fix, version reporting fix, VS Code picker cleanup (D-15 post-release patch)

---

## Install

### macOS
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agentkit/releases/latest/download/agentkit-macos-arm64 \
  && chmod +x agentkit && sudo mv agentkit /usr/local/bin/
```

### Linux
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agentkit/releases/latest/download/agentkit-linux-amd64 \
  && chmod +x agentkit && sudo mv agentkit /usr/local/bin/
```

### Windows
Download [`agentkit-windows-amd64.exe`](https://github.com/JkhatriInfobox/agentkit/releases/latest) from the latest release and add it to your PATH.

> No Python or pip required. The binary is self-contained.

Verify:
```bash
agentkit --version
```

---

## Quick Start

```bash
cd ~/my-project

# Install core agents + governance
agentkit init

# Or install a domain pack — core is included automatically
agentkit init --pack go-development
agentkit init --pack terraform-provider
agentkit init --pack jira-integration

# Verify everything installed cleanly
agentkit doctor
```

Open VS Code or Claude Code — agents are ready in `.github/agents/`.

---

## Upgrading

### Standard upgrade (all users)

Two-step upgrade — the binary first, then agent files in each project:

```bash
# Step 1: upgrade the agentkit binary
agentkit self-update

# Step 2: refresh agent files in each project
cd ~/my-project
agentkit sync
```

### Upgrading to v0.2.0 — D-15 Single-Agent Architecture

v0.2.0 includes a **breaking change**: the four separate governance agents (`developer`, `reviewer`, `architect`, `platform-engineer`) are consolidated into a single `agentforge` agent with `mode:` routing.

**What changed in workflow YAML files:**

| Before (pre-v0.2.0) | After (v0.2.0+) |
|---------------------|-----------------|
| `agent: reviewer` | `agent: agentforge` + `mode: reviewer` |
| `agent: developer` | `agent: agentforge` + `mode: developer` |
| `agent: architect` | `agent: agentforge` + `mode: architect` |
| `agent: platform-engineer` | `agent: agentforge` + `mode: platform-engineer` |
| `agent_profiles:` in pack.yaml | `mode_profiles:` |

**Migrate automatically after `agentkit self-update && agentkit sync`:**

```bash
# 1. Check if your repo has legacy syntax
forge doctor --check-legacy

# 2. Preview what will change (read-only, no writes)
forge migrate d15 analyze

# 3. Apply the migration (backup created automatically at .forge/migrate-d15-backup-*.tar.gz)
forge migrate d15 apply

# 4. Undo if needed
forge migrate d15 rollback

# 5. Recompile artifacts
forge compile --source . --target .
```

**Before / after example:**

```yaml
# Before (pre-v0.2.0)
steps:
  - name: review
    agent: reviewer
  - name: implement
    agent: developer

# After (v0.2.0+)
steps:
  - name: review
    agent: agentforge
    mode: reviewer
  - name: implement
    agent: agentforge
    mode: developer
```

> **Specialist agents are unchanged** — `jira-agent`, `oss-agent`, `docs-writer`, `release-manager` are not affected and continue to work as-is.

Full upgrade guide: [docs/upgrade-guide.md](https://github.com/JkhatriInfobox/agents/blob/main/docs/upgrade-guide.md)

---

## Migrating from `make seed`

If you set up agentkit using the older `make seed` method, migrate to the current binary in one command:

```bash
# Preview what will change (safe — no writes)
agentkit migrate --dry-run

# Migrate and install the right pack for your project
agentkit migrate --pack go-development
agentkit migrate --pack terraform-provider
agentkit migrate --pack ansible-collection
agentkit migrate --pack python-automation
agentkit migrate --pack jira-integration

# Just core agents (no domain pack)
agentkit migrate

# Non-interactive (CI / scripted use)
agentkit migrate --pack go-development --yes
```

### What migrate does

1. **Shows** what old tooling will be removed and what memory is protected
2. **Asks for confirmation** (skip with `--yes`)
3. **Removes** old compiled files: `.github/agents/`, `.github/skills/`, `.github/hooks/`, `.github/prompts/`, `AGENTS.md`, `CLAUDE.md`
4. **Preserves** all agent memory and project intelligence — `.ai/` is never touched
5. **Installs** fresh tooling via `agentkit init`

### Memory is always preserved

```
.ai/
  memory/      ← ✅ KEPT — agent learnings (developer.md, architect.md …)
  repository/  ← ✅ KEPT — project intelligence (file map, tech stack, deps)
  standards/   ← ✅ KEPT — coding conventions the agent learned
  patterns/    ← ✅ KEPT — codebase patterns
  decisions/   ← ✅ KEPT — ADRs and architectural decisions
```

---

## Domain Packs

Domain packs install everything you need for a specific technology: core agents + domain-specific agents, skills, and hooks — in a single command.

```bash
agentkit init --pack <name>
```

| Pack | Use for | Skills included | Domain hooks |
|------|---------|-----------------|--------------|
| `go-development` | Go services, CLIs, scripts | foundation, service-dev, CLI, scripting, testing, code-review | go-fmt, go-vet |
| `terraform-provider` | Terraform provider development | provider-dev, resource, data-source, docs, acceptance-testing, advanced-types, provider-client | terraform-fmt |
| `ansible-collection` | Ansible modules, plugins, collections | foundation, modules, module-utils, lookup, filter, callback, inventory, integration-testing, docs, maintenance, regression-audit | ansible-lint, yaml-lint |
| `python-automation` | Python scripts, CLIs, automation tools | scripting, testing | coverage-check, lint-gate, diff-size-guard |
| `jira-integration` | JIRA ticket management, creation, updates, time logging | foundation, ticket-read, ticket-create, ticket-update, comment-management, time-tracking, custom-fields | secret-guard |
| `oss-maintenance` | Open-source project maintenance | changelog-management, issue-triage, issue-classification, bug-reproduction, release-note-generation, community-response, contributor-onboarding, roadmap-management | — |

Each pack automatically includes `core` (`agentforge`, specialist agents, hooks, prompts). You do not need to install core separately.

---

## JIRA Integration

The `jira-integration` pack adds a governed JIRA agent to your project with a strict **read-free / write-gated** permission model.

### Setup

```bash
# 1. Install the pack
agentkit init --pack jira-integration

# 2. Edit .ai/jira-config.yaml (created automatically)
#    Set base_url and email

# 3. Set your API token (never put this in a file)
export JIRA_API_TOKEN=your_token_here
# Get your token at: https://id.atlassian.com/manage-profile/security/api-tokens

# 4. Open VS Code → select [jira-agent] from the Agent dropdown
```

### What the JIRA agent can do

| Operation | Permission required? |
|-----------|---------------------|
| Search issues with JQL | ❌ No — proceed freely |
| Read issue details, comments, worklogs | ❌ No — proceed freely |
| Discover custom fields | ❌ No — proceed freely |
| Create issue | ✅ Yes — shows payload, asks approval |
| Update fields, labels, priority | ✅ Yes — shows diff, asks approval |
| Transition status | ✅ Yes — shows current → new, asks approval |
| Add/reply to comment | ✅ Yes — shows preview, asks approval |
| Log work time | ✅ Yes — shows time + date, asks approval |

### Example interactions

```
# Read a ticket
[jira-agent] Show me PROJ-123 with full context for development

# Search
[jira-agent] Find all open bugs in the auth service assigned to me

# Create a developer-ready ticket
[jira-agent] Create a story for adding JWT refresh token rotation to the auth service

# Start work on a ticket
[jira-agent] Load PROJ-456 context and hand off to developer

# Update and log
[jira-agent] Move PROJ-123 to In Review and post a comment that the PR is ready
[jira-agent] Log 2h 30m on PROJ-123 for implementing the refresh token logic
```

### Developer handoff workflow

```
1. [jira-agent]              Load PROJ-123 — formats full context (description, ACs, comments, epic)
2. [jira-agent]              Hand off to developer
3. [agentforge mode:developer]  Implement the work
4. [jira-agent]              Move to In Review + post comment   (asks approval)
5. [jira-agent]              Log 3h work time                  (asks approval)
6. [agentforge mode:reviewer]   Review the diff
7. [jira-agent]              Move to Done                       (asks approval)
```

---

## What Gets Installed

`agentkit init` (core) installs files into your project:

```
.github/
  agents/          ← AI agents (use in VS Code Copilot / Claude Code)
    agentforge.agent.md   ← single governance agent (developer/reviewer/architect/platform modes)
    docs-writer.agent.md
    release-manager.agent.md
    jira-agent.agent.md
    oss-agent.agent.md
  instructions/    ← Always-on coding standards (auto-applied by the AI)
    python-style.instructions.md
    verify-first.instructions.md
  hooks/           ← Governance hooks (auto-run on file edits)
    ruff-format.json
    secret-guard.json
    block-dangerous.json
    intel-refresh.json
    wiki-audit-flag.json
  prompts/         ← Slash-command prompts (/dev-plan, /plan-review, etc.)
    dev-plan.prompt.md
    plan-review.prompt.md
    wiki-init.prompt.md
    wiki-audit.prompt.md
    root-cause.prompt.md
AGENTS.md          ← Always-on Python dev conventions
.vscode/
  mcp.json         ← MCP server configuration
.ai/
  repository/      ← Auto-generated project intelligence
```

> **Note:** `developer.agent.md`, `reviewer.agent.md`, `architect.agent.md` are no longer installed as separate files (removed in v0.2.1). All governance is handled by `agentforge.agent.md` with mode routing.

---

## Agents

Select an agent from the `[Agent ▼]` dropdown in VS Code Copilot chat, or use `@agent-name` in Claude Code.

### Primary governance agent (v0.2.0+)

The `agentforge` agent routes internally to the right mode based on your request. You can also set the mode explicitly in workflow YAML or via the `AGENTFORGE_MODE` env var.

| Agent | Mode | Role | Capabilities |
|-------|------|------|-------------|
| `agentforge` | `developer` | Implement features and fix bugs | Read + Write + Execute |
| `agentforge` | `reviewer` | Review code — finds real bugs, never style issues. **Read-only: never writes or commits.** | Read + Search + Execute (CI only) |
| `agentforge` | `architect` | Design systems, write ADRs, evaluate trade-offs | Read only (proposes, never implements) |
| `agentforge` | `platform-engineer` | Build and maintain the AgentForge platform | Read + Write + Execute |

```yaml
# Explicit mode in a workflow step
agent: agentforge
mode: reviewer

# Or via environment variable
export AGENTFORGE_MODE=developer
```

### Specialist agents

| Agent | Role | Capabilities |
|-------|------|-------------|
| `jira-agent` | Full JIRA operations — writes require approval | Read + Execute + HTTP (write-gated) |
| `jira-reader` | Read JIRA issues and search — no write, no approvals needed | Read + Execute (read-only) |
| `oss-maintainer` | Issue triage, community responses, release notes | Read + Write |
| `oss-triager` | Classify and route incoming issues | Read only |
| `release-manager` | Changelog, version bump, tag, publish | Read + Write + Execute |
| `docs-writer` | Write README, API docs, changelogs | Read + Write |

### How to use in VS Code Copilot
1. Open Copilot Chat (`Ctrl+Shift+I` / `Cmd+Shift+I`)
2. Click `[Agent ▼]` dropdown → select `agentforge` (or a specialist agent)
3. Type your request — governance mode is routed automatically

### How to use in Claude Code
```
[agentforge] Implement the login endpoint with JWT auth
[agentforge] Review my changes in src/api.py
[jira-agent] Show me PROJ-123 and format it for development
[docs-writer] Update the README with the new API endpoints
```

---

## Bundles

| Bundle | Contents | Use when |
|--------|----------|----------|
| `core` | agentforge, specialist agents, hooks, prompts | Every project (default) |
| `testing` | test-runner agent, pytest skill, testing instructions | Python projects with tests |
| `packaging` | release-manager agent, Python packaging skill | Libraries / packages |
| `scripting` | script-writer agent, debugging skill | Automation / scripts |
| `mcp` | Full MCP config (filesystem + git + github + fetch) | When you need GitHub MCP |
| `vscode` | VS Code extension recommendations | VS Code users |
| `oss-maintenance` | oss-maintainer, oss-triager, release-manager + 8 oss skills | Open source projects |

```bash
agentkit init --pack go-development        # recommended — includes core
agentkit init --bundle core,testing        # fine-grained bundle install
agentkit list                              # see all packs and bundles
```

---

## Commands

### `agentkit init`
```bash
agentkit init                              # install core bundle (default)
agentkit init --pack go-development        # install Go domain pack (includes core)
agentkit init --pack jira-integration      # install JIRA integration pack
agentkit init --bundle core,testing        # install specific bundles
agentkit init --force                      # overwrite locally modified files
agentkit init --user                       # install to user profile (~/.agentkit/)
agentkit init --project /path/to/repo      # install into a specific project path
```

### `agentkit self-update`
```bash
agentkit self-update                       # download latest and replace binary
agentkit self-update --version v0.2.1      # pin to a specific version
agentkit self-update --yes                 # skip confirmation
```

### `agentkit sync`
```bash
agentkit sync           # update installed files (safe — skips local edits)
agentkit sync --force   # overwrite even locally modified files
agentkit sync --user    # sync user-profile install
```

### `agentkit doctor`
```bash
agentkit doctor         # check workspace install
agentkit doctor --user  # check user-profile install
```

### `agentkit migrate`
```bash
agentkit migrate --pack go-development     # migrate from make seed + install Go pack
agentkit migrate --dry-run                 # preview only (no writes)
agentkit migrate --yes                     # skip confirmation
```

### `forge migrate d15` — workflow syntax migration (v0.2.0+)
```bash
forge doctor --check-legacy        # detect legacy governance agent syntax
forge migrate d15 analyze          # count and list affected files (read-only)
forge migrate d15 apply            # rewrite files in place (backup created)
forge migrate d15 rollback         # restore from backup
```

### `agentkit intel`
```bash
agentkit intel build    # full rebuild of all inventories
agentkit intel refresh  # fast refresh (file/tech/deps only)
agentkit intel verify   # check inventories are fresh (exits 1 if stale)
```

---

## Prompts (Slash Commands)

| Prompt | Command | Use |
|--------|---------|-----|
| `dev-plan` | `/dev-plan` | Generate a structured TODO plan before writing code |
| `plan-review` | `/plan-review` | Architect reviews your plan — loops until approved |
| `wiki-init` | `/wiki-init` | One-time: enrich `.ai/` wiki with real project knowledge |
| `wiki-audit` | `/wiki-audit` | Architect audits wiki after each change |
| `root-cause` | `/root-cause` | Investigate failures by actual execution before fixing |

---

## Hooks

| Hook | Trigger | Action |
|------|---------|--------|
| `ruff-format` | After every `.py` edit | Auto-format with ruff |
| `secret-guard` | Before any file write | Block secrets from being committed |
| `block-dangerous` | Before shell commands | Block `rm -rf`, `format`, etc. |
| `intel-refresh` | After source file edits | Refresh `.ai/repository/` intelligence |
| `wiki-audit-flag` | After source edits | Flag that architect wiki review is needed |
| `mode-guard` | PreToolUse (governance) | Enforce read-only for reviewer/architect modes |

Domain packs add additional hooks (e.g. `go-fmt`, `terraform-fmt`, `ansible-lint`).

---

## User Profile Install

Install agents globally so they're available in every project without running `agentkit init` each time:

```bash
agentkit init --user    # installs to ~/.agentkit/
agentkit doctor --user  # verify
agentkit sync --user    # update
```

---

## Global `.gitignore`

agentkit installs local developer tooling that should **not** be committed to your repositories. Use a global gitignore:

```bash
touch ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

Add to `~/.gitignore_global`:

```gitignore
# AgentForge / agentkit — local developer tooling (never commit)
.agentkit/
.ai/
.forge/
.claude/
.github/agents/
.github/skills/
.github/hooks/
.github/prompts/
.github/instructions/
.github/copilot-instructions.md
.vscode/mcp.json
AGENTS.md
```

---

## Typical Workflow

```bash
# 1. One-time: set up global gitignore
git config --global core.excludesfile ~/.gitignore_global

# 2. New project
cd ~/my-go-project
agentkit init --pack go-development

# 3. Migrate existing project from make seed
cd ~/my-project
agentkit migrate --pack go-development

# 4. Upgrade binary + refresh project files
agentkit self-update
agentkit sync

# 5. After upgrading to v0.2.0 — migrate workflow syntax (if needed)
forge doctor --check-legacy
forge migrate d15 apply
forge compile --source . --target .

# 6. Daily: use agentforge for governance work
# [agentforge] Review my changes in src/api.py
# [agentforge] Implement the login endpoint with JWT auth
# [jira-agent] Load PROJ-123 and hand off to developer
```

---

## FAQ

**Do I need Python or pip?**
No. The binary is fully self-contained.

**What changed in v0.2.1?**
Patch release with four fixes: (1) `agentkit self-update` SSL certificate failure on macOS, (2) `agentkit --version` always reporting the wrong version (`0.1.14`), (3) legacy governance agents (`developer`, `reviewer`, `architect`) appearing in the VS Code agent picker after the D-15 migration, (4) domain pack skills (e.g. Ansible skills) not being injected into `agentforge` correctly. No breaking changes — upgrade with `agentkit self-update`.

**What changed in v0.2.0?**
The four governance agents (`developer`, `reviewer`, `architect`, `platform-engineer`) are merged into a single `agentforge` agent with mode routing. Specialist agents (`jira-agent`, `docs-writer`, `oss-agent`, `release-manager`) are unchanged. If you have workflow YAML files using the old syntax, run `forge migrate d15 apply` to update them automatically.

**Do I need to change anything when upgrading to v0.2.0?**
Only if you have workflow YAML files or `pack.yaml` files using governance agent names directly. Run `forge doctor --check-legacy` after upgrading — if it says clean, no action is needed.

**`agentkit self-update` fails with SSL error on macOS — what do I do?**
Upgrade to v0.2.1 which fixes this. If you're on an older binary and can't self-update, re-run the install curl from the releases page, or install `certifi` to fix the current binary: `pip install certifi`.

**What's the difference between `--pack` and `--bundle`?**
`--pack` is the recommended way — installs core + a complete domain-specific setup in one command. `--bundle` is for fine-grained control over exactly which components are installed.

**Does this modify my `.gitignore`?**
No. agentkit never touches your `.gitignore`. Use the global gitignore approach above.

**Will `agentkit migrate` delete my agent memory?**
No. `migrate` only removes compiled tooling files. Everything in `.ai/` is preserved. Run `--dry-run` first to see exactly what changes.

**Is my JIRA API token safe?**
Yes — read from `JIRA_API_TOKEN` env var at runtime. Never stored in files.

**Can the reviewer agent fix code?**
No. The reviewer mode in `agentforge` enforces a hard-stop refusal when asked to write or commit. Switch to developer mode to apply findings.

**How do I set the agentforge governance mode explicitly?**
In a workflow step: `mode: reviewer`. Via env var: `export AGENTFORGE_MODE=reviewer`. Or just describe your task — the agent routes automatically.

**How do I upgrade agentkit?**
`agentkit self-update` for the binary, then `agentkit sync` in each project.

**Can I roll back the D-15 migration?**
Yes — `forge migrate d15 apply` creates a backup at `.forge/migrate-d15-backup-*.tar.gz`. Run `forge migrate d15 rollback` to restore original files.

**I still see `developer`, `reviewer`, `architect` in my VS Code agent picker after upgrading.**
Run `agentkit sync` in your project to pull the updated agent files. If you manage agent files manually, remove `.github/agents/developer.agent.md`, `.github/agents/reviewer.agent.md`, `.github/agents/architect.agent.md` — they are no longer part of the D-15 architecture.

---

## Releases

Binaries published for every release:

- `agentkit-macos-arm64` — macOS Apple Silicon
- `agentkit-linux-amd64` — Linux x86_64
- `agentkit-windows-amd64.exe` — Windows x86_64

See [Releases](https://github.com/JkhatriInfobox/agentkit/releases) for all versions.

**Latest: [v0.2.1](https://github.com/JkhatriInfobox/agentkit/releases/tag/v0.2.1)** — SSL self-update fix, version reporting fix, VS Code picker cleanup, compiler domain-skill injection fix
