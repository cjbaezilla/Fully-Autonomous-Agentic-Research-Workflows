# OpenCode Tooling, Skills & Agents — Deep Dive

> How tools, agents, and skills work individually and together in OpenCode.

---

## Table of Contents

1. [Tools](#1-tools)
2. [Agents](#2-agents)
3. [Skills](#3-skills)
4. [How They Interact](#4-how-they-interact)
5. [Configuration Reference](#5-configuration-reference)
6. [Practical Examples](#6-practical-examples)

---

## 1. Tools

Tools are the **actions** an LLM can perform. Every tool call goes through a permission layer.

### 1.1 Built-in Tools

| Tool | Purpose | Permission key |
|------|---------|----------------|
| `bash` | Run shell commands | `bash` |
| `edit` | Exact string replacement in files | `edit` |
| `write` | Create/overwrite files | `edit` (shared) |
| `patch` | Apply diff patches | `edit` (shared) |
| `read` | Read file contents | `read` |
| `grep` | Regex content search (ripgrep) | `grep` |
| `glob` | File pattern matching | `glob` |
| `list` | List directory contents | `list` |
| `lsp` | LSP code intelligence (experimental) | `lsp` |
| `skill` | Load a skill's SKILL.md content | `skill` |
| `todowrite` | Create/update task lists | `todowrite` |
| `todoread` | Read task lists | `todoread` |
| `webfetch` | Fetch web page content | `webfetch` |
| `websearch` | Search the web (Exa AI) | `websearch` |
| `question` | Ask the user questions | `question` |
| `task` | Invoke a subagent | (implicit) |

### 1.2 Permission Levels

Every tool has three possible permission states:

- **`allow`** — Runs immediately, no prompts.
- **`ask`** — Prompts the user for approval before each execution.
- **`deny`** — Completely disabled for that agent/session.

### 1.3 Global Tool Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "edit": "deny",
    "bash": "ask",
    "webfetch": "allow"
  }
}
```

### 1.4 Wildcard Permissions

Use `*` to control multiple tools at once:

```json
{
  "permission": {
    "mymcp_*": "ask"
  }
}
```

### 1.5 Bash Command-Level Permissions

Permissions can be granular to specific bash commands using glob patterns:

```json
{
  "agent": {
    "build": {
      "permission": {
        "bash": {
          "*": "ask",
          "git status *": "allow",
          "git diff *": "allow",
          "git push": "deny"
        }
      }
    }
  }
}
```

The **last matching rule wins** — put the wildcard `*` first, specific rules after.

### 1.6 Extensions Beyond Built-in

- **Custom tools**: Define your own functions in config. See [Custom Tools](https://opencode.ai/docs/custom-tools/).
- **MCP servers**: Integrate external services (databases, APIs, third-party) via Model Context Protocol. See [MCP Servers](https://opencode.ai/docs/mcp-servers/).
- **Ignore patterns**: `.ignore` file can override `.gitignore` for `grep`/`glob`/`list` (ripgrep-based).

---

## 2. Agents

Agents are **specialized AI assistants** with their own prompts, models, tool access, and permissions. They let you create purpose-driven behaviors.

### 2.1 Agent Types

| Type | Role | How it's invoked |
|------|------|------------------|
| **Primary** | Main assistant the user talks to directly | Tab key / `switch_agent` keybind |
| **Subagent** | Specialized helper invoked by primary agents or via `@` mention | Automatic or `@name` |

### 2.2 Built-in Agents

| Agent | Mode | Behavior |
|-------|------|----------|
| **Build** | Primary | Full access, all tools enabled. Default agent. |
| **Plan** | Primary | Restricted: edits/patches/writes and bash default to `ask`. Read-only analysis. |
| **General** | Subagent | Full tool access (except todo). Multi-step research and execution. |
| **Explore** | Subagent | Read-only. Fast codebase exploration. Cannot modify files. |
| **Compaction** | Primary (hidden) | Automatic context compression for long sessions. |
| **Title** | Primary (hidden) | Auto-generates short session titles. |
| **Summary** | Primary (hidden) | Auto-creates session summaries. |

### 2.3 Agent Configuration Options

| Option | Description |
|--------|-------------|
| `description` | **Required.** What the agent does, when to use it. |
| `mode` | `primary`, `subagent`, or `all` (default). |
| `model` | Override the LLM for this agent (e.g., `anthropic/claude-sonnet-4-20250514`). |
| `prompt` | Path to custom system prompt file: `{file:./prompts/review.txt}`. |
| `temperature` | 0.0–1.0. Lower = deterministic, higher = creative. |
| `top_p` | Alternative randomness control (0.0–1.0). |
| `steps` | Max agentic iterations before forced text response. |
| `tools` | Enable/disable specific tools: `{ "write": false, "bash": true }`. |
| `permission` | Per-agent permission overrides (edit, bash, webfetch, skill, task). |
| `disable` | `true` to remove the agent. |
| `hidden` | `true` hides from `@` autocomplete (subagents only). |
| `color` | UI color: hex (`#FF5733`) or theme name (`accent`, `error`, etc.). |
| `reasoningEffort` | Provider-specific: OpenAI reasoning effort level. |
| *any other* | Passed through directly to the provider as model options. |

### 2.4 Defining Agents in JSON

```json
{
  "agent": {
    "code-reviewer": {
      "description": "Reviews code for best practices and potential issues",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "prompt": "You are a code reviewer. Focus on security, performance, and maintainability.",
      "tools": {
        "write": false,
        "edit": false
      },
      "permission": {
        "bash": {
          "*": "deny",
          "git diff *": "allow"
        }
      }
    }
  }
}
```

### 2.5 Defining Agents in Markdown

Place files in `~/.config/opencode/agents/` (global) or `.opencode/agents/` (project).

The filename becomes the agent name: `security-auditor.md` → agent `security-auditor`.

```markdown
---
description: Performs security audits and identifies vulnerabilities
mode: subagent
tools:
  write: false
  edit: false
permission:
  bash:
    "*": deny
    "grep *": allow
---

You are a security expert. Focus on identifying potential security issues.
Look for:
- Input validation vulnerabilities
- Authentication and authorization flaws
- Data exposure risks
- Dependency vulnerabilities
```

### 2.6 Task Permissions

Control which subagents an agent can invoke via the `task` tool:

```json
{
  "agent": {
    "orchestrator": {
      "mode": "primary",
      "permission": {
        "task": {
          "*": "deny",
          "orchestrator-*": "allow",
          "code-reviewer": "ask"
        }
      }
    }
  }
}
```

Last matching rule wins. Users can always invoke any subagent via `@` regardless of task permissions.

---

## 3. Skills

Skills are **reusable instruction sets** loaded on-demand. They're Markdown files with frontmatter that agents discover and load via the `skill` tool.

### 3.1 File Placement

Each skill is a folder containing `SKILL.md`. OpenCode searches these locations:

| Location | Scope |
|----------|-------|
| `.opencode/skills/<name>/SKILL.md` | Project |
| `~/.config/opencode/skills/<name>/SKILL.md` | Global |
| `.claude/skills/<name>/SKILL.md` | Project (Claude-compatible) |
| `~/.claude/skills/<name>/SKILL.md` | Global (Claude-compatible) |
| `.agents/skills/<name>/SKILL.md` | Project (agent-compatible) |
| `~/.agents/skills/<name>/SKILL.md` | Global (agent-compatible) |

### 3.2 Discovery

- **Project-local**: Walks up from CWD to the git worktree, loading matching skills along the way.
- **Global**: Loaded from home directory paths.

### 3.3 Frontmatter Requirements

```yaml
---
name: git-release              # Required. 1-64 chars, lowercase alphanumeric + hyphens
description: Create consistent releases and changelogs  # Required. 1-1024 chars
license: MIT                   # Optional
compatibility: opencode        # Optional
metadata:                      # Optional string-to-string map
  audience: maintainers
  workflow: github
---
```

**Name rules:**
- Lowercase alphanumeric with single hyphen separators
- Must match the directory name
- Regex: `^[a-z0-9]+(-[a-z0-9]+)*$`

### 3.4 Full Skill Example

`.opencode/skills/git-release/SKILL.md`:

```markdown
---
name: git-release
description: Create consistent releases and changelogs
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---

## What I do
- Draft release notes from merged PRs
- Propose a version bump
- Provide a copy-pasteable `gh release create` command

## When to use me
Use this when you are preparing a tagged release.
Ask clarifying questions if the target versioning scheme is unclear.
```

### 3.5 How the Skill Tool Works

The agent sees available skills in the `skill` tool description:

```
<available_skills>
  <skill>
    <name>git-release</name>
    <description>Create consistent releases and changelogs</description>
  </skill>
</available_skills>
```

The agent loads a skill by calling:

```
skill({ name: "git-release" })
```

This returns the full SKILL.md content into the conversation context.

### 3.6 Skill Permissions

Control access with pattern-based permissions:

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

| Permission | Behavior |
|------------|----------|
| `allow` | Skill loads immediately |
| `deny` | Skill hidden from agent, access rejected |
| `ask` | User prompted before loading |

### 3.7 Override Per Agent

**In agent frontmatter (Markdown):**

```markdown
---
description: Documentation writer
permission:
  skill:
    "documents-*": "allow"
---
```

**In config (JSON):**

```json
{
  "agent": {
    "plan": {
      "permission": {
        "skill": {
          "internal-*": "allow"
        }
      }
    }
  }
}
```

### 3.8 Disable the Skill Tool Entirely

```json
{
  "agent": {
    "plan": {
      "tools": {
        "skill": false
      }
    }
  }
}
```

---

## 4. How They Interact

### 4.1 The Full Flow

```
User message
    │
    ▼
Primary Agent (Build/Plan/custom)
    │
    ├─→ Uses tools directly (bash, edit, read, grep, etc.)
    │       │
    │       └─→ Each tool call → permission check → allow/ask/deny
    │
    ├─→ Loads skills via `skill` tool
    │       │
    │       └─→ Permission check → SKILL.md content injected into context
    │
    └─→ Invokes subagents via `task` tool
            │
            ├─→ task permission check (which subagents can be invoked?)
            │
            └─→ Subagent runs in child session
                    │
                    └─→ Has its own tools, permissions, model, prompt
```

### 4.2 Layered Permission Model

Permissions cascade and override at each level:

```
Global config (opencode.json)
    ↓  can be overridden by
Per-agent config (agent.tools / agent.permission)
    ↓  can be overridden by
Markdown agent frontmatter (tools / permission fields)
```

**Example cascade:**

```json
{
  "permission": {
    "edit": "allow",
    "bash": "ask"
  },
  "agent": {
    "plan": {
      "permission": {
        "edit": "deny",
        "bash": "deny"
      }
    }
  }
}
```

- **Build agent**: `edit` = allow, `bash` = ask (inherits global)
- **Plan agent**: `edit` = deny, `bash` = deny (overrides global)

### 4.3 Tools Inside Agents

Each agent declares which tools it has access to via the `tools` config. This is separate from permissions:

```json
{
  "agent": {
    "explore": {
      "tools": {
        "write": false,
        "edit": false,
        "bash": false,
        "read": true,
        "grep": true,
        "glob": true
      }
    }
  }
}
```

`tools: false` removes the tool entirely. `permission: "deny"` blocks access but the tool may still appear in descriptions.

### 4.4 Skills + Agents

Skills are tool-agnostic knowledge that any agent can load:

1. Agent receives the `skill` tool description listing available skills.
2. Agent decides (based on task context) which skill to load.
3. `skill` permission is checked (global → agent-level override).
4. SKILL.md content is injected into the conversation.
5. Agent follows the skill's instructions using its available tools.

**Example workflow:**

```
User: "Prepare a release for v2.1.0"
    │
    ▼
Build agent sees skill "git-release" is available
    │
    ▼
Calls skill({ name: "git-release" })
    │
    ▼
SKILL.md content loaded — agent now knows the release workflow
    │
    ▼
Agent uses its tools (bash, read, edit) to execute the workflow
```

### 4.5 Subagent Orchestration

Primary agents can delegate to subagents:

```
User: "Review all changed files for security issues"
    │
    ▼
Build agent (primary)
    │
    ├─→ task({ subagent_type: "explore", prompt: "Find all files changed since last commit" })
    │
    ├─→ task({ subagent_type: "code-reviewer", prompt: "Analyze file1.ts for vulnerabilities" })
    │
    ├─→ task({ subagent_type: "code-reviewer", prompt: "Analyze file2.ts for vulnerabilities" })
    │
    └─→ Aggregates results, presents summary to user
```

Each subagent:
- Runs in its own session (isolated context)
- Has its own model, tools, permissions, and prompt
- Returns results to the parent agent
- Can be navigated to via `session_child_first` / `session_child_cycle` keybinds

### 4.6 Skills + Subagents

Subagents can also load skills. A security-auditor subagent might have a skill for OWASP checks:

```json
{
  "agent": {
    "security-auditor": {
      "mode": "subagent",
      "permission": {
        "skill": {
          "owasp-*": "allow"
        }
      }
    }
  }
}
```

---

## 5. Configuration Reference

### 5.1 opencode.json (Full Example)

```json
{
  "$schema": "https://opencode.ai/config.json",

  "permission": {
    "edit": "allow",
    "bash": "ask",
    "webfetch": "allow",
    "skill": {
      "*": "allow",
      "internal-*": "deny"
    }
  },

  "agent": {
    "build": {
      "mode": "primary",
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.3,
      "prompt": "{file:./prompts/build.txt}",
      "tools": {
        "write": true,
        "edit": true,
        "bash": true,
        "read": true,
        "grep": true,
        "glob": true,
        "skill": true,
        "task": true,
        "webfetch": true,
        "question": true
      },
      "permission": {
        "bash": {
          "*": "ask",
          "git status *": "allow",
          "git diff *": "allow",
          "npm test *": "allow"
        }
      }
    },

    "plan": {
      "mode": "primary",
      "model": "anthropic/claude-haiku-4-20250514",
      "temperature": 0.1,
      "tools": {
        "write": false,
        "edit": false,
        "patch": false,
        "bash": false,
        "skill": true
      },
      "permission": {
        "skill": {
          "*": "allow"
        }
      }
    },

    "code-reviewer": {
      "description": "Reviews code for best practices and potential issues",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "prompt": "You are a code reviewer. Focus on security, performance, and maintainability.",
      "temperature": 0.1,
      "tools": {
        "write": false,
        "edit": false,
        "bash": false
      },
      "permission": {
        "bash": {
          "*": "deny",
          "git diff *": "allow",
          "git log *": "allow"
        }
      }
    },

    "explore": {
      "description": "Fast read-only codebase exploration",
      "mode": "subagent",
      "hidden": false,
      "tools": {
        "read": true,
        "grep": true,
        "glob": true,
        "list": true,
        "skill": true,
        "write": false,
        "edit": false,
        "bash": false
      }
    }
  }
}
```

### 5.2 Markdown Agent Example

`.opencode/agents/docs-writer.md`:

```markdown
---
description: Writes and maintains project documentation
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.3
tools:
  bash: false
permission:
  skill:
    "docs-*": "allow"
---

You are a technical writer. Create clear, comprehensive documentation.

Focus on:
- Clear explanations
- Proper structure
- Code examples
- User-friendly language
```

---

## 6. Practical Examples

### 6.1 Read-only Code Explorer

An agent that can only read and search, used for onboarding new developers:

```json
{
  "agent": {
    "onboarder": {
      "description": "Helps new developers understand the codebase",
      "mode": "subagent",
      "tools": {
        "read": true,
        "grep": true,
        "glob": true,
        "list": true,
        "skill": true,
        "write": false,
        "edit": false,
        "bash": false,
        "task": false
      },
      "permission": {
        "skill": {
          "architecture-*": "allow",
          "conventions-*": "allow"
        }
      }
    }
  }
}
```

### 6.2 CI/CD Pipeline Agent

An agent that runs tests and checks but can't modify source code:

```json
{
  "agent": {
    "ci-runner": {
      "description": "Runs tests, linters, and build checks",
      "mode": "subagent",
      "tools": {
        "bash": true,
        "read": true,
        "write": false,
        "edit": false
      },
      "permission": {
        "bash": {
          "*": "deny",
          "npm test *": "allow",
          "npm run lint *": "allow",
          "npm run build *": "allow",
          "npm run typecheck *": "allow"
        }
      }
    }
  }
}
```

### 6.3 Skill-Driven Workflow

Create a skill for database migrations:

`.opencode/skills/db-migration/SKILL.md`:

```markdown
---
name: db-migration
description: Guide for creating and running database migrations safely
license: MIT
metadata:
  database: postgres
  tool: prisma
---

## Pre-flight checks
1. Read the current schema from `prisma/schema.prisma`
2. Check existing migrations in `prisma/migrations/`
3. Verify no pending migrations: `npx prisma migrate status`

## Creating a migration
1. Modify `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name <descriptive-name>`
3. Review the generated SQL in the migration folder
4. Update any affected TypeScript types

## Rollback procedure
1. Do NOT use `prisma migrate reset` in production
2. Create a new migration that reverses the changes
3. Test on staging before production
```

Then an agent uses it:

```json
{
  "agent": {
    "db-admin": {
      "description": "Handles database schema changes and migrations",
      "mode": "subagent",
      "permission": {
        "skill": {
          "db-*": "allow"
        },
        "bash": {
          "*": "ask",
          "npx prisma *": "allow"
        }
      }
    }
  }
}
```

### 6.4 Multi-Agent Pipeline

Orchestrate multiple agents for a complex task:

```
User: "Refactor the authentication module"
    │
    ▼
Build agent (primary)
    │
    ├─→ @explore "Map all files in src/auth/ and their dependencies"
    │       Returns: file list, import graph
    │
    ├─→ @code-reviewer "Analyze current auth implementation for issues"
    │       Returns: security findings, code smells
    │
    ├─→ skill({ name: "refactor-checklist" })
    │       Loads: step-by-step refactoring guide
    │
    ├─→ Makes changes using edit/write tools
    │
    ├─→ @ci-runner "Run tests for auth module"
    │       Returns: test results
    │
    └─→ Presents summary with all findings and changes
```

---

## Summary

| Concept | What it is | Key file | Tool used |
|---------|-----------|----------|-----------|
| **Tool** | An action the LLM can perform | `opencode.json` (permissions) | — |
| **Agent** | A specialized AI assistant with its own config | `opencode.json` or `.md` in agents dir | `task` (to invoke subagents) |
| **Skill** | Reusable instructions loaded on-demand | `SKILL.md` in skills dirs | `skill` |

The three layers compose as follows:
- **Agents** define *who* is working (prompt, model, personality).
- **Tools** define *what* they can do (capabilities, actions).
- **Skills** define *how* they should approach specific tasks (knowledge, procedures).
- **Permissions** control *which* of the above are accessible at each level.
