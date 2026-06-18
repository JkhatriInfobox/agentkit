# agentkit

> AI Engineering Platform CLI — install governed agents, skills, hooks, and coding standards into any project in seconds.

Works with **GitHub Copilot** (VS Code) and **Claude** (Claude Code / Claude Desktop).

**Latest: [v0.3.1](https://github.com/JkhatriInfobox/agentkit/releases/tag/v0.3.1)** — Security hardening: SHA256 binary integrity verification, TLS hardening, safe tar extraction, marketplace SSL hardening, 20 new security tests.

---

## Install

### macOS (Apple Silicon)
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agentkit/releases/latest/download/agentkit-macos-arm64 \
  && chmod +x agentkit && sudo mv agentkit /usr/local/bin/
```

### macOS (Intel)
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agentkit/releases/latest/download/agentkit-macos-amd64 \
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

Two-step upgrade — binary first, then agent files in each project:

```bash
# Step 1: upgrade the agentkit binary (SHA256 verified from v0.3.1)
agentkit self-update

# Step 2: refresh agent files in each project
cd ~/my-project
agentkit sync
```

### Upgrading from pre-v0.2.0 — D-15 Single-Agent Architecture

v0.2.0 introduced a **breaking change**: the four separate governance agents (`developer`, `reviewer`, `architect`, `platform-engineer`) are consolidated into a single `agentforge` agent with `mode:` routing.

**Migrate automatically after upgrading:**

```bash
# 1. Check if your repo has legacy syntax
forge doctor --check-legacy

# 2. Preview what will change (read-only, no writes)
forge migrate d15 analyze

# 3. Apply the migration (backup created at .forge/migrate-d15-backup-*.tar.gz)
forge migrate d15 apply

# 4. Undo if needed
forge migrate d15 rollback
```

**Before / after:**

```yaml
# Before (pre-v0.2.0)
steps:
  - name: review
    agent: reviewer

# After (v0.2.0+)
steps:
  - name: review
    agent: agentforge
    mode: reviewer
```

| Before (pre-v0.2.0) | After (v0.2.0+) |
|---------------------|-----------------|
| `agent: reviewer` | `agent: agentforge` + `mode: reviewer` |
| `agent: developer` | `agent: agentforge` + `mode: developer` |
| `agent: architect` | `agent: agentforge` + `mode: architect` |
| `agent: platform-engineer` | `agent: agentforge` + `mode: platform-engineer` |

> **Specialist agents are unchanged** — `jira-agent`, `oss-agent`, `docs-writer`, `release-manager` are not affected.

Full upgrade guide: [docs/upgrade-guide.md](https://github.com/JkhatriInfobox/agents/blob/main/docs/upgrade-guide.md)

---

## Migrating from `make seed`

```bash
# Preview what will change (safe — no writes)
agentkit migrate --dry-run

# Migrate and install the right pack for your project
agentkit migrate --pack go-development
agentkit migrate --pack python-automation
agentkit migrate --pack ansible-collection
agentkit migrate --pack jira-integration

# Non-interactive (CI / scripted)
agentkit migrate --pack go-development --yes
```

**Memory is always preserved:**

```
.ai/
  memory/      <- KEPT: agent learnings
  repository/  <- KEPT: project intelligence
  standards/   <- KEPT: coding conventions
  patterns/    <- KEPT: codebase patterns
  decisions/   <- KEPT: ADRs
```

---

## What Gets Installed

```
.github/
  agents/
    agentforge.agent.md   <- single governance agent (developer/reviewer/architect modes)
    docs-writer.agent.md
  skills/
    <pack-skills>/SKILL.md
  hooks/
    ruff-format.json
    secret-guard.json
    block-dangerous.json
    intel-refresh.json
    wiki-audit-flag.json
  instructions/
    python-style.instructions.md
  prompts/
    dev-plan.prompt.md
    plan-review.prompt.md
    wiki-init.prompt.md
    wiki-audit.prompt.md
    root-cause.prompt.md
AGENTS.md
.claude/
  agents/
    agentforge.md         <- governance agent for Claude Code
.vscode/
  mcp.json               <- MCP server configuration
.ai/
  repository/            <- auto-generated project intelligence
```

> `developer.agent.md`, `reviewer.agent.md`, `architect.agent.md` are no longer installed as separate files (removed in v0.2.1). All governance is handled by `agentforge.agent.md` with mode routing.

---

## Agents

Select an agent from the `[Agent]` dropdown in VS Code Copilot chat, or use `@agent-name` in Claude Code.

### Primary governance agent

The `agentforge` agent routes internally to the right mode based on your request.

| Agent | Mode | Role | Capabilities |
|-------|------|------|-------------|
| `agentforge` | `developer` | Implement features, fix bugs | Read + Write + Execute |
| `agentforge` | `reviewer` | Code review — finds bugs, **never writes or commits** | Read + Search |
| `agentforge` | `architect` | Design systems, write ADRs — proposes only | Read only |
| `agentforge` | `platform-engineer` | Platform/infrastructure work | Read + Write + Execute |

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
| `oss-agent` | Issue triage, community responses, release notes | Read + Write |
| `release-manager` | Changelog, version bump, tag, publish | Read + Write + Execute |
| `docs-writer` | Write README, API docs, changelogs | Read + Write |

### How to use in VS Code Copilot
1. Open Copilot Chat (`Ctrl+Shift+I` / `Cmd+Shift+I`)
2. Click `[Agent]` dropdown and select `agentforge` (or a specialist agent)
3. Describe your task — governance mode is routed automatically

### How to use in Claude Code
```
[agentforge] Implement the login endpoint with JWT auth
[agentforge] Review my changes in src/api.py
[jira-agent] Show me PROJ-123 and format it for development
[docs-writer] Update the README with the new API endpoints
```

---

## Domain Packs

```bash
agentkit init --pack <name>
```

| Pack | Use for | Skills included |
|------|---------|-----------------|
| `go-development` | Go services, CLIs, scripts | foundation, service-dev, CLI, scripting, testing, code-review |
| `terraform-provider` | Terraform provider development | provider-dev, resource, data-source, docs, acceptance-testing |
| `ansible-collection` | Ansible modules, plugins, collections | foundation, modules, module-utils, lookup, filter, integration-testing, docs |
| `python-automation` | Python scripts, CLIs, automation | scripting, testing |
| `jira-integration` | JIRA ticket management | foundation, ticket-read, ticket-create, ticket-update, time-tracking |
| `oss-maintenance` | Open-source project maintenance | changelog-management, issue-triage, bug-reproduction, release-note-generation |

Each pack automatically includes `core`. You do not need to install core separately.

---

## JIRA Integration

The `jira-integration` pack adds a governed JIRA agent with a strict **read-free / write-gated** permission model.

```bash
agentkit init --pack jira-integration
```

**Token-saving CLI tool** installed to `.agentkit/tools/jira_cli.py`:

```bash
# READ (no approval needed)
python .agentkit/tools/jira_cli.py get PROJ-123
python .agentkit/tools/jira_cli.py search "project=PROJ AND status='In Progress'"

# WRITE (ask_user approval required first)
python .agentkit/tools/jira_cli.py create --title "feat: ..." --type Story --project PROJ
python .agentkit/tools/jira_cli.py transition PROJ-123 "In Review"
python .agentkit/tools/jira_cli.py comment PROJ-123 "PR is ready"
```

Config: `.ai/jira-config.yaml` + `JIRA_API_TOKEN` env var.

---

## Bundles

| Bundle | Contents | Use when |
|--------|----------|----------|
| `core` | agentforge, specialist agents, hooks, prompts | Every project (default) |
| `testing` | pytest skill, testing instructions | Python projects with tests |
| `packaging` | Python packaging skill | Libraries / packages |
| `scripting` | debugging skill | Automation / scripts |
| `mcp` | Full MCP config (filesystem + git + github + fetch) | When you need GitHub MCP |
| `vscode` | VS Code extension recommendations | VS Code users |

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
agentkit init --user                       # install to user profile (~/.agentkit/)
agentkit init --project /path/to/repo      # install into a specific project path
agentkit init --target /path/to/repo       # alias for --project
agentkit init --force                      # overwrite locally modified files
```

### `agentkit self-update`
```bash
agentkit self-update                       # download latest, verify SHA256, replace binary
agentkit self-update --version v0.3.1      # pin to a specific version
agentkit self-update --yes                 # skip confirmation
```

> **Integrity (v0.3.1+):** Every release includes a `.sha256` checksum file. `agentkit self-update` downloads and verifies the SHA256 before installing. Mismatch aborts with exit 1. TLS certificate verification is enforced with no bypass.

### `agentkit update`
```bash
agentkit update                            # upgrade via pip (use if installed with pip/pipx)
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

### `forge migrate d15` — workflow syntax migration
```bash
forge doctor --check-legacy        # detect legacy governance agent syntax
forge migrate d15 analyze          # list affected files (read-only)
forge migrate d15 apply            # rewrite files in place (backup created)
forge migrate d15 rollback         # restore from backup
```

### `agentkit intel`
```bash
agentkit intel build    # full rebuild of all .ai/repository/ inventories
agentkit intel refresh  # fast refresh
agentkit intel verify   # check inventories are fresh (exits 1 if stale)
```

### `agentkit list`
```bash
agentkit list           # show all available packs and bundles
```

---

## Prompts (Slash Commands)

| Prompt | Command | Use |
|--------|---------|-----|
| `dev-plan` | `/dev-plan` | Generate a structured TODO plan before writing code |
| `plan-review` | `/plan-review` | Architect reviews your plan — loops until approved |
| `wiki-init` | `/wiki-init` | Enrich `.ai/` wiki with real project knowledge |
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
| `wiki-audit-flag` | After source edits | Flag that wiki review is needed |

Domain packs add additional hooks (e.g. `go-fmt`, `terraform-fmt`, `ansible-lint`).

---

## User Profile Install

Install agents globally — available in every project without running `agentkit init` each time:

```bash
agentkit init --user    # installs to ~/.agentkit/
agentkit doctor --user  # verify
agentkit sync --user    # update
```

---

## Global `.gitignore`

agentkit installs local developer tooling that should **not** be committed. Use a global gitignore:

```bash
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
agentkit self-update   # SHA256 verified from v0.3.1
agentkit sync

# 5. Migrate workflow syntax (if upgrading from pre-v0.2.0)
forge doctor --check-legacy
forge migrate d15 apply

# 6. Use agentforge for daily work
# [agentforge] Review my changes in src/api.py
# [agentforge] Implement the login endpoint with JWT auth
# [jira-agent] Load PROJ-123 and hand off to developer
```

---

## FAQ

**Do I need Python or pip?**
No. The binary is fully self-contained.

**What changed in v0.3.1?**
Security hardening patch: (1) `agentkit self-update` now verifies SHA256 checksum before installing — mismatch aborts with exit 1, (2) TLS certificate bypass (`ssl.CERT_NONE`) removed — TLS failures abort cleanly, (3) `forge migrate d15 rollback` uses safe tar extraction (`filter="data"`) to block path traversal, (4) marketplace SSL hardened with explicit TLS context, (5) release pipeline now publishes `.sha256` files alongside every binary. No breaking changes — upgrade with `agentkit self-update`.

**What changed in v0.3.0?**
`agentkit init` is now the complete single workflow — installs agents for both Copilot (`.github/`) and Claude Code (`.claude/`) in one command. Python CLI tools deployed to `.agentkit/tools/` automatically. `.ai/` knowledge templates seeded on first init.

**What changed in v0.2.1?**
Patch release: (1) `agentkit self-update` SSL certificate failure on macOS fixed, (2) `agentkit --version` version reporting fixed, (3) legacy governance agents no longer appear in VS Code agent picker after D-15 migration, (4) domain pack skills now inject correctly into `agentforge`. No breaking changes.

**What changed in v0.2.0?**
The four governance agents (`developer`, `reviewer`, `architect`, `platform-engineer`) were merged into a single `agentforge` agent with mode routing. If you have workflow YAML files using the old syntax, run `forge migrate d15 apply` to update them automatically.

**Do I need to change anything when upgrading to v0.2.0+?**
Only if you have workflow YAML files using governance agent names directly. Run `forge doctor --check-legacy` — if it says clean, no action is needed.

**What is the difference between `agentkit update` and `agentkit self-update`?**
- `agentkit update` — upgrades via pip (use if installed with `pip install` or `pipx`)
- `agentkit self-update` — downloads and replaces the binary from GitHub Releases (use if installed via `curl`)

**Does `agentkit self-update` verify the download?**
Yes — from v0.3.1, every release includes a `.sha256` checksum file. The binary is verified before install. TLS is enforced with no bypass. Mismatch aborts with exit 1.

**What is the difference between `--pack` and `--bundle`?**
`--pack` is recommended — installs core + a complete domain-specific setup in one command. `--bundle` is for fine-grained control.

**Does this modify my `.gitignore`?**
No. agentkit never touches your `.gitignore`. Use the global gitignore approach above.

**Will `agentkit migrate` delete my agent memory?**
No. `migrate` only removes compiled tooling files. Everything in `.ai/` is preserved. Run `--dry-run` first to preview.

**Is my JIRA API token safe?**
Yes — read from `JIRA_API_TOKEN` env var at runtime. Never stored in files.

**Can the reviewer mode fix code?**
No. `agentforge` in reviewer mode enforces a hard-stop refusal when asked to write or commit. Switch to developer mode to apply findings.

**Can I roll back the D-15 migration?**
Yes — `forge migrate d15 apply` creates a backup at `.forge/migrate-d15-backup-*.tar.gz`. Run `forge migrate d15 rollback` to restore.

**I still see `developer`, `reviewer`, `architect` in my VS Code agent picker.**
Run `agentkit sync` to pull updated agent files. Or manually remove `.github/agents/developer.agent.md`, `reviewer.agent.md`, `architect.agent.md` — they are not part of the v0.2.0+ architecture.

---

## Releases

Binaries published for every release:

| Platform | Binary | SHA256 |
|----------|--------|--------|
| macOS Apple Silicon | `agentkit-macos-arm64` | `agentkit-macos-arm64.sha256` |
| macOS Intel | `agentkit-macos-amd64` | `agentkit-macos-amd64.sha256` |
| Linux x86_64 | `agentkit-linux-amd64` | `agentkit-linux-amd64.sha256` |
| Windows x86_64 | `agentkit-windows-amd64.exe` | `agentkit-windows-amd64.exe.sha256` |

See [Releases](https://github.com/JkhatriInfobox/agentkit/releases) for all versions.

**Latest: [v0.3.1](https://github.com/JkhatriInfobox/agentkit/releases/tag/v0.3.1)** — Security hardening: SHA256 integrity verification, TLS hardening, safe tar extraction, 20 new security tests.
