# agentkit

> AI Engineering Platform CLI — install governed agents, skills, hooks, and coding standards into any project in seconds.

Works with **GitHub Copilot** (VS Code) and **Claude** (Claude Code / Claude Desktop).

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

# Install core agents (developer, reviewer, architect, docs-writer)
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
| `oss-maintenance` | Open-source project maintenance | oss-specific skills | — |

Each pack automatically includes `core` (developer, reviewer, architect, docs-writer, hooks, prompts). You do not need to install core separately.

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

# Custom fields
[jira-agent] What custom fields does our project use? Show values from PROJ-1
[jira-agent] Set story points on PROJ-123 to 5
```

### Custom fields

On first use, the agent can discover all custom fields in your JIRA project:

```
[jira-agent] Discover custom fields on PROJ-1 and help me map them to config
```

The agent fetches all fields, shows which ones have values, and asks which you want to map. Approved mappings are saved to `.ai/jira-config.yaml`:

```yaml
jira:
  base_url: https://your-org.atlassian.net
  email: you@example.com
  default_project: PROJ
  custom_fields:
    story_points: customfield_10016
    sprint: customfield_10020
    acceptance_criteria: customfield_10034
```

### Developer handoff workflow

```
1. [jira-agent]  Load PROJ-123 — formats full context (description, ACs, comments, epic)
2. [jira-agent]  Hand off to developer
3. [developer]   Implement the work
4. [jira-agent]  Move to In Review + post comment   (asks approval)
5. [jira-agent]  Log 3h work time                  (asks approval)
6. [reviewer]    Review the diff
7. [jira-agent]  Move to Done                       (asks approval)
```

### Two agents installed

- **`jira-agent`** — full operations: read + write (with approval gate)
- **`jira-reader`** — read-only: search, view tickets, check worklogs, discover fields — no approval prompts ever

---

## What Gets Installed

`agentkit init` (core) installs files into your project:

```
.github/
  agents/          ← AI agents (use in VS Code Copilot / Claude Code)
    developer.agent.md
    reviewer.agent.md
    architect.agent.md
    docs-writer.agent.md
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

Domain packs additionally install agent variants and domain skills under `.github/skills/`. The `jira-integration` pack also installs `.ai/jira-config.yaml`.

---

## Agents

Select an agent from the `[Agent ▼]` dropdown in VS Code Copilot chat, or use `@agent-name` in Claude Code.

| Agent | Role | Capabilities |
|-------|------|-------------|
| `developer` | Implement features and fix bugs | Read + Write + Execute |
| `reviewer` | Review code — finds real bugs, never style issues | Read + Search + Execute |
| `architect` | Design systems, write ADRs, evaluate trade-offs | Read only (proposes, never implements) |
| `docs-writer` | Write README, API docs, changelogs | Read + Write |
| `jira-agent` | Full JIRA operations — writes require approval | Read + HTTP (write-gated) |
| `jira-reader` | Read JIRA issues and search — no write, no approvals | Read + HTTP |
| `oss-maintainer` | Issue triage, community responses, release notes | Read + Write |
| `oss-triager` | Classify and route incoming issues | Read only |
| `release-manager` | Changelog, version bump, tag, publish | Read + Write + Execute |
| `test-runner` | Run and fix pytest suites | Read + Execute |
| `script-writer` | Write automation scripts | Read + Write + Execute |

### How to use agents in VS Code Copilot
1. Open Copilot Chat (`Ctrl+Shift+I` / `Cmd+Shift+I`)
2. Click `[Agent ▼]` dropdown → select an agent
3. Type your request

### How to use agents in Claude Code
```
[jira-agent] Show me PROJ-123 and format it for development
[developer] Add input validation to the login endpoint
[reviewer] Review my changes in src/api.py
```

---

## Bundles

Bundles are groups of related agents, skills, and hooks. Install only what you need.

| Bundle | Contents | Use when |
|--------|----------|----------|
| `core` | developer, reviewer, architect, docs-writer, hooks, prompts | Every project (default) |
| `testing` | test-runner agent, pytest skill, testing instructions | Python projects with tests |
| `packaging` | release-manager agent, Python packaging skill | Libraries / packages |
| `scripting` | script-writer agent, debugging skill | Automation / scripts |
| `mcp` | Full MCP config (filesystem + git + github + fetch) | When you need GitHub MCP |
| `vscode` | VS Code extension recommendations | VS Code users |
| `oss-maintenance` | oss-maintainer, oss-triager, release-manager | Open source projects |

```bash
# Install a domain pack (recommended — includes core automatically)
agentkit init --pack go-development

# Or install individual bundles
agentkit init --bundle core
agentkit init --bundle core,testing,oss-maintenance

# See all packs and bundles
agentkit list
```

---

## Commands

### `agentkit init`
Install agents and supporting files into your project.

```bash
agentkit init                              # install core bundle (default)
agentkit init --pack go-development        # install Go domain pack (includes core)
agentkit init --pack terraform-provider    # install Terraform domain pack
agentkit init --pack jira-integration      # install JIRA integration pack
agentkit init --bundle core,testing        # install specific bundles
agentkit init --force                      # overwrite locally modified files
agentkit init --user                       # install to user profile (~/.agentkit/)
agentkit init --project /path/to/repo      # install into a specific project path
```

### `agentkit migrate`
Migrate a project from the old `make seed` setup to agentkit. Removes old compiled tooling, preserves all `.ai/` memory, then installs fresh via `agentkit init`.

```bash
agentkit migrate                           # migrate + install core
agentkit migrate --pack go-development     # migrate + install Go pack
agentkit migrate --pack jira-integration   # migrate + add JIRA
agentkit migrate --dry-run                 # preview what would change (no writes)
agentkit migrate --yes                     # skip confirmation prompt
```

### `agentkit sync`
Update installed files when a new version of agentkit is available. Skips files you've modified locally.

```bash
agentkit sync           # update all installed files (safe — skips local edits)
agentkit sync --force   # overwrite even locally modified files
agentkit sync --user    # sync user-profile install
```

### `agentkit doctor`
Verify all installed files are intact and match the expected checksums.

```bash
agentkit doctor         # check workspace install
agentkit doctor --user  # check user-profile install
```

### `agentkit list`
Show all available domain packs and bundles.

```bash
agentkit list
```

Output:
```
Domain Packs (install with --pack):
  go-development      Go services, CLIs, scripts
  terraform-provider  Terraform provider development
  ansible-collection  Ansible modules, plugins, collections
  python-automation   Python scripts, CLIs, automation tools
  jira-integration    JIRA ticket management with write-gated permission model
  oss-maintenance     Open-source project maintenance

Utility Bundles (install with --bundle):
  core        developer, reviewer, architect, docs-writer
  testing     test-runner + pytest skill
  ...
```

### `agentkit intel`
Generate and manage `.ai/repository/` — auto-generated intelligence about your codebase.

```bash
agentkit intel build    # full rebuild of all inventories
agentkit intel refresh  # fast refresh (file/tech/deps only)
agentkit intel verify   # check inventories are fresh (exits 1 if stale)
```

### `agentkit update`
Upgrade agentkit itself from the source repository.

```bash
agentkit update
```

---

## Prompts (Slash Commands)

After `agentkit init`, these prompts are available in VS Code Copilot via `/` in chat:

| Prompt | Command | Use |
|--------|---------|-----|
| `dev-plan` | `/dev-plan` | Generate a structured TODO plan before writing code |
| `plan-review` | `/plan-review` | Architect reviews your plan — loops until approved |
| `wiki-init` | `/wiki-init` | One-time: enrich `.ai/` wiki with real project knowledge |
| `wiki-audit` | `/wiki-audit` | Architect audits wiki after each change |
| `root-cause` | `/root-cause` | Investigate failures by actual execution before fixing |

---

## Hooks

Hooks run automatically during agent sessions:

| Hook | Trigger | Action |
|------|---------|--------|
| `ruff-format` | After every `.py` edit | Auto-format with ruff |
| `secret-guard` | Before any file write | Block secrets from being committed |
| `block-dangerous` | Before shell commands | Block `rm -rf`, `format`, etc. |
| `intel-refresh` | After source file edits | Refresh `.ai/repository/` intelligence |
| `wiki-audit-flag` | After source edits | Flag that architect wiki review is needed |

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

agentkit installs local developer tooling — agents, skills, hooks, and project intelligence — that is personal to your machine and should **not** be committed to your repositories.

The recommended approach is a **global gitignore** so you never accidentally commit these files from any project:

```bash
# 1. Create (or open) your global gitignore
touch ~/.gitignore_global

# 2. Tell git to use it (one-time setup)
git config --global core.excludesfile ~/.gitignore_global
```

Add the following to `~/.gitignore_global`:

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

> **Why global instead of per-repo?**
> These files are your personal AI tooling overlay — not part of the project's source code. A global gitignore keeps every repo clean without requiring every contributor to add the same entries to every `.gitignore`.

If you prefer per-repo ignoring, append the same block to the project's `.gitignore` file instead.

---

## Typical Workflow

```bash
# 1. One-time: set up global gitignore
git config --global core.excludesfile ~/.gitignore_global

# 2. Migrate existing project from make seed (if applicable)
cd ~/my-project
agentkit migrate --pack go-development

# 3. New project with JIRA + Go
cd ~/my-go-project
agentkit init --pack go-development
agentkit init --pack jira-integration
export JIRA_API_TOKEN=your_token

# 4. Daily: load a JIRA ticket and start work
# [jira-agent] Show PROJ-123 and hand off to developer
# [developer]  <implements the work>
# [jira-agent] Move PROJ-123 to In Review and log 3h

# 5. When new agentkit version is released
agentkit sync
```

---

## FAQ

**Do I need Python or pip?**
No. The binary is fully self-contained.

**What's the difference between `--pack` and `--bundle`?**
`--pack` is the recommended way to get started — it installs core + a complete domain-specific setup. `--bundle` is for advanced use when you want fine-grained control over exactly which components are installed.

**Does this modify my `.gitignore`?**
No. agentkit never touches your `.gitignore`. Use the [global gitignore](#global-gitignore) approach above to keep all agentkit files out of every repo automatically.

**Will `agentkit migrate` delete my agent memory?**
No. `migrate` only removes compiled tooling files. Everything in `.ai/` (memory, standards, patterns, decisions, repository intelligence) is preserved. Run `agentkit migrate --dry-run` first to see exactly what will change.

**Is my JIRA API token safe?**
Yes — the token is never stored in files. It's read from the `JIRA_API_TOKEN` environment variable at runtime. The config file (`.ai/jira-config.yaml`) only stores your base URL and email, both of which are non-sensitive.

**Can I customise the installed agents?**
Yes — edit any file in `.github/agents/`. `agentkit sync` will detect your changes and skip those files unless you pass `--force`.

**What is `.agentkit/installed.json`?**
The lockfile — tracks what was installed and the checksum of each file. Used by `sync` and `doctor`.

**What is `.ai/repository/`?**
Auto-generated project intelligence — file inventory, tech stack, dependency map, test coverage summary, ownership info. Generated by `agentkit intel build` and refreshed automatically by the `intel-refresh` hook.

---

## Releases

Binaries are published for every release:

- `agentkit-macos-arm64` — macOS Apple Silicon
- `agentkit-linux-amd64` — Linux x86_64
- `agentkit-windows-amd64.exe` — Windows x86_64

See [Releases](https://github.com/JkhatriInfobox/agentkit/releases) for all versions.

**Latest: [v0.1.8](https://github.com/JkhatriInfobox/agentkit/releases/tag/v0.1.8)** — JIRA integration pack, `agentkit migrate` command, pre-push CI hook
