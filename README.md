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

# Verify everything installed cleanly
agentkit doctor
```

That's it. Open VS Code or Claude Code — agents are ready in `.github/agents/`.

---

## What Gets Installed

`agentkit init` installs files into your project:

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
# Install a single bundle
agentkit init --bundle core

# Install multiple bundles at once
agentkit init --bundle core,testing,oss-maintenance

# See all bundles and their contents
agentkit list
```

---

## Commands

### `agentkit init`
Install agents and supporting files into your project.

```bash
agentkit init                              # install core bundle (default)
agentkit init --bundle core,testing       # install multiple bundles
agentkit init --bundle oss-maintenance    # add OSS agents to existing install
agentkit init --force                     # overwrite locally modified files
agentkit init --user                      # install to user profile (~/.agentkit/)
agentkit init --project /path/to/repo     # install into a specific project path
```

### `agentkit sync`
Update installed files when a new version of agentkit is available. Skips files you've modified locally.

```bash
agentkit sync           # update all installed files (safe — skips local edits)
agentkit sync --force   # overwrite even locally modified files
agentkit sync --user    # sync user-profile install
```

### `agentkit doctor`
Verify all installed files are intact and match the expected checksums. Run this after `git clean` or if something feels off.

```bash
agentkit doctor         # check workspace install
agentkit doctor --user  # check user-profile install
```

Output:
```
agentkit doctor — 18 file(s) tracked
✓ 18 file(s) clean
All files clean.
```

### `agentkit list`
Show all available bundles and the files each one installs.

```bash
agentkit list
```

### `agentkit intel`
Generate and manage `.ai/repository/` — auto-generated intelligence about your codebase (file inventory, tech stack, dependencies, test coverage, ownership).

```bash
agentkit intel build    # full rebuild of all inventories
agentkit intel refresh  # fast refresh (file/tech/deps only)
agentkit intel verify   # check inventories are fresh (exits 1 if stale)
```

Intel runs automatically after `agentkit init`. Agents use `.ai/repository/` to understand your project without re-scanning every time.

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

---

## User Profile Install

Install agents globally so they're available in every project without running `agentkit init` each time:

```bash
agentkit init --user    # installs to ~/.agentkit/
agentkit doctor --user  # verify
agentkit sync --user    # update
```

---

## Typical Workflow

```bash
# 1. New project setup (once)
cd ~/my-project
agentkit init --bundle core,testing

# 2. Daily use — open VS Code, select [developer] agent, start coding
# Agents are in .github/agents/ — Copilot picks them up automatically

# 3. When new agentkit version is released
agentkit sync

# 4. After git clean or if something looks wrong
agentkit doctor
```

---

## FAQ

**Do I need Python or pip?**
No. The binary is fully self-contained.

**Does this modify my `.gitignore`?**
No. Add `.agentkit/` to your `.gitignore` manually if you don't want the lockfile committed, or use a global gitignore.

**Can I customise the installed agents?**
Yes — edit any file in `.github/agents/`. `agentkit sync` will detect your changes and skip those files unless you pass `--force`.

**What is `.agentkit/installed.json`?**
The lockfile — tracks what was installed and the checksum of each file. Used by `sync` and `doctor`. Safe to commit or ignore.

**What is `.ai/repository/`?**
Auto-generated project intelligence — file inventory, tech stack, dependency map, test coverage summary, ownership info. Generated by `agentkit intel build` and refreshed automatically by the `intel-refresh` hook on every file edit.

---

## Releases

Binaries are published for every release:

- `agentkit-macos-arm64` — macOS Apple Silicon
- `agentkit-linux-amd64` — Linux x86_64
- `agentkit-windows-amd64.exe` — Windows x86_64

See [Releases](https://github.com/JkhatriInfobox/agentkit/releases) for all versions.
