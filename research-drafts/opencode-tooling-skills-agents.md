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

AGENTS.md sits beneath both of these layers. It is not an agent itself; it does not have a personality or specific role. It is not a skill that gets invoked only for particular tasks. Instead, AGENTS.md represents the baseline rules and context that persist throughout the entire session, regardless of which agent is active. Every agent—whether the primary build agent, the plan agent, or any custom subagent—receives the AGENTS.md content as part of its system instructions. This makes AGENTS.md the authoritative source for project conventions, coding standards, architectural decisions, and workflow preferences that all team members and all AI assistants must adhere to.

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

AGENTS.md can contain instructions about how tools should be used, but it cannot change the underlying permission system. This distinction is crucial: permissions are controlled by opencode.json or agent configuration and represent hard constraints—what tools are allowed or denied—whereas AGENTS.md represents soft guidance about how to use tools effectively and responsibly.

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

Manual creation is straightforward: create a file named AGENTS.md at the appropriate location and write your instructions in markdown format. The file can contain any content you would put in a system prompt—rules, examples, workflows, architectural notes, coding standards. There is no required format beyond being valid markdown. However, organizing the content with clear headings makes it easier to maintain and for the AI to parse. Example structure might include sections like Project Overview, Coding Standards, Testing Requirements, Git Workflow, and Common Pitfalls.

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
│  • AGENTS.md provides the foundation—the always-present rules        │
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
