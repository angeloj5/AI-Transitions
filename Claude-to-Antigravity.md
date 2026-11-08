# 🚀 Antigravity Customizations Guide (`.claude` → `.agents`)

In Antigravity, the equivalent of a project-level `.claude/` folder is **`.agents/`** (or `.agent/`) at your repository root, and the global equivalent of `~/.claude/` is **`~/.gemini/config/`**.

---

## 🗺️ Side-by-Side Mapping (`.claude` → Antigravity)

| Component | Claude Code | Antigravity (Project / Workspace) | Antigravity (Global) |
| :--- | :--- | :--- | :--- |
| **Root Folder** | `.claude/` | `.agents/` (or `.agent/`) | `~/.gemini/config/` |
| **Instructions / Rules** | `CLAUDE.md` or `.claude/CLAUDE.md` | `AGENTS.md`, `GEMINI.md`, or `.agents/rules/*.md` | `~/.gemini/GEMINI.md` or `~/.gemini/config/rules/*.md` |
| **Skills** | `.claude/skills/<name>/SKILL.md` | `.agents/skills/<name>/SKILL.md` | `~/.gemini/config/skills/<name>/SKILL.md` |
| **Slash Commands** | `.claude/commands/<name>.md` | *(Unified with Skills)* `.agents/skills/<name>/SKILL.md` | `~/.gemini/config/skills/<name>/SKILL.md` |
| **Subagents** | `.claude/agents/<name>.md` | `.agents/agents/<name>.md` | `~/.gemini/config/agents/<name>.md` |
| **Hooks** | `.claude/settings.json` (`hooks`) | `.agents/hooks.json` | `~/.gemini/config/hooks.json` |
| **MCP Servers** | `.mcp.json` / `~/.claude.json` | `.agents/mcp_config.json` | `~/.gemini/config/mcp_config.json` |
| **Plugins (Bundles)** | `.claude/plugins/<name>/` | `.agents/plugins/<name>/plugin.json` | `~/.gemini/config/plugins/<name>/plugin.json` |

---

## 📁 Example `.agents/` Directory Structure

```text
your-project/
├── AGENTS.md                        # Always-on project instructions (like CLAUDE.md)
└── .agents/
    ├── rules/
    │   └── typescript-style.md      # Scoped rules (always_on, glob, or model_decision)
    ├── skills/
    │   └── deploy-staging/
    │       ├── SKILL.md             # Replaces both skills and slash commands
    │       ├── scripts/             # Optional helper scripts/utilities
    │       ├── references/          # Optional detailed docs loaded on demand
    │       ├── examples/            # Optional reference implementations
    │       └── resources/           # Optional templates/assets
    ├── agents/
    │   └── code-reviewer.md         # Custom subagent definition
    ├── plugins/
    │   └── my-team-kit/
    │       └── plugin.json          # Bundled skills, rules, hooks, and MCP servers
    ├── hooks.json                   # Lifecycle hooks (e.g., pre/post tool execution)
    ├── mcp_config.json              # Project-scoped MCP servers
    ├── skills.json                  # Optional custom skill path pointers
    └── plugins.json                 # Optional custom plugin path pointers
```

---

## 🧩 Component Details & File Formats

### 1. ⚡ Commands and Skills (`skills/<name>/SKILL.md`)
In Antigravity, custom slash commands and skills are unified. Any skill placed in `skills/<name>/SKILL.md` is loaded on-demand by the agent **and** can be triggered directly by typing `/<name>` in chat.

```markdown
---
name: deploy-staging
description: Deploy the current branch to the staging environment and run smoke tests.
---
# Instructions
1. Run the preparation script: `[prepare.sh](./scripts/prepare.sh)`
2. Execute the deployment and verify smoke tests.
```
* Both `name` (lowercase, hyphenated, matching the folder name) and `description` are required in the YAML frontmatter.

---

### 2. 🤖 Subagents (`agents/<name>.md`)
Defined via Markdown with YAML frontmatter specifying tool permissions:

```markdown
---
name: code-reviewer
description: Reviews diffs for security and style issues.
subagent: true
mainAgent: false
tools: [read_file, search_web]
---
You are a senior security-focused code reviewer...
```
* **Common tool names**: `read_file`, `edit_file`, `write_file`, `run_command`, `manage_task`, `search_web`.

---

### 3. 📜 Rules (`AGENTS.md`, `GEMINI.md`, and `rules/<name>.md`)
In addition to root or subdirectory `AGENTS.md` / `GEMINI.md` files (which are always active for their directory tree), rules inside `.agents/rules/` support YAML frontmatter triggers:

```markdown
---
trigger: glob
globs: "**/*.ts, **/*.tsx"
---
Always prefer interfaces over type aliases for public APIs.
```
* **Supported triggers**: `always_on`, `glob`, `model_decision`, `manual`.

---

### 4. 🪝 Lifecycle Hooks (`hooks.json`)
Whereas Claude Code puts hooks inside `.claude/settings.json`, Antigravity uses a dedicated `.agents/hooks.json` file. Each top-level key is a named hook group that can subscribe to lifecycle events (`SessionStart`, `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation`, `Stop`):

```json
{
  "safety-and-lint": {
    "PreToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/safety-check.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "edit_file|write_file",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/lint.sh"
          }
        ]
      }
    ]
  }
}
```
* **Input/Output**: Hooks receive context as JSON over `stdin` (in `camelCase`, e.g., `toolCall.name`, `toolCall.args`) and return JSON on `stdout` (for example, `{"decision": "allow" | "deny" | "ask" | "force_ask", "reason": "..."}` in `PreToolUse`).

---

### 5. 🔌 MCP Servers (`mcp_config.json`)
Placed at `~/.gemini/config/mcp_config.json` (global), `.agents/mcp_config.json`, or inside a plugin (`plugins/<name>/mcp_config.json`). Supports local **Stdio** servers and remote **Streamable HTTP** servers:

```json
{
  "mcpServers": {
    "sqlite-helper": {
      "command": "sqlite-mcp-server",
      "args": ["/path/to/database.db"],
      "env": {
        "DB_READONLY": "true"
      }
    },
    "remote-service": {
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

---

### 6. 📦 Plugins (`plugins/<name>/`)
To bundle rules, skills, subagents, hooks, and MCP servers into a single toggleable unit (similar to `.claude/plugins/`), place them under `.agents/plugins/<name>/` or `~/.gemini/config/plugins/<name>/`:

```text
.agents/plugins/my-team-kit/
├── plugin.json          # Required manifest: {"name": "my-team-kit", "description": "..."}
├── mcp_config.json      # Optional bundled MCP servers
├── hooks.json           # Optional bundled hooks (must be at plugin root)
├── agents/              # Optional bundled subagents
├── rules/               # Optional bundled rules (e.g., rules/AGENTS.md)
└── skills/              # Optional bundled skills
    └── release/
        └── SKILL.md
```

---

### 7. 🧭 Custom Paths & Precedence (`skills.json` / `plugins.json`)
* **Custom folders**: If your repo stores skills or plugins outside `.agents/` (e.g., `tools/ai/skills`), you can register them using `.agents/skills.json` or `.agents/plugins.json`:
  ```json
  {
    "entries": [
      { "path": "tools/ai/skills", "exclude": ["experimental-.*"] }
    ]
  }
  ```
* **Loading Precedence (highest to lowest)**:
  1. **Workspace (`.agents/`)** — walks up from the current working directory to the repository root (`.git`)
  2. **Workspace `skills.json` / `plugins.json`**
  3. **Global (`~/.gemini/config/`)**
  4. **Built-in customizations**
  5. **Global declared JSON configs**

---

## 💡 Nuances & Best Practices

* **🧠 Progressive Disclosure (Context Efficiency)**:
  * **Skills**: Only the `name` and `description` from each `SKILL.md` frontmatter are injected into the system prompt by default. The full `SKILL.md` (and any linked files in `scripts/`, `references/`, `examples/`, or `resources/`) is only read on-demand when the skill is triggered.
  * **Rules**: Each rule file is capped at **24 KB** (and supports `@[label](path)` file includes). Always-on rules share a **20,000-token** budget; if exceeded, excess rules are automatically converted to file-path pointers so the agent can read them on demand rather than bloating every turn.
* **📂 Subdirectory Rules (`AGENTS.md` / `GEMINI.md`)**:
  Just like placing `CLAUDE.md` in nested folders, you can drop a standalone `AGENTS.md` or `GEMINI.md` into any subdirectory (e.g., `frontend/AGENTS.md`). As the agent reads or edits files in `frontend/`, it walks up the directory tree to the repository root and automatically loads every `AGENTS.md` / `GEMINI.md` along the path.
* **🔄 Migrating Existing Claude Configurations**:
  * Ask Antigravity directly to *"migrate my Claude settings"* to automatically convert `~/.claude` skills, commands, subagents, `CLAUDE.md` rules, and MCP servers into an Antigravity plugin.
  * For Claude Code plugins (`~/.claude/plugins/`), you can also run `agy plugin import claude` via the CLI.

---

## 📝 Change control log

Record every change made to this guide so edits stay traceable.

| Date | Version | Author | Section(s) affected | Description of change |
| :--- | :--- | :--- | :--- | :--- |
| 2026-11-07 | 1.1 | @angeloj5 | All headings, Change control log | Added emojis to headings and key bullets; added this Change control log section. |
| 2026-11-07 | 1.0 | @angeloj5 | All | Initial version of the guide. |
