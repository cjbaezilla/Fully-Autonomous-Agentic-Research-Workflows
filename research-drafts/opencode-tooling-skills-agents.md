# OpenCode Tooling, Skills & Agents Deep Dive

> How tools, agents, and skills work individually and together in OpenCode.

---

## Table of Contents

1. [Tools](#1-tools)
2. [Agents](#2-agents)
3. [Skills](#3-skills)
4. [How They Interact](#4-how-they-interact)
5. [Configuration Reference](#5-configuration-reference)
6. [Practical Examples](#6-practical-examples)
7. [AGENTS.md - Project Rules and Custom Instructions](#7-agentsmd---project-rules-and-custom-instructions)
   - 7.1 [What AGENTS.md Is and Its Role in the Ecosystem](#71-what-agentsmd-is-and-its-role-in-the-ecosystem)
   - 7.2 [How AGENTS.md Interacts with Agents](#72-how-agentsmd-interacts-with-agents)
   - 7.3 [How AGENTS.md Interacts with Skills](#73-how-agentsmd-interacts-with-skills)
   - 7.4 [How AGENTS.md Interacts with Tools and Permissions](#74-how-agentsmd-interacts-with-tools-and-permissions)
   - 7.5 [Scope of AGENTS.md](#75-scope-of-agentsmd)
   - 7.6 [How AGENTS.md Is Configured](#76-how-agentsmd-is-configured)
   - 7.7 [Precedence and Resolution](#77-precedence-and-resolution)
   - 7.8 [Claude Code Compatibility Layer](#78-claude-code-compatibility-layer)
   - 7.9 [Practical Patterns and Best Practices](#79-practical-patterns-and-best-practices)
   - 7.10 [Complete Ecosystem Diagram](#710-complete-ecosystem-diagram)
8. [Commands - Custom Prompt Workflows](#8-commands-custom-prompt-workflows)
   - 8.1 [What Commands Are and Why They Exist](#81-what-commands-are-and-why-they-exist)
   - 8.2 [Creating Commands via Markdown Files](#82-creating-commands-via-markdown-files)
   - 8.3 [Creating Commands via JSON Configuration](#83-creating-commands-via-json-configuration)
   - 8.4 [Prompt Configuration and Template Syntax](#84-prompt-configuration-and-template-syntax)
   - 8.5 [Command Options](#85-command-options)
   - 8.6 [Commands vs Skills vs Agents](#86-commands-vs-skills-vs-agents)
   - 8.7 [Built-in Commands and Overrides](#87-built-in-commands-and-overrides)
   - 8.8 [Practical Command Patterns](#88-practical-command-patterns)
9. [Custom Tools - Extending Agent Capabilities](#9-custom-tools-extending-agent-capabilities)
   - 9.1 [What Custom Tools Are and Their Role](#91-what-custom-tools-are-and-their-role)
   - 9.2 [Tool File Structure and the tool() Helper](#92-tool-file-structure-and-the-tool-helper)
   - 9.3 [Single Tool vs Multiple Tools Per File](#93-single-tool-vs-multiple-tools-per-file)
   - 9.4 [Tool Arguments and Schema Validation](#94-tool-arguments-and-schema-validation)
   - 9.5 [Tool Context and Session Awareness](#95-tool-context-and-session-awareness)
   - 9.6 [Name Collisions and Precedence](#96-name-collisions-and-precedence)
   - 9.7 [Tools in Any Language via Wrapper Pattern](#97-tools-in-any-language-via-wrapper-pattern)
   - 9.8 [Custom Tools vs MCP Servers vs Built-in Tools](#98-custom-tools-vs-mcp-servers-vs-built-in-tools)
   - 9.9 [Practical Custom Tool Examples](#99-practical-custom-tool-examples)
10. [Summary](#10-summary-revised)

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

## 7. AGENTS.md - Project Rules and Custom Instructions

AGENTS.md files are the central mechanism for providing persistent custom instructions to OpenCode. They act as the constitution for your AI assistant, establishing ground rules, conventions, and context that apply across all agents and sessions. Unlike agents, which are specialized AI assistants with specific roles, or skills, which are reusable instruction sets loaded on demand, AGENTS.md represents the foundational layer of project-wide or globally applicable guidance that every agent inherits automatically.

### 7.1 What AGENTS.md Is and Its Role in the Ecosystem

AGENTS.md is a markdown file that contains custom instructions which OpenCode automatically injects into the system prompt of every agent session. Think of it as a playbook that all your AI assistants must read and follow whenever they work within a particular context. This file exists at two primary levels: project-level, stored as AGENTS.md in your project root directory, and global-level, stored at ~/.config/opencode/AGENTS.md. When you start an OpenCode session within a project, the system automatically discovers and loads the relevant AGENTS.md files from the current directory and its ancestors, building a cumulative instruction set that travels with you as you navigate the project structure.

The role of AGENTS.md differs fundamentally from agents and skills. An agent is a specialized assistant with its own identity, model configuration, tool permissions, and custom prompt. For instance, you might have a "code-reviewer" agent that focuses specifically on security analysis, or a "plan" agent that works in read-only mode. These agents are invoked explicitly, either as primary agents you switch to or as subagents that primary agents delegate work to. Skills, on the other hand, are reusable chunks of knowledge about specific workflows or domains, like "git-release" or "db-migration." Skills are loaded on demand when an agent determines they are relevant to the task at hand.

AGENTS.md sits beneath both of these layers. It is not an agent itself; it does not have a personality or specific role. It is not a skill that gets invoked only for particular tasks. Instead, AGENTS.md represents the baseline rules and context that persist throughout the entire session, regardless of which agent is active. Every agent, whether the primary build agent, the plan agent, or any custom subagent, receives the AGENTS.md content as part of its system instructions. This makes AGENTS.md the authoritative source for project conventions, coding standards, architectural decisions, and workflow preferences that all team members and all AI assistants must adhere to.

In practical terms, AGENTS.md serves as your project's "constitution." It establishes the non-negotiable rules: authentication flows must use OAuth2, all code must include TypeScript types, tests must be placed alongside source files, and commit messages must follow conventional commits. These instructions are injected automatically, so you do not need to remember to invoke a skill or switch to a specific agent to get guidance on project standards. The moment an AI assistant starts working in your project, it knows the rules because they are part of its foundational prompt.

### 7.2 How AGENTS.md Interacts with Agents

The interaction between AGENTS.md and agents operates on a layering principle. When an agent session begins, OpenCode assembles the complete system prompt by gathering instructions from multiple sources, with AGENTS.md forming the base layer. The final system prompt that the language model sees includes the AGENTS.md content, followed by any agent-specific prompt defined in the agent's configuration. This layering means that agent-specific instructions can refine, elaborate on, or in some cases override the general project rules, but they cannot entirely negate the core constraints established in AGENTS.md.

To understand this relationship, consider a project with a root AGENTS.md file containing the instruction: "All code changes must include corresponding unit tests with at least 80% coverage." Now suppose you have defined a "quick-fix" agent in opencode.json with the prompt: "You are a fast code repair assistant. Make minimal changes to fix bugs, skip writing tests for trivial fixes." When you invoke the quick-fix agent, the language model receives both sets of instructions. The AGENTS.md rule about tests is present in the prompt, and the agent-specific prompt adds nuance. The agent might interpret that "trivial fixes" can bypass the test requirement, but it still operates within the broader framework established by AGENTS.md. The two instruction sets coexist and must be reconciled by the model.

The precedence between these layers is not a simple override mechanism where one completely replaces the other. Instead, they are concatenated into a single system prompt, with the order typically being: global AGENTS.md, then project AGENTS.md (from nearest to furthest up the directory tree), then agent-specific prompt content. The language model sees all of this context and must synthesize it. In practice, more specific instructions tend to take precedence in the model's behavior because they are closer to the task description, but the general rules remain in effect as constraints.

This layering has important implications for how you design your instructions. AGENTS.md should contain project-wide policies that you want all agents to follow, regardless of their specific role. Agent-specific prompts should contain role definitions and situational guidance that adapt the general policies to the agent's particular function. For example, AGENTS.md might state: "Use TypeScript for all new code." The explore agent's prompt can add: "When exploring code, identify TypeScript configuration files and key type definitions." The build agent's prompt can add: "When implementing features, ensure strict typing and run tsc --noEmit to validate." Both agents follow the TypeScript rule, but they apply it in ways appropriate to their roles.

### 7.3 How AGENTS.md Interacts with Skills

AGENTS.md and skills operate on different axes: AGENTS.md provides always-present baseline rules, while skills provide domain-specific knowledge loaded on demand. The relationship between them is complementary and can be quite powerful when used intentionally.

AGENTS.md can reference skills by name and instruct agents about when to use them. For instance, your project AGENTS.md might contain a section like:

```
## Standard Workflows

When performing a release, use the git-release skill.
When modifying the database schema, consult the db-migration skill first.
When writing API endpoints, follow the rest-api-style-guide skill.
```

These instructions become part of the agent's baseline knowledge. An agent working in the project will see these references in its system prompt and can decide to load the relevant skill using the skill tool when the appropriate situation arises. The skill itself is not automatically loaded; the agent must still call the skill tool to retrieve the skill's detailed instructions. But the AGENTS.md reference primes the agent to know that such a skill exists and when it should be considered.

Conversely, AGENTS.md can also establish rules about skill usage. You might include instructions like: "Always review the documentation skill before writing user-facing documentation" or "Do not use experimental skills without explicit user approval." These behavioral guidelines remain in the agent's mind throughout the session and influence its decision-making about whether and when to invoke skills.

Skills themselves are separate files located in .opencode/skills/, ~/.config/opencode/skills/, or Claude-compatible locations. When an agent calls the skill tool with a skill name, the system loads the SKILL.md content and injects it into the conversation context. At that point, the skill's instructions become part of the immediate working context, sitting alongside the already-present AGENTS.md content. The agent now has both the project baseline rules and the skill-specific guidance available to guide its actions.

This layering enables sophisticated workflows. Imagine a project AGENTS.md that says: "All code must be reviewed by the code-reviewer agent" alongside a code-quality skill that contains a detailed checklist. The primary agent might first load the code-quality skill to understand the review criteria, then invoke the code-reviewer subagent, which itself has access to the same AGENTS.md rules plus its own specialized prompt. The result is a consistent application of standards across multiple layers of execution.

### 7.4 How AGENTS.md Interacts with Tools and Permissions

AGENTS.md can contain instructions about how tools should be used, but it cannot change the underlying permission system. This distinction is crucial: permissions are controlled by opencode.json or agent configuration and represent hard constraints (what tools are allowed or denied), whereas AGENTS.md represents soft guidance about how to use tools effectively and responsibly.

For example, AGENTS.md might contain instructions like: "Always use the Read tool before editing a file to understand its current state" or "When searching for code, prefer Grep over Glob for content searches" or "Batch similar Edit operations together to reduce round trips." These are best practices that help the agent work more efficiently and safely, but they are not enforced by the system. The agent could theoretically ignore them, though a well-crafted instruction set will encourage compliance.

Tool permissions, on the other hand, are authoritative. If opencode.json sets "bash": "deny" for a particular agent, that agent cannot execute bash commands regardless of what AGENTS.md says. If AGENTS.md contains "Use bash to run tests," that instruction is moot if the agent lacks bash permission. The permission system overrides any textual guidance.

This separation serves an important purpose. Permissions are about security and access control; they are binary decisions that the system enforces. AGENTS.md is about workflow optimization and knowledge sharing; it guides behavior within the bounds of what is permitted. You should use AGENTS.md to teach good habits and project conventions, and use permission configuration to enforce safety boundaries.

The interaction becomes interesting when AGENTS.md tries to guide around permission restrictions. Consider AGENTS.md that says: "If you need to run a test but bash is not available, ask the user to run it manually and provide the results." This is valid guidance: it acknowledges a limitation and provides a fallback strategy. The AGENTS.md instruction respects the permission reality while still moving the work forward.

Similarly, AGENTS.md can contain conditional logic about tool usage: "Use the Edit tool for small targeted changes; for larger refactorings, use the Patch tool if available." This kind of nuanced guidance helps agents make better decisions about which tools to employ when multiple options exist within their permission set.

### 7.5 Scope of AGENTS.md

AGENTS.md files operate at different scopes, and understanding these scopes is essential for managing instruction sets effectively.

Project scope is the most common and straightforward. An AGENTS.md file placed in the root of a project applies whenever you are working within that project's directory tree. If your project has subdirectories for different components, as long as you are somewhere under the project root, the root AGENTS.md is in effect. You can also place additional AGENTS.md files in subdirectories to create more specific rules for those subdirectories. OpenCode walks up the directory tree from the current working directory, collecting AGENTS.md files as it goes. This means that if you are in a subdirectory that contains its own AGENTS.md, both that local file and the project root file will be loaded, with the local file's content appearing later in the prompt (effectively allowing it to add to or refine the root rules for that specific location).

Global scope applies across all projects and sessions. The file at ~/.config/opencode/AGENTS.md is loaded for every OpenCode session, regardless of what project you are in. This is the place for personal preferences that you want everywhere: your preferred code style, your expectations for documentation, your general workflow preferences. Global AGENTS.md is always combined with any project-level AGENTS.md, with project rules taking precedence for overlapping topics simply because they appear later in the prompt and are therefore closer to the task.

Team scope emerges when AGENTS.md is committed to Git. Because AGENTS.md is just a file in your project repository, committing it means all team members automatically get the same set of project rules when they pull the code. This creates a powerful mechanism for establishing and maintaining consistent AI assistant behavior across the entire team. Everyone's OpenCode instance will load the same project AGENTS.md, ensuring that regardless of who is working on the code, the AI assistant provides guidance aligned with team standards.

Session scope refers to the fact that AGENTS.md is loaded once at the beginning of a session and persists throughout that session. The content is not re-read on every agent invocation or tool call; it becomes part of the session's system prompt. This means if you edit AGENTS.md during an active session, those changes will not take effect until you start a new session. The persistence ensures that the instruction set remains stable and predictable throughout a work session.

### 7.6 How AGENTS.md Is Configured

AGENTS.md configuration occurs through file placement, automatic generation via the /init command, manual creation, and through the instructions field in opencode.json. Each method serves different purposes and offers different capabilities.

File placement follows a simple convention: AGENTS.md is placed in the directory where you want it to apply. The most common placement is the project root directory, making it apply to the entire project. You can also place AGENTS.md in subdirectories to create rules that apply only when working within that subdirectory. When OpenCode needs to assemble the instruction set for a session, it starts from the current working directory and walks upward toward the root, loading every AGENTS.md file it finds along the way. This creates a cascade of rules, with directories closer to the current location having more specific influence.

The /init command provides automatic generation of AGENTS.md. When you run /init in a project, OpenCode scans the project structure to infer rules and conventions. It looks at existing configuration files like package.json, tsconfig.json, .eslintrc, and the codebase itself to deduce the technology stack, coding conventions, and project organization. It then generates a starter AGENTS.md file in the project root with these inferred rules. This automated initialization gives you a baseline that you can then edit and refine. The /init command is particularly valuable when starting with a new codebase or when onboarding a new project, as it quickly establishes a working set of instructions based on what already exists.

Manual creation is straightforward: create a file named AGENTS.md at the appropriate location and write your instructions in markdown format. The file can contain any content you would put in a system prompt (rules, examples, workflows, architectural notes, coding standards). There is no required format beyond being valid markdown. However, organizing the content with clear headings makes it easier to maintain and for the AI to parse. Example structure might include sections like Project Overview, Coding Standards, Testing Requirements, Git Workflow, and Common Pitfalls.

The instructions field in opencode.json offers an alternative or complementary method to specify custom instructions. This field can contain either a string with direct instruction text or an object with a file or url key to reference external content. For example:

```json
{
  "instructions": {
    "file": "./AGENTS.md"
  }
}
```

Or using a glob pattern to load multiple files:

```json
{
  "instructions": {
    "glob": "rules/**/*.md"
  }
}
```

The glob pattern capability allows you to modularize your rules into multiple files for better organization, while still having them combined into the session's instruction set. This is particularly useful for large projects with extensive rule sets that would be unwieldy in a single file.

Referencing external files can be done both in opencode.json, as shown above, and manually within AGENTS.md content itself. You might include in your AGENTS.md a line like: "See docs/ARCHITECTURE.md for system design details" which prompts the agent to read that file when needed. Or you might structure your AGENTS.md to include actual content from external files by copying it in, though that requires manual maintenance. The most powerful pattern is using opencode.json's glob to automatically include multiple markdown files from a rules directory, keeping each rule set focused on a specific domain.

Remote URL instructions are supported through the instructions field. You can specify:

```json
{
  "instructions": {
    "url": "https://example.com/project-rules.md"
  }
}
```

This allows teams to maintain shared rule sets in a central location and have all projects reference them. When using a remote URL, OpenCode fetches the content at session start and includes it in the instruction set. This enables corporate or team-wide standards that are maintained in one place but applied across many projects. Note that using remote instructions requires network access and introduces a dependency on the remote server being available.

### 7.7 Precedence and Resolution

The precedence system for AGENTS.md determines which instructions take effect when multiple files are applicable. Understanding this order is essential for predicting agent behavior and for organizing your instruction files properly.

The fundamental precedence order is: local files first, then global files. By "local files first," we mean that AGENTS.md files closer to the current working directory take precedence over those further away. Specifically, OpenCode traverses upward from the current directory to the git worktree root, loading each AGENTS.md it encounters. The files are incorporated into the system prompt in the order they are found, starting from the farthest (typically the project root) and ending with the nearest (potentially the current directory itself). This means that if two AGENTS.md files contain conflicting instructions, the one from the directory closer to your current location appears later in the prompt and thus tends to override the earlier one.

After project-level files, global AGENTS.md from ~/.config/opencode/AGENTS.md is loaded. Global instructions come after all project-level ones, giving project rules the ability to override personal preferences when there is a conflict. This design respects that project standards should take precedence over individual habits when working on a shared codebase.

The CLAUDE.md fallback operates at lower precedence than AGENTS.md. OpenCode supports Claude Code compatibility by looking for CLAUDE.md files in the same locations (project root and global ~/.claude/CLAUDE.md) if no AGENTS.md is found, or as supplementary sources. However, when both AGENTS.md and CLAUDE.md exist, AGENTS.md takes precedence and CLAUDE.md may be ignored entirely. This encourages migration to the AGENTS.md naming convention while maintaining backward compatibility for existing Claude Code users.

Within each category (project-local or global), the "first match wins" principle applies for certain types of content, but the overall concatenation means later files can effectively override earlier ones by restating rules. For configuration items that are meant to be replaced rather than accumulated, the file that appears later in the traversal order dominates. For narrative instructions that are additive, all content contributes.

The opencode.json instructions field integrates with this precedence system in a specific way. Content specified via the instructions field is combined with AGENTS.md content, but the exact ordering depends on configuration. Typically, instructions from opencode.json are added after or alongside AGENTS.md content, effectively giving them similar precedence to project-level rules. If you use glob patterns in opencode.json to load multiple files, those files are incorporated in the order specified by the glob expansion, but they are still part of the project-level instruction set.

To illustrate with an example: Suppose you have a monorepo with packages/frontend/AGENTS.md containing frontend-specific rules and packages/backend/AGENTS.md containing backend rules. When you work in packages/frontend/src, OpenCode loads packages/frontend/AGENTS.md, then packages/AGENTS.md (if it exists), then the project root AGENTS.md, then global AGENTS.md. The frontend-specific rules appear last and therefore have the strongest influence on agent behavior in that directory.

### 7.8 Claude Code Compatibility Layer

OpenCode maintains compatibility with Claude Code's instruction system through the CLAUDE.md fallback mechanism. This allows teams transitioning from Claude Code to OpenCode to retain their existing instruction files without immediate disruption.

The compatibility layer works as follows: When OpenCode starts a session, it searches for instruction files in this order: AGENTS.md in the project root, AGENTS.md in parent directories up to the worktree root, AGENTS.md in the global ~/.config/opencode/ directory, then CLAUDE.md in the same locations, then CLAUDE.md in the global ~/.claude/ directory. If AGENTS.md files are found, CLAUDE.md files are ignored. Only if no AGENTS.md files exist does OpenCode fall back to loading CLAUDE.md files. This ensures that AGENTS.md is the preferred format but does not break existing setups that have not yet migrated.

The ~/.claude/skills/ directory is also recognized. Claude Code stored skills in ~/.claude/skills/, and OpenCode continues to look there in addition to its own ~/.config/opencode/skills/ and .opencode/skills/ locations. This means you do not need to move your existing Claude Code skills if you are using OpenCode; they will be discovered automatically. However, new skills should be placed in the OpenCode-standard locations for clarity and to ensure forward compatibility.

Environment variables can disable the Claude Code compatibility layer if you want to enforce AGENTS.md usage strictly. For example, setting OPENCODE_DISABLE_CLAUDE_FALLBACK=1 prevents OpenCode from looking for CLAUDE.md files at all. This might be useful in environments where you want to ensure only explicitly OpenCode-formatted instructions are used, or during a migration where you need to detect any remaining CLAUDE.md files. Check the OpenCode documentation for the exact environment variable names as they may evolve.

For teams migrating from Claude Code to OpenCode, the recommended path is to rename CLAUDE.md to AGENTS.md and review the content for any format adjustments. The instruction syntax is largely compatible, so renaming may be sufficient. If you were using CLAUDE.md features that differ from AGENTS.md, consult the migration guide. The glob pattern support in opencode.json's instructions field also provides an opportunity to restructure your rules into more modular files, something that was not as easily supported in Claude Code.

### 7.9 Practical Patterns and Best Practices

Based on how AGENTS.md interacts with the rest of the OpenCode ecosystem, several practical patterns have emerged that help teams make effective use of this feature.

For project AGENTS.md, include the rules and context that you want all contributors and all AI assistants to follow. This typically means: the programming languages and frameworks in use, key architectural patterns and conventions, coding standards (naming, file organization, error handling), testing requirements (test frameworks, coverage expectations, where tests live), documentation expectations (docstrings, README maintenance), and workflow conventions (branch naming, commit message format, review process). Also include any domain-specific knowledge that is not obvious from reading the code, such as business logic constraints, external system integrations, and deployment considerations. The project AGENTS.md should be committed to Git and treated as part of the project's shared knowledge base.

For global AGENTS.md, include your personal preferences that you want applied across all projects. This might include your preferred verbosity level for responses, your expectation that the AI explains its reasoning before taking action, your tool usage preferences (like "always confirm before writing files"), and any personal productivity patterns. If you work on multiple projects with different conventions, be careful not to put project-specific rules in your global AGENTS.md, as that would create conflicts when working on projects with different standards. Use global AGENTS.md for universal personal preferences, and rely on project AGENTS.md for team and project standards.

Monorepos with multiple packages benefit from a hierarchical AGENTS.md structure. Create a root AGENTS.md with overall project policies, then package-specific AGENTS.md files in each major subdirectory to add rules particular to that package. For example, a monorepo might have a root AGENTS.md stating "All packages use TypeScript and strict mode," while packages/frontend/AGENTS.md adds "Use React functional components with hooks" and packages/backend/AGENTS.md adds "All API endpoints must have OpenAPI specifications." This hierarchy allows both shared standards and package-specific customization.

Modular rule files can be managed through opencode.json's glob instructions. Instead of putting everything in one massive AGENTS.md, create a rules/ directory with files like coding-standards.md, testing.md, security.md, and then reference them with:

```json
{
  "instructions": {
    "glob": "rules/*.md"
  }
}
```

This makes the rules easier to maintain and allows different team members to own different rule documents. The instructions are concatenated at runtime, so they function as one coherent instruction set.

Lazy loading patterns for large rule sets can improve session startup performance. If your project has hundreds of rules, consider organizing them into a core set that goes in AGENTS.md and an extended set that is referenced conditionally. For example, keep only essential rules in AGENTS.md and put specialized knowledge in separate files that AGENTS.md refers to with instructions like "When working on performance, additionally consult rules/performance.md." The agent can then be instructed to read that file when needed, rather than having all rules loaded into the initial prompt. This reduces token usage and keeps the initial context more focused.

The key best practice is to treat AGENTS.md as the single source of truth for project conventions. Keep it up to date as standards evolve. Review it periodically with your team. Ensure it contains not just rules but also the rationale behind them, so the AI can make better judgments when faced with gray areas. And remember that AGENTS.md is read by both humans and AI; write it clearly and organize it well for both audiences.

### 7.10 Complete Ecosystem Diagram

The following diagram illustrates how AGENTS.md fits within the complete OpenCode ecosystem, showing its relationships to opencode.json, agent definitions, skills, tools, and permissions. This textual diagram should be read from top to bottom, with arrows indicating influence, injection, or control relationships.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         OpenCode Session                                │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │              COMPILED SYSTEM PROMPT (read-only)              │    │
│  │                                                               │    │
│  │  [Global AGENTS.md]  ← from ~/.config/opencode/AGENTS.md    │    │
│  │  [Project AGENTS.md #1] ← from project/AGENTS.md            │    │
│  │  [Project AGENTS.md #2] ← from subdir/AGENTS.md (overrides) │    │
│  │  [Agent-specific prompt] ← from agent config/prompt file    │    │
│  │                                                               │    │
│  │  All agents (primary and subagents) receive this same base  │    │
│  │  instruction set plus their own role-specific additions.    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    opencode.json (config file)                │    │
│  │                                                               │    │
│  │  {                                                            │    │
│  │    "permission": {                                           │    │
│  │      "edit": "ask",                                          │    │
│  │      "bash": "deny",                                         │    │
│  │      "skill": { "*": "allow", "internal-*": "deny" }        │    │
│  │    },                                                        │    │
│  │    "agent": {                                                │    │
│  │      "build": { "mode": "primary", ... },                   │    │
│  │      "plan": { "mode": "primary", ... },                    │    │
│  │      "code-reviewer": { "description": "...", ... }         │    │
│  │    },                                                        │    │
│  │    "instructions": { "file": "./AGENTS.md" }                │    │
│  │  }                                                            │    │
│  │                                                               │    │
│  │  Controls: permissions, agent definitions, instruction source│    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (injects into prompt)                   │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                     Agent Definitions                          │    │
│  │                                                               │    │
│  │  Primary Agents: Build, Plan, custom                         │    │
│  │  Subagents: loaded from .opencode/agents/*.md or JSON       │    │
│  │                                                               │    │
│  │  Each agent has:                                             │    │
│  │  • description (what it does)                               │    │
│  │  • mode (primary/subagent)                                  │    │
│  │  • model, temperature, steps                                │    │
│  │  • tools: which tools it can access                         │    │
│  │  • permission: tool usage rules                             │    │
│  │                                                               │    │
│  │  Agent-specific prompt is layered onto AGENTS.md content    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (can invoke)                             │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                        Skills                                 │    │
│  │                                                               │    │
│  │  Located in:                                                 │    │
│  │  • .opencode/skills/<name>/SKILL.md                         │    │
│  │  • ~/.config/opencode/skills/<name>/SKILL.md               │    │
│  │  • .claude/skills/<name>/SKILL.md (Claude compatibility)   │    │
│  │                                                               │    │
│  │  Each skill defines:                                         │    │
│  │  • name, description, license                               │    │
│  │  • what the agent should do when the skill is active        │    │
│  │                                                               │    │
│  │  Loaded on-demand via skill tool. Content injected into     │    │
│  │  conversation context when called. AGENTS.md already there. │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (constrained by)                         │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    Tools & Permissions                        │    │
│  │                                                               │    │
│  │  Tools: actions agents can perform (bash, edit, read, etc.) │    │
│  │                                                               │    │
│  │  Permissions control capability:                             │    │
│  │  • Global: opencode.json "permission"                       │    │
│  │  • Agent-level: agent.permission overrides                  │    │
│  │  • Tool-level: enable/disable via agent.tools               │    │
│  │                                                               │    │
│  │  AGENTS.md guides behavior within permitted tools.          │    │
│  │  Cannot override a "deny" permission.                        │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    Claude Code Compatibility                  │    │
│  │                                                               │    │
│  │  • CLAUDE.md files used as fallback if no AGENTS.md present │    │
│  │  • ~/.claude/skills/ also recognized                        │    │
│  │  • Can be disabled via env vars                             │    │
│  │  • Provides migration path from Claude Code                 │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│                                                                         │
│  Summary of relationships:                                             │
│                                                                         │
│  • AGENTS.md provides the foundation (the always-present rules)     │
│    and context that every agent inherits.                            │
│  • opencode.json controls the system: permissions, agents, and       │
│    can also specify instruction sources.                             │
│  • Agents define roles and have their own prompts layered on top     │
│    of AGENTS.md.                                                     │
│  • Skills add specialized knowledge when loaded, riding on top       │
│    of the AGENTS.md foundation.                                      │
│  • Tools are the actions available; permissions gate them; AGENTS.md │
│    guides how to use them wisely within those gates.                │
│  • Claude Code compatibility ensures legacy instructions still work. │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Commands - Custom Prompt Workflows

### 8.1 What Commands Are and Why They Exist

Commands are user-defined reusable prompt templates that can be invoked quickly in the OpenCode terminal interface by typing `/command-name`. Think of them as shortcuts for complex instructions that you find yourself repeating frequently. Instead of typing out a detailed request every time you want to perform a specific type of analysis or workflow, you create a command once and then invoke it with a simple slash command, optionally providing arguments.

The purpose of commands is to reduce repetitive typing and to codify best practices for recurring tasks. For example, if you frequently ask the AI to review code for security vulnerabilities, you could create a `/security-audit` command that includes your complete checklist of security concerns. When you want a security review, you simply type `/security-audit` followed by any specific files or areas you want focused on. The command expands into your full prompt template, and the AI responds with the audit.

Commands sit in the OpenCode ecosystem alongside agents, skills, and tools, but they serve a distinct role. Agents are specialized AI assistants with their own model configuration and tool permissions. Skills are reusable instruction sets loaded on demand that guide how an agent should approach a particular domain. Tools are the concrete actions that agents can perform. Commands, by contrast, are purely about prompt templating. They are a user convenience feature that lets you encapsulate complex prompts and invoke them rapidly. They do not create new agents, they do not load external knowledge files, and they do not add capabilities beyond what the current agent already has. Instead, they simply provide a shorthand for sending specific kinds of messages to the currently active agent.

When you invoke a command, the template content is processed to substitute any arguments you provide, and the resulting prompt is sent to the current agent as if you had typed it yourself. The agent then responds using its normal capabilities, tools, and permissions. This means commands are a surface-level convenience layer -- powerful for productivity, but not a fundamental extension of the OpenCode system itself.

### 8.2 Creating Commands via Markdown Files

The most common way to create a command is by writing a markdown file with frontmatter in a commands directory. OpenCode looks for commands in two primary locations: project-level commands stored in `.opencode/commands/` within your project directory, and global commands stored in `~/.config/opencode/commands/` for use across all projects.

Each command is a single markdown file. The filename (without the `.md` extension) becomes the command name. For example, if you create a file named `.opencode/commands/review-pr.md`, you can invoke it in the OpenCode terminal by typing `/review-pr`. The filename should follow the same rules as the command name itself: lowercase letters, numbers, and hyphens are fine; spaces and special characters should be avoided.

The markdown file begins with a YAML frontmatter section that defines important properties of the command. After the frontmatter, the rest of the file content becomes the prompt template that will be sent to the AI when the command is invoked.

The frontmatter can include the following fields:

- `description` (required): A short description of what the command does. This description appears in autocomplete suggestions and help listings, so make it clear and concise.
- `agent` (optional): Specifies which agent should execute this command. By default, the command runs using whatever agent is currently active. If you set this field to a specific agent name, OpenCode will switch to that agent before processing the command prompt. The agent must exist in your configuration.
- `model` (optional): Overrides the default model for the agent when this command runs. This allows you to use a different AI model for specific commands that might benefit from a more capable or more economical model.

Here is a complete example of a command file:

```markdown
---
description: Perform a thorough code review focusing on best practices
agent: code-reviewer
model: anthropic/claude-sonnet-4-20250514
---

Please review the following code changes with attention to:

1. Code quality and readability
2. Potential bugs or edge cases
3. Security vulnerabilities
4. Performance implications
5. Adherence to project conventions

Focus particularly on:
- Input validation
- Error handling
- Authentication and authorization
- Data exposure risks

Provide specific, actionable feedback. Include code snippets where improvements are needed.
```

This file would be placed at `.opencode/commands/code-review.md` and invoked as `/code-review`. When invoked, it switches to the `code-reviewer` agent (if not already active) and uses the specified model, then sends the template content as the prompt. The agent will then perform a code review according to the instructions.

You can also define commands directly in the global location `~/.config/opencode/commands/` to make them available in every project. This is useful for personal workflow commands that you use regardless of the codebase you are working in. Project-specific commands go in the project's `.opencode/commands/` directory and are committed to the repository to share with team members.

### 8.3 Creating Commands via JSON Configuration

Commands can also be defined directly in your `opencode.json` configuration file using the `command` field. This method is useful when you want to generate commands programmatically, keep everything in one configuration file, or define commands that reference external template files.

The JSON configuration for commands uses an object where each key is the command name and the value is a command definition object. The definition object supports these properties:

- `template` (required): The prompt text that will be sent to the agent. This can be a string containing the full template, or an object with a `file` key pointing to a file path, or a `url` key pointing to a remote URL. Using a file allows you to maintain large templates separately.
- `description` (required): A brief description shown in help and autocomplete.
- `agent` (optional): Name of the agent to use for this command. If not specified, the current agent is used.
- `model` (optional): Model override, same as in markdown frontmatter.
- `subtask` (optional): Boolean that, when true, forces the command to be executed as a subagent invocation rather than as a direct prompt to the current agent. This keeps the primary session's context cleaner by isolating the command's work in a separate session.

Example JSON configuration:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "command": {
    "test-runner": {
      "description": "Run tests and analyze results",
      "template": "Run the test suite and provide a summary of any failures. Include suggestions for fixing failing tests.",
      "agent": "ci-runner",
      "model": "anthropic/claude-haiku-4-20250514"
    },
    "changelog": {
      "description": "Generate a changelog from recent commits",
      "template": {
        "file": "./templates/changelog-template.md"
      },
      "subtask": true
    }
  }
}
```

The `template.file` path is relative to the location of the `opencode.json` file. You can organize your command templates in a separate directory for better project organization. This keeps your configuration concise while allowing templates to be as large and complex as needed.

Using the JSON method, you can also define commands that override built-in commands. See section 8.7 for details.

### 8.4 Prompt Configuration and Template Syntax

Command templates are dynamic prompt texts that can incorporate arguments provided by the user at invocation time, inject shell command outputs, and reference external files. Understanding these template features is key to creating powerful, reusable commands.

**Arguments**

When you invoke a command, you can provide arguments after the command name. For example: `/my-command arg1 arg2 arg3`. The template can reference these arguments using placeholder syntax.

There are three ways to reference arguments:

- `$ARGUMENTS` - This placeholder is replaced with all arguments as a single string, exactly as provided. Use this when you want to pass through a freeform string or when the exact formatting of multiple arguments matters. Example: if you type `/greeting Hello world`, and the template contains `Say: $ARGUMENTS`, the result is `Say: Hello world`.

- `$1`, `$2`, `$3` - These are positional argument placeholders. `$1` gets the first argument, `$2` the second, and so on. This is useful when you need to treat arguments separately. Example template: `Analyze file $1 and suggest improvements for $2.` Invocation: `/analyze src/index.ts performance` yields `Analyze file src/index.ts and suggest improvements for performance.`

- `$*` - This collects all arguments starting from the current position. It is not commonly used but can be helpful for variadic arguments.

Arguments are optional. If a template references an argument that was not provided, that placeholder is replaced with an empty string. If more arguments are provided than referenced, the extra arguments appear only in `$ARGUMENTS` if that is used; otherwise they are ignored.

**Shell Output Injection**

Sometimes you want to include the output of a shell command in your prompt. For example, you might want the AI to analyze the current git status or the result of a build. Command templates support shell output injection using backticks with an exclamation mark: `` `!command` ``. When the template is processed, any such pattern executes the specified shell command in a subshell and replaces the pattern with the command's standard output.

Example template:

```
Current git status:
```
!git status
```

Files that have changed:
```
!git diff --name-only HEAD~1
```

Please review these changes.
```

When this command runs, the two shell commands are executed, their outputs are captured, and they are substituted into the template before it is sent to the AI. The AI then sees the actual git status and diff output.

Be cautious with shell injection. The commands are executed using the system shell, so they can potentially be destructive. Since command files are typically created by trusted users in their own projects, this is usually not a concern, but it is something to keep in mind when sharing command files or using commands from untrusted sources.

**File References**

Templates can also include the contents of files using the `@filename` syntax. When the template is processed, `@filename` is replaced with the full contents of that file, read relative to the current working directory.

Example:

```
Here is the package.json file:
@package.json

Based on this configuration, suggest improvements to the build process.
```

The `@` syntax supports glob patterns as well. You can write `@src/**/*.ts` to include multiple files, though the behavior for multiple matches may depend on the OpenCode version (typically it concatenates them with file headers). For precise file content, specifying a single file is most reliable.

File references are powerful for making external context available to the AI without requiring the user to manually copy-paste file contents. However, they can increase token usage substantially if large files are included. Use them judiciously.

**Combining Features**

These template features can be combined in a single command. Here is a more complex example:

```
Review $1 for $2 issues. Use strict mode.

Current branch: `!git branch --show-current`
Changed files:
```
!git diff --name-only main
```

Relevant config file:
@tsconfig.json
```

This template uses a positional argument `$1` (the file to review) and `$2` (the type of issues to focus on), plus shell outputs for git context and a file reference for the TypeScript configuration.

### 8.5 Command Options

Commands have several configurable options that control their behavior. Understanding each option helps you create commands that fit your workflow precisely.

**description**

The description is the only strictly required field. It should be a short, clear phrase that tells the user what the command does. It appears in command autocomplete and in the output of a help command if one exists. Keep it under about 60 characters for best display. Examples: "Run linter and fix auto-fixable issues", "Generate documentation for a module", "Summarize recent commits".

**template**

The template is the core of the command -- it is the prompt text that will be sent to the AI after argument substitution. If defined as a string, the template is written directly in the configuration. If defined as an object with a `file` key, OpenCode reads the template content from the specified file path. The file approach is better for long or complex templates, templates that you want to version control separately, or templates that non-technical team members might edit without touching configuration files.

When using an external file, you can still include arguments, shell injections, and file references within that file. The file is simply read and its contents used as the template string.

**agent**

By default, commands execute in whatever agent is currently active. The `agent` option lets you specify a particular agent to use. When the command is invoked, OpenCode switches to that agent before sending the template prompt. This is useful for commands that require specific tool permissions or a specialized prompt. For example, a command that runs tests should probably use the `ci-runner` agent if you have one defined with appropriate bash permissions. A command that explores the codebase should use the `explore` agent.

If the specified agent does not exist, the command fails with an error. The agent name is the same as how you would reference it in other parts of the configuration: either the built-in agent names like "build" or "plan", or custom agent names defined in opencode.json or in `.opencode/agents/`.

**subtask**

The `subtask` option is a boolean (true/false). When true, the command is executed not as a direct message to the specified agent, but as a subtask invocation. That means OpenCode internally uses the `task` tool to create a subagent session, even if the specified agent is already the current agent. The subagent receives the template as its prompt and returns results to the primary session.

Why use `subtask: true`? It keeps the primary conversation context clean. The command's interaction happens in a separate session, so the primary session's message history does not get cluttered with the detailed back-and-forth of the command execution. Only the final result is brought back. This is particularly useful for commands that involve many tool calls, generate a lot of intermediate output, or are conceptually separate from the main conversation thread. Commands that are informational queries or that produce a single answer might not need subtask isolation, but commands that perform complex workflows often benefit from it.

**model**

The `model` option lets you override the default model for the agent on a per-command basis. The value is a model identifier string like "anthropic/claude-sonnet-4-20250514" or "openai/gpt-4-turbo". If not specified, the agent uses whatever model is configured for that agent, or the global default.

Use this option when a particular command requires a more capable model for complex reasoning, or when you want to use a more economical model for straightforward tasks to reduce cost or latency. For instance, a command that generates a comprehensive architecture review might explicitly use the best model available, while a simple documentation formatting command could use a faster, cheaper model.

### 8.6 Commands vs Skills vs Agents

It is important to understand how commands differ from skills and agents, as all three can feel like reusable functionality but they operate at different levels of the OpenCode system.

**Commands** are prompt templates. They are the simplest mechanism: text that gets substituted with arguments and sent to an agent. They do not define new behavior beyond what the agent already can do. They are purely a convenience for invoking an agent with a particular instruction set. Commands are invoked by the user typing `/command-name` in the terminal. They exist only as configuration; there is no runtime object or file beyond the template. Commands have no intelligence of their own -- they are static text that gets processed by the OpenCode client before being delivered to an agent.

**Agents** are specialized AI assistants. They have their own model configuration, system prompt, tool permissions, and behavioral characteristics. Agents represent a shift in the AI's identity or capabilities. When you switch agents or invoke a subagent, you are getting a different assistant with different rules about what it can do and how it thinks. Agents are defined in configuration (opencode.json) or as markdown files in agents directories. They are invoked via agent switching (tab completion), the `@agent-name` mention syntax, or the `task` tool from a primary agent. Agents are a core part of the OpenCode runtime; they manage their own sessions and have persistent identity across interactions.

**Skills** are reusable instruction sets that extend an agent's knowledge for specific domains or workflows. Skills are defined as SKILL.md files that an agent can load on demand using the `skill` tool. When a skill is loaded, its content is injected into the conversation context, providing the agent with detailed guidance about how to approach a particular task. Skills are not standalone; they require an agent to load them. They are invoked indirectly: an agent decides to load a skill based on the task, or a user can instruct an agent to use a particular skill. Skills are discoverable -- the agent sees a list of available skills through the `skill` tool description.

The relationship among these three concepts can be summarized as follows: An agent is the performer. A skill is a script the performer can consult for specific acts. A command is simply a quick way to say a particular line to the performer. If you need different capabilities, you need a different agent. If you need domain-specific knowledge, you use a skill. If you are tired of typing the same long instruction, you create a command.

That said, they do compose. A command could specify an `agent` field to run with a specific agent, effectively combining command convenience with agent specialization. A command template could instruct an agent to load a particular skill, effectively creating a shortcut that activates both a prompt and a skill. These compositions are valid and useful, but the fundamental distinction remains: commands are about prompt reuse; agents are about identity and capability; skills are about domain knowledge.

### 8.7 Built-in Commands and Overrides

OpenCode includes several built-in commands that provide core functionality. These built-in commands are hardcoded into the application and are always available unless explicitly overridden. The built-in commands typically include:

- `/init` -- Initializes a project by generating AGENTS.md and configuration based on the codebase.
- `/undo` -- Reverts the last AI action or tool call.
- `/redo` -- Reapplies an undone action.
- `/share` -- Shares the current session or exports content.
- `/help` -- Shows help information about commands and usage.

These commands serve essential workflow functions. However, you can override any built-in command by defining a custom command with the same name. This is done either by creating a markdown file with that name in `.opencode/commands/` or by defining it in the `command` section of `opencode.json`. When a command name matches both a built-in and a custom definition, the custom definition takes precedence.

Overriding built-in commands should be done carefully. If you override `/undo` with a custom template, you are replacing its functionality entirely. The new command will replace the default behavior and may confuse users who expect the standard behavior. Overriding is most appropriate when you want to extend or modify a built-in command's behavior, but you should ensure your custom command still performs the essential function users expect, or you should choose a different name to avoid confusion.

In practice, overriding built-in commands is rarely needed. The built-in commands are foundational. More common is creating new commands with unique names that suit your project's workflows. If you do override, document it clearly for your team.

### 8.8 Practical Command Patterns

Commands shine for recurring workflows that involve a structured prompt. Here are several practical patterns that demonstrate how commands can encapsulate useful procedures.

**Test Runner**

Create a command that runs your test suite and interprets the results:

```markdown
---
description: Run full test suite and analyze failures
agent: ci-runner
subtask: true
---

Run all tests using the project's test script.

If tests fail:
1. Capture the failure output
2. Identify the failing test files
3. Suggest possible causes based on the error messages
4. Recommend specific fixes when obvious

Provide a summary:
- Total tests run
- Pass/fail count
- Any flaky tests noted
- Next steps for resolving failures
```

Invocation: `/run-tests` (or `/run-tests --watch` if you add an argument to pass flags).

**Code Reviewer**

A command for comprehensive code review:

```markdown
---
description: Detailed code review against best practices
agent: code-reviewer
model: anthropic/claude-sonnet-4-20250514
---

Perform a thorough code review of the changes in `!git diff HEAD~1`.

Check for:
- Logic errors and edge cases
- Security vulnerabilities (OWASP Top 10)
- Performance issues
- Code readability and maintainability
- Adherence to project style guide
- Proper error handling
- Test coverage for changes

For each issue found:
- Quote the problematic code
- Explain why it is problematic
- Provide a corrected version

At the end, give an overall assessment and prioritize the issues by severity.
```

This command uses shell output injection to get the actual diff, making it easy to review the most recent changes.

**PR Description Generator**

When creating pull requests, you often need to write a description that summarizes the changes. A command can automate this by examining the commits and generating a structured PR description:

```markdown
---
description: Generate PR description from commits
subtask: true
---

Generate a pull request description based on the commits that are about to be pushed.

First, run: `!git log --oneline @{u}..HEAD` to see commits not yet pushed.

Structure the PR description as:

## Summary
One or two sentences describing the overall change.

## Changes
- Bullet list of key changes (one bullet per significant commit, expanded with explanation)

## Testing
- How was this tested? (manual, automated, etc.)

## Screenshots
- If there are UI changes, note that screenshots should be added

## Additional Notes
- Any other information reviewers should know

Keep the tone professional and concise. Assume the reviewer is familiar with the codebase.
```

This command can be invoked after you have made commits and are ready to push, helping you produce a consistent PR template.

**Changelog Generator**

Similar to the PR generator, a changelog command can produce release notes from git tags:

```markdown
---
description: Create changelog for a release
template:
  file: ./templates/changelog-template.md
subtask: true
---

Using conventional commits, generate a changelog for the version specified in argument $1.

Run these commands first:
```
!git tag -l | sort -V
```
```
!git log --pretty=format:'%s' $1..HEAD
```

Categorize commits by type: feat, fix, breaking change, docs, etc.
Follow the project's changelog format (see @CHANGELOG.md for examples).
Include PR references if present in commit messages.

Output should be ready to copy-paste into the CHANGELOG.md file.
```

**Documentation Reviewer**

A command to check documentation quality:

```markdown
---
description: Review documentation for clarity and completeness
---

Analyze the provided documentation file ($1) or the file at cursor.

Check for:
- Clear, descriptive headings
- Complete examples with expected output
- Explanation of concepts before diving into details
- Links to related resources
- Absence of broken references
- Appropriate level of detail for intended audience

Provide specific suggestions for improvement, including rewritten sections where needed.
```

These patterns illustrate how commands can automate repetitive prompt crafting, incorporate dynamic context via shell and file references, and delegate to appropriate agents. The key is identifying tasks that you perform repeatedly and that follow a consistent structure.

### 8.9 Commands Ecosystem Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         OpenCode Session                                │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │              COMPILED SYSTEM PROMPT (read-only)              │    │
│  │                                                               │    │
│  │  [Global AGENTS.md]  ← from ~/.config/opencode/AGENTS.md    │    │
│  │  [Project AGENTS.md] ← from project AGENTS.md files         │    │
│  │  [Agent-specific prompt]                                      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (injected into)                            │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    Commands System                            │    │
│  │                                                               │    │
│  │  Command definitions live in:                                │    │
│  │  • .opencode/commands/*.md (project)                        │    │
│  │  • ~/.config/opencode/commands/*.md (global)                │    │
│  │  • opencode.json "command" field                            │    │
│  │                                                               │    │
│  │  When user types /command-name:                              │    │
│  │  1. Command file is located                                  │    │
│  │  2. Frontmatter read (description, agent, model)            │    │
│  │  3. Template content is processed:                           │    │
│  │     - Arguments substituted ($1, $ARGUMENTS, etc.)           │    │
│  │     - Shell outputs injected (`!cmd`)                        │    │
│  │     - File contents injected (@file)                         │    │
│  │  4. If agent specified, switch to that agent                 │    │
│  │  5. Processed prompt is sent to the agent                    │    │
│  │                                                               │    │
│  │  Result: Agent receives a normal user message containing     │    │
│  │  the expanded template, and responds accordingly.            │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    Agents (active layer)                      │    │
│  │                                                               │    │
│  │  The agent (primary or specified) receives the command's     │    │
│  │  expanded prompt as if typed by the user. Agent then uses    │    │
│  │  its normal tools, skills, and reasoning to generate a       │    │
│  │  response.                                                    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  Key relationships:                                                   │
│  • Commands are purely a prompt templating layer                   │    │
│  • They do not alter agent capabilities or permissions            │    │
│  • They can optionally specify which agent to use and which model │    │
│  • They are user-facing shortcuts, not runtime components         │    │
│  • Can override built-in commands like /init, /undo, /help        │    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---


## 9. Custom Tools - Extending Agent Capabilities

### 9.1 What Custom Tools Are and Their Role

Custom tools are functions that you, as a user or project maintainer, can create to extend the capabilities of OpenCode agents beyond the built-in tool set. While OpenCode provides essential tools like `bash`, `read`, `write`, `edit`, `grep`, and others, these may not cover every specialized need your workflow might have. Custom tools allow you to define arbitrary functions that the AI can invoke during a conversation, enabling integration with databases, APIs, custom scripts, or any other system you can programmatically access.

The role of custom tools is to bridge the gap between the AI assistant and your specific environment. Suppose your project uses a proprietary deployment system, a custom monitoring dashboard, a specialized code generation tool, or an internal documentation database. With custom tools, you can write a function that exposes that system to the AI. The AI can then call this tool just like it calls built-in tools, receiving results and incorporating them into its reasoning and actions.

Custom tools work alongside built-in tools and also alongside MCP (Model Context Protocol) servers. The distinction is mostly about implementation style and integration depth. Built-in tools are first-class citizens of OpenCode, implemented by the core team and always available. MCP servers are external processes that provide tools over a standardized protocol; they allow third-party integrations without modifying OpenCode itself. Custom tools, on the other hand, are written by you, placed in your project's `.opencode/tools/` directory (or global equivalent), and are loaded directly by OpenCode at startup. They are simpler to create than full MCP servers and do not require running a separate server process, yet they give you tremendous flexibility.

In terms of capability, custom tools can do anything a program can do: query a database, call an HTTP API, run a script in any programming language, transform data, validate patterns, you name it. The tool definition itself is written in TypeScript or JavaScript and uses a helper function called `tool()` from the `@opencode-ai/plugin` package. But the actual execution logic inside the tool can invoke external programs or scripts written in Python, Ruby, Bash, Go, or any language that can be run from the command line. This wrapper pattern means you are not limited to JavaScript for your tool implementation; you just need a small JavaScript shim that defines the interface and delegates to your actual logic.

Custom tools become available to agents according to the tool permissions system. If you define a custom tool named `database`, you can then control its usage via the `permission` setting in `opencode.json` or in agent configuration, just like built-in tools. The tool's name appears in the agent's tool list, and the agent can call it when permitted.

### 9.2 Tool File Structure and the tool() Helper

Each custom tool is defined in a single file, either a `.ts` (TypeScript) or `.js` (JavaScript) file. These files live in the `.opencode/tools/` directory of your project for project-specific tools, or in `~/.config/opencode/tools/` for global tools available in all projects. The filename determines the tool's name. If you create a file called `database.ts`, the tool will be named `database` and can be invoked by the AI as `database(...)`. If you create `my-utils.js`, the tool name becomes `my-utils`.

The tool file must export a function created by the `tool()` helper. The `tool()` function is imported from the `@opencode-ai/plugin` package. It takes an options object and returns a function that conforms to the OpenCode tool interface. The options object includes at minimum a `schema` that defines the tool's arguments (their types, required/optional status, and descriptions) and a `description` that explains what the tool does. Optionally, it can also specify a `context` boolean to receive the session context object (more on that in section 9.5).

Here is a minimal tool definition:

```typescript
import { tool } from '@opencode-ai/plugin';

export const database = tool({
  description: 'Query the project database for information',
  schema: {
    query: {
      type: 'string',
      description: 'SQL query to execute',
      required: true
    }
  },
  execute: async (args, context) => {
    // Implementation goes here
    const result = await runDatabaseQuery(args.query);
    return { result };
  }
});
```

The `schema` section uses Zod under the hood to define and validate arguments. OpenCode provides a simplified type syntax that maps to Zod types: `string`, `number`, `boolean`, `array`, `object`, and also Zod's refinements like `optional`, `default`, etc. The actual implementation of argument validation is handled by the framework based on this schema.

The `execute` function receives two parameters: `args` (an object containing the arguments provided by the AI, already validated against the schema) and `context` (a session context object that gives information about the current session, if you requested it by setting `context: true` in the options). The `execute` function can be asynchronous (returning a Promise) or synchronous. It should return a value that will be serialized and sent back to the AI as the tool result. Typically this is a plain object; complex objects are JSON-serialized.

The `tool()` helper registers this function as a tool with OpenCode. By default, it uses the exported variable name (`database` in this example) as the tool name. You can override the tool name by passing a first argument to `tool()` that is a string with the desired name, but that is rarely necessary.

It is important to understand that the tool file is loaded once at OpenCode startup (or when the file changes, depending on configuration). The tool definition is then available for the duration of the session. If you edit a tool file, you typically need to restart OpenCode to see the changes, though some implementations may support hot-reloading during development.

### 9.3 Single Tool vs Multiple Tools Per File

A tool file can export a single tool (the default export shown above), or it can export multiple tools. When a file exports multiple functions using the `tool()` helper, each export becomes a separate tool, and the tool name is constructed as `<filename>_<exportname>`. This naming convention allows you to group related tools in a single file while keeping their names distinct.

For example, consider a file named `git-utils.ts` that contains:

```typescript
import { tool } from '@opencode-ai/plugin';

export const commit = tool({
  description: 'Create a git commit with message',
  schema: {
    message: { type: 'string', required: true },
    all: { type: 'boolean', default: false }
  },
  execute: async (args) => {
    // run git commit command
    return { success: true };
  }
});

export const branch = tool({
  description: 'Create a new git branch',
  schema: {
    name: { type: 'string', required: true },
    startPoint: { type: 'string', optional: true }
  },
  execute: async (args) => {
    // run git branch command
    return { success: true };
  }
});

export const merge = tool({
  description: 'Merge a branch into current branch',
  schema: {
    branch: { type: 'string', required: true },
    squash: { type: 'boolean', default: false }
  },
  execute: async (args) => {
    // run git merge command
    return { success: true };
  }
});
```

This file defines three tools: `git-utils_commit`, `git-utils_branch`, and `git-utils_merge`. The AI can invoke these as separate tools. The grouping is logical: all git-related utilities live in the same file, but they are exposed as individual tools with clear names.

If you prefer single, top-level tool names without the filename prefix, you can instead create separate files: `commit.ts`, `branch.ts`, `merge.ts` each in the `.opencode/tools/` directory. Then the tool names would simply be `commit`, `branch`, `merge`. Whether to group or separate is a matter of organization. Grouping keeps related code together and reduces file clutter; separating gives shorter tool names and clearer separation.

There is also a convention where the default export ( `export default`) becomes a tool with a name matching the filename. But the tool() helper pattern typically uses named exports. The exact mapping can depend on the OpenCode plugin system's implementation. The documentation suggests that each export using `tool()` becomes a tool with the name `filename_exportname`. So it is best to use named exports and adopt that naming scheme.

### 9.4 Tool Arguments and Schema Validation

Tools accept arguments from the AI. The schema defines what arguments are expected, their types, whether they are required or optional, and descriptions that help the AI understand how to use the tool correctly.

The schema is defined inside the `tool()` options using a simplified syntax that compiles to Zod. The available types mirror common data types:

- `string` -- Text
- `number` -- Numeric value (integer or float)
- `boolean` -- true or false
- `array` -- List of items; you provide an `items` field describing the element type
- `object` -- Nested object; you provide a `schema` field with nested properties

Additionally, you can mark a field as `optional: true` to make it not required. You can provide a `default` value that will be used if the argument is omitted (optional fields can have defaults).

You can also attach a `description` to each field to explain its purpose to the AI. The AI sees these descriptions when deciding how to call the tool, so clear descriptions are crucial for proper usage.

Example with full features:

```typescript
export const search = tool({
  description: 'Search the codebase with flexible criteria',
  schema: {
    pattern: {
      type: 'string',
      description: 'Search pattern (regex)',
      required: true
    },
    path: {
      type: 'string',
      description: 'Directory to search in (default: current)',
      optional: true,
      default: '.'
    },
    caseSensitive: {
      type: 'boolean',
      description: 'Whether to respect case',
      optional: true,
      default: false
    },
    maxResults: {
      type: 'number',
      description: 'Maximum number of results to return',
      optional: true,
      default: 100
    }
  },
  execute: async (args) => {
    // args.pattern is guaranteed to be a string
    // args.path is string, defaults to '.'
    // args.caseSensitive is boolean, defaults to false
    // args.maxResults is number, defaults to 100
    const results = performSearch(args.pattern, args.path, args.caseSensitive, args.maxResults);
    return { results };
  }
});
```

When the AI calls this tool, it will provide a JSON object with the arguments. The OpenCode runtime validates the arguments against the schema before passing them to `execute`. If required fields are missing or types do not match, the AI receives an error and may retry. This validation protects your tool from malformed input.

You can also import Zod directly and define the schema using full Zod syntax for more complex scenarios. The `tool()` helper accepts either the simplified object schema or a raw Zod schema. This allows you to use Zod's powerful features like refinements, transforms, unions, and discriminated unions. For instance, if your tool can accept either a file path or a URL as input, you could define a union type in Zod.

By carefully designing your schema, you can create tools that are easy for the AI to use correctly. Good schema design includes clear descriptions, sensible defaults, and appropriate types. Avoid overly generic types like `any` or overly permissive schemas; constrain the input to what your tool actually expects.

### 9.5 Tool Context and Session Awareness

When a tool executes, it often needs to know about the current session -- things like which agent is running, what the session ID is, the current working directory, and other contextual information. OpenCode provides this through a `context` object that is passed as the second argument to the `execute` function, but only if you set `context: true` in the tool options.

The context object typically contains these fields:

- `agent` -- The name of the agent that invoked the tool. Useful for customizing behavior based on agent permissions or specialization.
- `sessionId` -- A unique identifier for the current OpenCode session. You can use this for logging, tracking, or associating tool results with a particular conversation.
- `messageId` -- The identifier of the current message that triggered the tool call. Helps with correlating logs.
- `directory` -- The current working directory where the OpenCode session was started, or the active project root. This is the base path for relative file operations.
- `worktree` -- The git worktree root, if the project is inside a Git repository. This may be the same as directory or a parent directory.
- Additional fields might include configuration, environment, or user preferences depending on the OpenCode version.

To receive this context, declare it in the tool definition:

```typescript
export const mytool = tool({
  description: 'Does something with context',
  schema: { /* ... */ },
  context: true,  // <-- this enables context argument
  execute: async (args, context) => {
    console.log(`Agent ${context.agent} called me from session ${context.sessionId}`);
    console.log(`Working directory: ${context.directory}`);
    // Use context.directory to resolve file paths relative to the project
    return { success: true };
  }
});
```

If you do not need context, omit the `context` option and your execute function will take only one argument (the args). Keeping context disabled when unused is marginally more efficient and safer (less information leakage).

Session awareness enables tools to be adaptive. For example, a logging tool could prefix all log entries with the sessionId, allowing you to trace back which session produced which logs. A tool that interacts with a database might use the agent name to apply different access policies. A tool that reads or writes files can use `context.directory` to know where to operate without requiring the user to pass the path every time.

You should design your tools to be resilient to the context they receive. The directory might change if the user runs OpenCode from a different location. The agent name might be a custom agent you did not anticipate. Validate assumptions and provide fallbacks.

### 9.6 Name Collisions and Precedence

When you define a custom tool, its name is derived from the filename (and export name). If that name matches the name of a built-in tool, your custom tool takes precedence. This means the AI will call your custom function instead of the built-in one when it invokes that tool name. This is intentional: it allows you to override or extend built-in behavior.

For example, if you create a file `read.ts` that exports a custom `read` tool, when the AI tries to read a file, it will call your function instead of OpenCode's built-in `read` tool. This could be used to add logging, caching, encryption, or custom error handling around file reads. However, it also means you could accidentally break functionality if your custom `read` does not replicate the expected behavior. Built-in tools have a specific contract: their inputs and outputs follow certain conventions that other parts of the system might rely on. Overriding them should be done with caution and only when you understand the implications.

Name collisions can also occur between two custom tools if you have multiple files that generate the same tool name. For instance, if you have both `git.ts` with an export `status` and `status.ts` with a default export, you might end up with `git_status` and `status` as distinct tools. That is fine. But if you have `utils.ts` exporting `read` and also a separate `read.ts` exporting `read`, you would have `utils_read` and `read`. If `utils_read` is the one you intended, fine; but if both define `read` at the top level, OpenCode may have to resolve which one wins. The exact collision resolution depends on the tool loading order. Generally, later-loaded tools may override earlier ones if they produce the same name. To avoid ambiguity, choose unique names for your tools. A common pattern is to prefix tools with the project or domain: `mydb_query`, `mydb_schema`, rather than bare `query` and `schema`.

For clarity and to avoid accidental overrides, it is recommended to name your tools with a distinctive prefix, especially if the tool provides specialized functionality. Use hyphens to separate words. The tool name is what appears in agent prompts and logs, so make it descriptive yet concise.

If you do intentionally want to override a built-in tool, make sure your custom implementation fully adheres to the built-in tool's interface (argument names, types, and return format) to maintain compatibility. Test thoroughly.

### 9.7 Tools in Any Language via Wrapper Pattern

While tool definitions are written in TypeScript/JavaScript as required by the `@opencode-ai/plugin`, the actual work done by the tool can be performed by programs in any language. This is achieved through a wrapper pattern: the JavaScript execute function acts as a thin shim that invokes an external script or command, passing arguments and returning results.

A common approach is to have your tool's `execute` function use the `Bun.$` utility (or Node's `child_process`) to run a shell command that executes your script in whatever language you prefer. The Bun runtime is often available in OpenCode environments because OpenCode itself may use Bun. The `Bun.$` function allows running a command and capturing its output.

For example, suppose you have a Python script `scripts/analyze.py` that performs static analysis. You can create a JavaScript wrapper tool:

```typescript
import { tool } from '@opencode-ai/plugin';

export const analyze = tool({
  description: 'Run static analysis on code',
  schema: {
    path: { type: 'string', description: 'Path to analyze', required: true }
  },
  execute: async (args) => {
    // Use Bun.$ to run the Python script
    const result = Bun.$`python scripts/analyze.py ${args.path}`;
    // The output (stdout) is captured as a string
    return { analysis: result };
  }
});
```

This pattern lets you write the core logic in Python, which might have better libraries for analysis, while the tool wrapper remains minimal TypeScript for OpenCode integration. You could just as well use a Bash script, a Ruby program, a Go binary, or any executable. Just make sure the script is available in the environment where OpenCode runs, and that you handle errors and timeouts appropriately.

If you need to pass complex data structures to the external script, you can pass them as JSON on the command line or via stdin. Similarly, the script can output JSON which the wrapper then parses and returns to OpenCode. This keeps the interface between the tool wrapper and the external logic well-defined.

This wrapper approach maximizes flexibility: you are not forced to implement your tool in JavaScript. You can leverage existing codebases, specialized libraries, or language-specific tooling that you already have. The only requirement is that you can invoke it from a shell and exchange data via standard output or files.

### 9.8 Custom Tools vs MCP Servers vs Built-in Tools

OpenCode offers three primary ways to extend its capabilities: built-in tools, custom tools, and MCP servers. Understanding the trade-offs helps you choose the right approach for your needs.

**Built-in Tools** are part of the OpenCode core. They are written, maintained, and shipped with OpenCode itself. They provide fundamental operations like file system access, shell execution, web fetching, and task orchestration. You cannot modify built-in tools (except by overriding via custom tools of the same name, if really needed). The advantage of built-in tools is reliability, consistency, and full integration with OpenCode's permission and session management. They are always available and well-documented.

**Custom Tools** are user-defined extensions that you add to your project or global configuration. They are loaded directly by OpenCode at runtime. Custom tools are ideal for project-specific functionality, for wrapping existing scripts, and for integrations where you control the code. They are easy to develop, just write a TypeScript file, no server process needed. They run within the same runtime as OpenCode (or in a subprocess if you launch external commands). Custom tools have direct access to the session context if needed, and they participate in the permission system via their name. However, they are limited to the languages and runtimes available in your OpenCode environment; they depend on the `@opencode-ai/plugin` package; and they require a restart to reload changes (unless you use a watcher). Custom tools are best for functionality that is specific to your project or team and that you want to version control alongside the codebase.

**MCP Servers** (Model Context Protocol) are external processes that communicate with OpenCode via a standardized protocol. An MCP server can provide tools, resources, and prompts. OpenCode connects to MCP servers that are running separately and discovers their capabilities. The advantage of MCP is that it enables third-party integrations without requiring the integration code to be part of OpenCode itself. MCP servers can be written in any language that supports the protocol, and they run independently. They can be shared across different AI applications that support MCP, not just OpenCode. They can also be more isolated from OpenCode's process, which can improve stability and security. However, MCP servers require running a separate process, managing their lifecycle, and handling the protocol implementation. They are more heavyweight than custom tools for simple integrations.

In summary:

- Use built-in tools for general file and shell operations -- they are already there.
- Use custom tools for project-specific or simple external integrations that you want to keep in the project repository and version control. Custom tools are quick to write and integrate tightly with OpenCode's context.
- Use MCP servers for more complex or standalone integrations, especially if you want to share the integration across multiple AI applications or if you need the integration to run as a separate service with its own lifecycle.

Many teams will use all three: built-in for core operations, custom tools for project-specific helpers, and MCP for broader ecosystem integrations like database browsers or API explorers.

### 9.9 Practical Custom Tool Examples

**Database Query Tool**

A tool that allows the AI to run SQL queries against your project's database, with safety checks:

```typescript
import { tool } from '@opencode-ai/plugin';
import { exec } from 'child_process';
import { promisify } from 'util';
const execAsync = promisify(exec);

export const query = tool({
  description: 'Execute a read-only SQL query on the project database',
  schema: {
    sql: {
      type: 'string',
      description: 'SQL SELECT statement to execute',
      required: true
    },
    format: {
      type: 'string',
      description: 'Output format: table, json, or csv',
      optional: true,
      default: 'table'
    }
  },
  context: true,
  execute: async (args, context) => {
    // Safety: ensure only SELECT
    if (!args.sql.trim().toLowerCase().startsWith('select')) {
      throw new Error('Only SELECT queries are allowed');
    }

    // Use context.directory to find .env file perhaps
    const { stdout } = await execAsync(`psql -h localhost -U user dbname -c "${args.sql}"`, {
      cwd: context.directory,
      env: { ...process.env }
    });

    // Format based on requested format (simplified)
    let result = stdout;
    if (args.format === 'json') {
      // convert to JSON using some parser
      result = convertToJSON(stdout);
    }

    return { rows: result };
  }
});
```

**Code Formatter Tool**

A tool that runs a formatter on specified files:

```typescript
import { tool } from '@opencode-ai/plugin';
import { Bun } from 'bun';

export const format = tool({
  description: 'Format code files using the project formatter',
  schema: {
    files: {
      type: 'array',
      items: { type: 'string' },
      description: 'List of files to format',
      required: true
    },
    dryRun: {
      type: 'boolean',
      description: 'Check without modifying',
      optional: true,
      default: false
    }
  },
  execute: async (args) => {
    const files = args.files.join(' ');
    const flag = args.dryRun ? '--check' : '--write';
    const cmd = Bun.$`prettier ${flag} ${files}`;
    // Wait for command to complete
    const output = await cmd;
    return { formatted: !args.dryRun, output };
  }
});
```

**Deployment Tool**

A tool that triggers a deployment to a staging environment:

```typescript
import { tool } from '@opencode-ai/plugin';
import { fetch } from 'undici';

export const deploy = tool({
  description: 'Trigger a deployment to the staging environment',
  schema: {
    branch: {
      type: 'string',
      description: 'Git branch to deploy',
      required: true
    },
    wait: {
      type: 'boolean',
      description: 'Wait for deployment to complete',
      optional: true,
      default: false
    }
  },
  context: true,
  execute: async (args, context) => {
    // Call deployment API
    const response = await fetch('https://deploy.example.com/api/trigger', {
      method: 'POST',
      body: JSON.stringify({
        repository: context.repository,
        branch: args.branch,
        sessionId: context.sessionId
      }),
      headers: { 'Content-Type': 'application/json' }
    });

    if (!response.ok) {
      throw new Error(`Deployment failed: ${response.statusText}`);
    }

    const data = await response.json();
    return { deploymentId: data.id, status: data.status };
  }
});
```

**Notification Tool**

A tool that sends a notification to a chat channel when something important happens:

```typescript
import { tool } from '@opencode-ai/plugin';
import { webhook } from './webhook-utils'; // custom helper

export const notify = tool({
  description: 'Send a notification to the team chat',
  schema: {
    message: {
      type: 'string',
      description: 'Notification message',
      required: true
    },
    level: {
      type: 'string',
      description: 'Severity: info, warning, error',
      optional: true,
      default: 'info'
    }
  },
  execute: async (args) => {
    const emoji = args.level === 'error' ? '🚨' : args.level === 'warning' ? '⚠️' : 'ℹ️';
    await webhook.send('https://chat.example.com/hooks/opencode', {
      text: `${emoji} *OpenCode* ${args.message}`
    });
    return { sent: true };
  }
});
```

These examples show how custom tools can encapsulate a wide range of functionality while providing a clean, typed interface to the AI. The AI does not need to know how the tool works internally; it just needs to understand what arguments to provide based on the schema description.

### 9.10 Custom Tools Ecosystem Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         OpenCode Session                                │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │              COMPILED SYSTEM PROMPT (read-only)              │    │
│  │                                                               │    │
│  │  [Global AGENTS.md]  ← from ~/.config/opencode/AGENTS.md    │    │
│  │  [Project AGENTS.md] ← from project AGENTS.md files         │    │
│  │  [Agent-specific prompt]                                      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (injected into)                            │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                 Custom Tools Loader                            │    │
│  │                                                               │    │
│  │  At startup, OpenCode scans:                                 │    │
│  │  • .opencode/tools/*.ts, *.js (project)                     │    │
│  │  • ~/.config/opencode/tools/*.ts, *.js (global)             │    │
│  │                                                               │    │
│  │  Each file is loaded and evaluated:                          │    │
│  │  1. Import tool() from @opencode-ai/plugin                  │    │
│  │  2. Execute file, collecting exports                        │    │
│  │  3. Each export using tool() becomes an available tool      │    │
│  │  4. Tool name = filename[_exportname]                       │    │
│  │  5. Tool schema registered for AI argument validation       │    │
│  │                                                               │    │
│  │  Custom tools are merged with built-in tools. If a custom   │    │
│  │  tool name matches a built-in tool, the custom tool takes   │    │
│  │  precedence (override).                                      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                           ↕ (available to)                             │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    Agents (with Tools)                         │    │
│  │                                                               │    │
│  │  Each agent has a set of tools it can access (configured    │    │
│  │  via agent.tools). When the agent's permissions allow a     │    │
│  │  custom tool (permission.<toolname>), the AI can invoke     │    │
│  │  that tool during conversation.                              │    │
│  │                                                               │    │
│  │  Tool call flow:                                             │    │
│  │  1. AI decides to call tool with arguments                  │    │
│  │  2. OpenCode validates arguments against tool.schema        │    │
│  │  3. If context: true, session context is provided           │    │
│  │  4. tool.execute(args, context) is called                   │    │
│  │  5. Return value is serialized and sent back to AI          │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  Key relationships:                                                   │    │
│  • Custom tools extend the agent's action space beyond built-ins   │    │
│  • Tools participate in the permission system (allow/ask/deny)     │    │
│  • Tools can receive context about the session (agent, directory)  │    │
│  • Tool definitions are loaded once at startup; editing requires   │    │
│    restart to see changes (unless hot-reload enabled)              │    │
│  • Tools can invoke external programs in any language via wrapper   │    │
│    pattern, enabling polyglot tool implementations                │    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---


## 10. Summary (Revised)

| Concept | What it is | Key file | Tool used |
|---------|-----------|----------|-----------|
| **Tool** | An action the LLM can perform | `opencode.json` (permissions) | N/A |
| **Agent** | A specialized AI assistant with its own config | `opencode.json` or `.md` in agents dir | `task` (to invoke subagents) |
| **Skill** | Reusable instructions loaded on-demand | `SKILL.md` in skills dirs | `skill` |
| **Command** | Reusable prompt template with arguments | `.opencode/commands/*.md` or `opencode.json` `command` field | `/command-name` in TUI |
| **Custom Tool** | User-defined function extending capabilities | `.opencode/tools/*.ts` or `.js` | Tool name (e.g., `database`) as defined |

The ecosystem composes as follows:

- **Agents** define *who* is working (prompt, model, personality, tool permissions).
- **Tools** define *what* they can do (capabilities, actions). Built-in tools provide the core set; custom tools extend it; MCP servers integrate external services.
- **Commands** provide a shorthand for common prompts, optionally specifying an agent and model. They are user-facing shortcuts that expand to full prompts.
- **Skills** define *how* agents should approach specific tasks (knowledge, procedures). They are loaded on demand and inject instructions into the conversation.
- **Permissions** control *which* tools and skills are accessible at each level.
- **AGENTS.md** provides the foundational project rules and context that all agents inherit automatically.

These layers work together to create a powerful, flexible AI assistant platform. Tools and skills are discovered and invoked by agents based on the needs of the task. Commands are invoked directly by the user to bootstrap common prompts. Agents make decisions about which tools to use and when to load skills, while respecting permission constraints. The configuration files (`opencode.json`, `.opencode/agents/*.md`, `.opencode/skills/*/SKILL.md`, `.opencode/commands/*.md`, `.opencode/tools/*.ts`) allow you to customize every aspect of this system to match your project's requirements and your personal workflow preferences.
