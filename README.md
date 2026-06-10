# agentkit

> AI Engineering Platform CLI — install governed agents, skills, hooks, and coding standards into any project in seconds.

Works with **GitHub Copilot** (VS Code) and **Claude** (Claude Code / Claude Desktop).

---

## Install

### macOS
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agents/releases/latest/download/agentkit-macos-arm64 \
  && chmod +x agentkit && sudo mv agentkit /usr/local/bin/
```

### Linux
```bash
curl -fsSL -o agentkit \
  https://github.com/JkhatriInfobox/agents/releases/latest/download/agentkit-linux-amd64 \
  && chmod +x agentkit && sudo mv agentkit /usr/local/bin/
```

### Windows
Download [`agentkit-windows-amd64.exe`](https://github.com/JkhatriInfobox/agents/releases/latest) from the latest release and add it to your PATH.

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
agentkit init --pack ansible-collection
agentkit init --pack python-automation

# Verify everything installed cleanly
agentkit doctor
```

Open VS Code or Claude Code — agents are ready in `.github/agents/`.

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
| `oss-maintenance` | Open-source project maintenance | oss-specific skills | — |

Each pack automatically includes `core` (developer, reviewer, architect, docs-writer, hooks, prompts). You do not need to install core separately.

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

Domain packs additionally install agent variants (e.g. `developer.agent.md` with Go/Terraform knowledge injected) and domain skills under `.github/skills/`.

---

## Agents

Select an agent from the `[Agent ▼]` dropdown in VS Code Copilot chat, or use `@agent-name` in Claude Code.

| Agent | Role | Capabilities |
|-------|------|-------------|
| `developer` | Implement features and fix bugs | Read + Write + Execute |
| `reviewer` | Review code — finds real bugs, never style issues | Read + Search + Execute |
| `architect` | Design systems, write ADRs, evaluate trade-offs | Read only (proposes, never implements) |
| `docs-writer` | Write README, API docs, changelogs | Read + Write |
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
[developer] Add input validation to the login endpoint
[reviewer] Review my changes in src/api.py
[architect] Design a caching strategy for the user service
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
agentkit init --pack ansible-collection    # install Ansible domain pack
agentkit init --pack python-automation     # install Python domain pack
agentkit init --bundle core,testing        # install specific bundles
agentkit init --force                      # overwrite locally modified files
agentkit init --user                       # install to user profile (~/.agentkit/)
agentkit init --project /path/to/repo      # install into a specific project path
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
# 1. One-time: set up global gitignore (see above)
git config --global core.excludesfile ~/.gitignore_global

# 2. New Go project setup
cd ~/my-go-project
agentkit init --pack go-development

# 3. New Terraform provider project
cd ~/my-tf-provider
agentkit init --pack terraform-provider

# 4. Daily use — open VS Code, select [developer] agent, start coding
# Agents are in .github/agents/ — Copilot picks them up automatically

# 5. When new agentkit version is released
agentkit sync

# 6. After git clean or if something looks wrong
agentkit doctor
```

---

## FAQ

**Do I need Python or pip?**
No. The binary is fully self-contained.

**What's the difference between `--pack` and `--bundle`?**
`--pack` is the recommended way to get started — it installs core + a complete domain-specific setup. `--bundle` is for advanced use when you want fine-grained control over exactly which components are installed.

**Does this modify my `.gitignore`?**
No. agentkit never touches your `.gitignore`. Use the [global gitignore](#global-gitignore) approach above to keep all agentkit files out of every repo automatically.

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

See [Releases](https://github.com/JkhatriInfobox/agents/releases) for all versions.

**Latest: [v0.1.7](https://github.com/JkhatriInfobox/agents/releases/tag/v0.1.7)** — Pack-first install + all 5 domain packs
