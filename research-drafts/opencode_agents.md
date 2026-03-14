# OpenCode Agents: Complete Reference Guide

## Introduction

Imagine having a team of specialized assistants, each with their own skills and personality, ready to help you with different kinds of tasks. That's what OpenCode agents are all about. They're like customizable AI helpers that you can configure to handle specific types of work—whether you need someone to build code, review it for problems, explore your project files, or plan out complex features.

In OpenCode, agents are the way you interact with artificial intelligence. Instead of having one generic AI that tries to do everything, agents let you create focused specialists. Think of it like having a carpenter, an electrician, and a plumber in your toolbox—each has their own tools and expertise, and you call on the right one for each job.

This guide will walk you through everything you need to know about OpenCode agents, from the built-in ones that come with the system to creating your own custom agents perfectly suited to your workflow.

---

## Understanding Agent Types

OpenCode distinguishes between two fundamental types of agents: **primary agents** and **subagents**. Understanding this distinction is key to using OpenCode effectively.

### Primary Agents

Primary agents are your main companions during a coding session. They're the ones you talk to directly, the ones that respond to your questions, and the ones that carry out your requests. When you start a new OpenCode session, you're automatically paired with a primary agent.

Primary agents have full access to their configured tools and can work independently. They're like your main assistant who handles the bulk of the work. OpenCode comes with two built-in primary agents:

- **Build**: The default agent, equipped with all tools and ready for any development task
- **Plan**: A cautious agent designed for analysis without making changes

You can switch between primary agents during a session using the **Tab** key (or your configured `switch_agent` keybind). This cycles through all primary agents that aren't disabled or hidden. Each time you press Tab, you switch to the next agent, and the interface updates to show which agent is currently active.

When you switch agents, the conversation context remains the same—you're not starting over. You're simply changing who you're talking to. This means if Build has been working on a feature and you switch to Plan, Plan can see everything that was discussed and built so far, and can offer analysis or suggestions based on that context.

The keyboard shortcut for switching agents is configurable through the `switch_agent` keybind in your TUI configuration. If you prefer a different key or key combination, you can customize it.

### Subagents

Subagents are specialists that get called in for specific tasks. They don't operate as your main conversational partner, but they can be summoned when needed. There are two ways subagents get involved:

1. **Automatically**: Primary agents can recognize when a task would be better handled by a subagent and invoke them automatically based on their descriptions. If you ask Build to "find all the places where we handle user authentication," Build might automatically invoke the Explore subagent to search the codebase because that's Explore's specialty.

This happens seamlessly—you might see a message indicating that a subagent has been called in, and when it finishes, the primary agent will return with the results.

2. **Manually**: You can explicitly call any subagent by typing `@` followed by the agent's name in your message. For example: `@explore find all files related to authentication`. This is like saying, "Hey, I need your expertise on this specific part."

Subagents are designed to be focused and efficient. They receive a clear task, complete it, and return their results to the primary agent that called them. Some subagents, like Explore, are intentionally limited—Explore can't modify files because its job is to investigate, not change things.

### Hidden System Agents

Beyond the primary and subagent categories, OpenCode has three hidden system agents that work behind the scenes:

- **Compaction**: Automatically manages conversation context when it gets too long
- **Title**: Generates concise session titles
- **Summary**: Creates session summaries for later reference

These agents run automatically when needed and don't appear in your agent selection. They're like back-office staff—essential but invisible.

---

## Built-in Agents

OpenCode ships with seven built-in agents, each serving a specific purpose. Let's explore each one in detail.

### Build Agent

**Mode**: Primary
**Default**: Yes

The Build agent is your all-purpose development companion. It's the default agent you start with because it has access to all tools—file editing, bash commands, web fetching, everything. Think of Build as your general contractor who can handle any aspect of the project.

When you're writing code, debugging issues, making architectural changes, or doing routine development work, Build is the agent you want. It can read files, make edits, run commands, search for patterns, and generally act as a full-stack development partner.

Because Build has full permissions, you should be aware that it can make changes to your codebase and execute system commands. That's not a risk—it's by design—but it means you should always review what Build is doing, especially for critical operations.

### Plan Agent

**Mode**: Primary
**Default**: No

The Plan agent is the cautious counterpart to Build. While Build is all about action, Plan is all about thinking. It's designed for analysis, planning, and review without making any actual changes to your codebase. By default, Plan has all write, edit, and bash tools set to `ask` permission, meaning you'll be prompted before any changes are made.

Plan is invaluable when you want to:
- Review code changes and get feedback
- Plan out features before implementation
- Analyze problems and create solution strategies
- Explore architectural options
- Generate documentation about existing code

Since Plan doesn't make changes unless you explicitly approve them, it's perfect for situations where you want to think through a problem without risk. You can have Plan analyze your entire codebase, identify potential issues, and suggest improvements, all while knowing that nothing will be modified unless you say so.

### General Agent

**Mode**: Subagent
**Default**: No

The General agent is your research specialist. While the primary agents handle conversations, General is designed to tackle complex, multi-step research tasks that require deep investigation. It has full tool access (except for todo management) and can make file changes when needed for research purposes.

General excels at:
- Answering complex questions that require multiple searches and investigations
- Researching best practices and documentation
- Finding examples and patterns across your codebase
- Gathering information from multiple sources and synthesizing it

You can summon General manually by typing `@general` in your message, or your primary agent might call on it automatically when it detects that a question would benefit from dedicated research.

### Explore Agent

**Mode**: Subagent
**Default**: No

The Explore agent is your codebase investigator. Unlike General, Explore is explicitly read-only—it cannot modify files. This restriction makes it fast and focused, perfect for quick exploration and information gathering. Think of Explore as your detective who can look around but not touch anything.

Explore is optimized for:
- Finding files by pattern or name
- Searching code for specific functions, classes, or keywords
- Answering questions about code structure
- Mapping out how different parts of the codebase connect
- Understanding dependencies and relationships

Because it's read-only, Explore is safe to let loose on your codebase without worrying about accidental changes. It's faster than General for simple lookup tasks and doesn't waste time with capabilities it doesn't need.

To use Explore, type `@explore` followed by your question, like `@explore where is the authentication logic?`.

### Compaction Agent

**Mode**: Primary (Hidden)
**Default**: System

The Compaction agent is a behind-the-scenes worker that manages conversation context. As you have long conversations with OpenCode, the amount of information accumulates. Eventually, you hit the context limit of your AI model—it can only process so much text at once. When this happens, the Compaction agent automatically steps in.

Compaction works by analyzing the conversation history and creating a condensed summary that preserves the important information while discarding details that are no longer relevant. It's like a smart archivist who keeps the essential notes but removes old, unnecessary paperwork.

You never interact with Compaction directly; it runs automatically when needed. You might notice signs that compaction is happening if you see references to earlier compacted messages being summarized, but for the most part, it's invisible.

### Title Agent

**Mode**: Primary (Hidden)
**Default**: System

The Title agent generates concise, descriptive titles for your OpenCode sessions. Every time you start a new conversation, OpenCode needs to know what that conversation is about so you can find it later. The Title agent automatically creates these titles based on the conversation content.

Like Compaction, the Title agent runs automatically in the background. You don't control it directly, but you benefit from its work when you see well-named sessions in your history.

### Summary Agent

**Mode**: Primary (Hidden)
**Default**: System

The Summary agent creates comprehensive summaries of your OpenCode sessions when they conclude or when you explicitly request a summary. These summaries capture the key points, decisions made, code written, and lessons learned.

Session summaries are valuable for:
- Catching up on work you did earlier
- Sharing context with team members
- Documenting decisions and rationale
- Creating audit trails of development work

You can typically trigger a summary with a command like `/summary` in the TUI interface. The Summary agent will then process the entire conversation and produce a condensed overview.

---

## Using Agents

Now that you understand the different types of agents, let's look at how to actually use them in practice.

### Switching Primary Agents

During an active OpenCode session, you can switch between available primary agents using the **Tab** key. This cycles through all primary agents that aren't disabled or hidden. Each time you press Tab, you switch to the next agent, and the interface updates to show which agent is currently active.

When you switch agents, the conversation context remains the same—you're not starting over. You're simply changing who you're talking to. This means if Build has been working on a feature and you switch to Plan, Plan can see everything that was discussed and built so far, and can offer analysis or suggestions based on that context.

The keyboard shortcut for switching agents is configurable through the `switch_agent` keybind in your TUI configuration. If you prefer a different key or key combination, you can customize it.

### Invoking Subagents

Subagents enter the conversation in two ways:

**Automatic Invocation**: When you ask a question or give a task to a primary agent, that agent might recognize that a subagent would be better suited to handle part of the work. For example, if you ask Build to "find all the places where we handle user authentication," Build might automatically invoke the Explore subagent to search the codebase because that's Explore's specialty.

This happens seamlessly—you might see a message indicating that a subagent has been called in, and when it finishes, the primary agent will return with the results.

**Manual Invocation via @-mentions**: You can explicitly call any subagent by typing `@` followed by the agent's name in your message. For example: `@explore find all files related to authentication`. This is like saying, "Hey, I need your expertise on this specific part."

Examples:
- `@explore what files contain database connections?`
- `@general research the best practices for API rate limiting`
- `@explore find all test files for the payment module`

When you use an @-mention, the subagent receives your message as its task, works independently (potentially creating its own conversation thread), and returns results. The primary agent then incorporates those results into the overall conversation.

### Navigating Between Sessions

When subagents are invoked, they sometimes create what are called "child sessions" or "sub-sessions." These are separate conversation threads that maintain their own context. This is particularly important because subagents might need to do multiple rounds of work, and keeping that work isolated prevents it from cluttering the main conversation.

The sub-session system lets you "zoom in" on what the subagent is doing and then "zoom back out" to the main conversation. OpenCode provides keybinds for this navigation:

- **Enter first child session** (`session_child_first`, default: `<Leader>+Down`): When a subagent has created a child session, this keybind lets you enter that session and see exactly what's happening in real-time. You can watch as the subagent searches, thinks, and responds.

- **Cycle between child sessions** (`session_child_cycle`, default: `Right`): If multiple sub-sessions exist, this moves to the next one in sequence.

- **Cycle in reverse** (`session_child_cycle_reverse`, default: `Left`): Moves to the previous child session.

- **Return to parent** (`session_parent`, default: `Up`): Takes you back to the main conversation from whatever sub-session you're currently viewing.

This navigation system is powerful for debugging and oversight. If you ask a question and see that a subagent is invoked, you can jump into that sub-session mid-work to see what's happening, ask clarifying questions, or even cancel it if it's going down the wrong path.

---

## Agent Configuration

Agents in OpenCode are configured through two main formats: JSON in your main configuration file, or Markdown files in dedicated agent directories. Both approaches achieve the same result, so you can choose the one that fits your workflow better.

### JSON Configuration

The primary way to configure agents is in your `opencode.json` or `opencode.jsonc` file using the `agent` key. This file lives in your project root for project-specific configuration, or in `~/.config/opencode/` for global configuration.

Here's the basic structure:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "agent-name": {
      "description": "What this agent does",
      "mode": "primary",
      "model": "anthropic/claude-sonnet-4-5",
      "prompt": "{file:./prompts/build.txt}",
      "tools": {
        "write": true,
        "edit": true,
        "bash": true
      }
    }
  }
}
```

Each agent is a key under the `agent` object. The key becomes the agent's identifier—the name you use to reference it or switch to it. Inside each agent object, you define its properties using the options we'll cover in detail later.

JSON configuration is great for:
- Sharing configuration in version control (like Git)
- Making systematic changes across multiple agents
- Programmatic generation of configurations
- Keeping everything in one place

### Markdown Configuration

Alternatively, you can define agents using Markdown files with YAML frontmatter. This approach is more modular and can be easier to manage for many custom agents or when sharing agent definitions.

To create a Markdown-based agent, create a `.md` file in one of these directories:
- Global: `~/.config/opencode/agents/`
- Per-project: `.opencode/agents/`

The filename (without the `.md` extension) becomes the agent name. So `security-auditor.md` creates an agent named `security-auditor`.

The file structure looks like this:

```markdown
---
description: Performs security audits and identifies vulnerabilities
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.1
tools:
  write: false
  edit: false
  bash: false
---

You are a security expert reviewing code for vulnerabilities.

Look for:
- Input validation issues
- Authentication and authorization flaws
- Data exposure risks
- Dependency vulnerabilities
- Configuration security problems

Provide findings with severity ratings and specific remediation steps.
```

The section between the `---` lines is YAML frontmatter that defines the agent's configuration options. Everything after the closing `---` is the agent's system prompt—the instructions that define how the agent behaves and what it focuses on.

Markdown configuration shines when:
- You have many agents and want them in separate files
- You want to use file-based versioning for individual agents
- You prefer writing prompts in Markdown where you can include formatting, code blocks, and lists
- You're creating a shareable collection of agents for others to use

Both JSON and Markdown configurations can coexist. OpenCode automatically loads agents from all configured locations and merges them according to the configuration precedence rules.

---

## Agent Configuration Options

Now let's examine each configuration option in detail. These options control every aspect of an agent's behavior, capabilities, and appearance.

### Description

The `description` field provides a brief explanation of what the agent does and when to use it. This description serves two important purposes:

1. It helps you remember what each agent is for when you're choosing between them
2. It enables automatic agent selection—primary agents can use these descriptions to decide when to invoke a subagent

A good description is concise but informative. Aim for about 10-20 words that capture the agent's purpose and specialty.

```json
{
  "agent": {
    "code-reviewer": {
      "description": "Reviews code for best practices and potential issues"
    }
  }
}
```

When you see a list of available agents to switch between, these descriptions appear alongside the names. And when a primary agent is deciding whether to call a subagent, it evaluates these descriptions to find the best match for the task.

The description is a **required** option—every agent must have one.

### Temperature

The `temperature` setting controls how creative or deterministic the AI's responses are. This might sound technical, but it's actually quite intuitive.

Think of temperature like this: when the AI generates text, it's essentially predicting what comes next. At low temperatures, it always picks the most likely next word—very predictable and focused. At high temperatures, it's more willing to pick less obvious words, which leads to more variety and creativity.

Temperature values range from 0.0 to 1.0:

**Low Temperature (0.0–0.2)**: The AI is very focused and sticks closely to patterns it has seen. This is ideal for:
- Code analysis and review (where precision matters)
- Technical documentation
- Following specific formats or templates
- Any task where consistency is critical

```json
{
  "agent": {
    "plan": {
      "temperature": 0.1,
      "description": "Analyzes code with focused, deterministic responses"
    }
  }
}
```

**Medium Temperature (0.3–0.5)**: A balance between creativity and consistency. Good for:
- General development tasks
- Writing code with some flexibility
- Most everyday AI assistance

**High Temperature (0.6–1.0)**: The AI is more experimental and varied. Useful for:
- Brainstorming sessions
- Creative problem-solving
- Generating multiple alternative approaches
- Tasks where diversity of thought is valuable

```json
{
  "agent": {
    "brainstorm": {
      "temperature": 0.7,
      "description": "Generates creative ideas and approaches"
    }
  }
}
```

If you don't specify a temperature, OpenCode uses model-specific defaults—typically 0 for most models, or 0.55 for Qwen models. But explicitly setting it gives you control over an agent's "personality."

### Max Steps

The `steps` (formerly `maxSteps`) setting limits how many iterations an agent can go through before it must produce a final response. An "iteration" in this context means a cycle where the AI thinks, uses tools, receives results, and thinks again. This is crucial for controlling costs and preventing runaway agent behavior.

By default, agents continue iterating until the AI decides it's done (or you interrupt). But for categories of work where you want to ensure efficiency, setting a step limit forces the agent to wrap up.

```json
{
  "agent": {
    "quick-research": {
      "description": "Fast research with limited iterations",
      "steps": 5
    }
  }
}
```

In this example, the agent can perform at most 5 cycles of thinking and tool use before it's compelled to provide a final answer. If it reaches the limit, it receives a special system instruction to summarize its findings and list any remaining tasks.

This option is particularly useful for:
- Budget-conscious projects where you want to cap agent time
- Production environments where response time matters
- Agents handling routine, bounded tasks
- Preventing infinite loops in edge cases

If you don't set a limit, the agent can theoretically run indefinitely (within the overall conversation constraints).

**Important**: The legacy `maxSteps` field is deprecated. Use `steps` instead.

### Disable

The `disable` option is a simple boolean that, when set to `true`, completely deactivates an agent. Disabled agents don't appear in agent selection lists, can't be invoked with @-mentions, and are essentially invisible to the system.

```json
{
  "agent": {
    "experimental-agent": {
      "disable": true
    }
  }
}
```

This is useful when:
- You have an agent configuration but want to temporarily turn it off without deleting it
- You're developing a custom agent and want to disable it until it's ready
- You want to remove an agent from rotation without losing the configuration
- You need to troubleshoot issues by eliminating certain agents from consideration

Disabled agents remain in your configuration file but are inert. Changing `disable` to `false` or removing the field entirely reactivates them.

### Prompt

The `prompt` option specifies a file containing the agent's system prompt—the fundamental instructions that define its behavior, expertise, and tone. This file contains the personality and methodology of the agent.

```json
{
  "agent": {
    "code-reviewer": {
      "prompt": "{file:./prompts/review.txt}"
    }
  }
}
```

The `{file:...}` syntax loads the prompt from a file on disk. The path is relative to the location of the configuration file that defines it. So if `prompt: "{file:./prompts/review.txt}"` is in your project's `opencode.json`, it looks for `./prompts/review.txt` in the project root.

Prompt files are plain text that can contain any instructions you want to give the agent. A good prompt typically includes:
- The agent's role and identity
- Its objectives and priorities
- How it should approach problems
- What to avoid or watch out for
- Expected output format (if any)
- Any special constraints or boundaries

For Markdown-defined agents, the prompt is everything after the YAML frontmatter—no separate file needed.

```markdown
---
description: Security auditor
mode: subagent
---

You are a security expert reviewing code for vulnerabilities.

Your methodology:
1. Examine input validation and sanitization
2. Check authentication and access controls
3. Look for data exposure risks
4. Review dependency security
5. Assess configuration safety

Provide specific findings with severity ratings and remediation steps.
```

The content after `---` becomes the prompt. This format is convenient because the prompt is right there in the same file as the configuration.

If you don't specify a prompt, the agent uses a default generic prompt. For most custom agents, you'll want to provide a specific prompt that guides the AI toward the behavior you need.

### Model

The `model` option overrides the default model for this agent, letting you assign different AI models to different tasks based on their strengths.

Models in OpenCode are identified using the format `provider/model-id`. For example:
- `anthropic/claude-sonnet-4-5` (Anthropic's Claude Sonnet)
- `openai/gpt-5` (OpenAI's GPT-5)
- `opencode/gpt-5.1-codex` (OpenCode Zen's GPT-5.1 Codex)

```json
{
  "agent": {
    "plan": {
      "model": "anthropic/claude-haiku-4-5"
    },
    "build": {
      "model": "anthropic/claude-sonnet-4-5"
    }
  }
}
```

In this example, the Plan agent uses the faster, cheaper Haiku model for analysis, while Build uses the more capable Sonnet model for implementation work.

The model option is powerful because different models have different:
- Capabilities (some are better at coding, others at analysis)
- Costs (some are much cheaper per token)
- Speed (some respond much faster)
- Context limits (how much they can process at once)

By assigning models per agent, you can optimize both performance and cost. Your research agent might use a fast model for quick lookups, while your complex builder uses a high-end model for tough coding problems.

If you don't specify a model:
- Primary agents use the globally configured model (the one you set with `/models` or in config)
- Subagents inherit the model from the primary agent that invoked them

This inheritance makes sense—if you're using a powerful model for your main work, your subagents probably want to use the same model to maintain compatibility and quality.

### Tools

The `tools` setting controls which tools (capabilities) an agent has access to. Tools are the actions an agent can perform in your environment—writing files, editing code, running bash commands, fetching web content, etc.

By default, agents inherit the global tool settings, but you can override them per agent to create specialists with focused capabilities.

```json
{
  "agent": {
    "plan": {
      "tools": {
        "write": false,
        "edit": false,
        "bash": false
      }
    }
  }
}
```

This Plan agent configuration disables all modification tools, making it read-only. Even if the global configuration enables these tools, this agent-level setting overrides them.

Tool settings are boolean:
- `true`: The tool is available
- `false`: The tool is unavailable

Common tools include:
- `write`: Create new files
- `edit`: Modify existing files
- `bash`: Execute shell commands
- `read`: Read file contents
- `grep`: Search for patterns in files
- `webfetch`: Retrieve web content
- MCP tools: Tools from connected MCP servers (use patterns like `mymcp_*`)

You can also use wildcards to control groups of tools. For example, to disable all tools from an MCP server:

```json
{
  "agent": {
    "safe-agent": {
      "tools": {
        "mymcp_*": false,
        "write": false,
        "bash": false
      }
    }
  }
}
```

This pattern is powerful for managing integrations. If you have an MCP server with many tools and you want to disable them all for a particular agent, the wildcard does it in one line.

Tool configuration is about safety and focus. A read-only agent can't accidentally modify things. An agent without bash access can't run dangerous commands. And an agent with limited tools is more focused on its specialty.

### Permissions

The `permission` setting goes beyond simple enable/disable for tools—it provides fine-grained control over what actions an agent can take, with different levels of approval.

Permissions apply to the `edit`, `bash`, and `webfetch` tools. For each, you can set:

**`"ask"`**: The agent can attempt the action, but you'll be prompted to approve it first. This is the default for Plan agent's modification tools. It's the middle ground—the agent can proceed but can't surprise you.

```json
{
  "agent": {
    "plan": {
      "permission": {
        "edit": "ask",
        "bash": "ask"
      }
    }
  }
}
```

**`"allow"`**: The agent has unrestricted access. No prompts, no warnings. This is Build's default for most tools, and it's appropriate when you fully trust the agent or the tool is harmless.

**`"deny"`**: The tool is completely disabled. The agent cannot use it at all, not even with your approval. The tool simply doesn't exist from this agent's perspective.

Permissions can also be set globally and overridden per agent, just like tool settings. The agent-level permissions take precedence.

#### Command-Specific Permissions

For the `bash` tool, you can get even more granular by specifying permissions for specific commands. This lets you allow safe commands while restricting dangerous ones, or require approval for specific operations.

```json
{
  "agent": {
    "coder": {
      "permission": {
        "bash": {
          "git status": "allow",
          "git commit": "ask",
          "git push": "ask",
          "rm -rf": "deny"
        }
      }
    }
  }
}
```

This configuration:
- Allows `git status` without prompt (harmless read-only)
- Asks before `git commit` and `git push` (destructive operations that need review)
- Denies `rm -rf` entirely (too dangerous)

You can use glob patterns to match multiple commands:

```json
{
  "agent": {
    "safe-coder": {
      "permission": {
        "bash": {
          "git *": "ask",
          "npm test": "allow",
          "*": "deny"
        }
      }
    }
  }
}
```

Here, any git command requires approval, npm test is always allowed, and all other commands are denied. Pattern order matters when using wildcards—the last matching rule wins, so place the catch-all `"*"` first and specific rules after, or structure logically based on your needs.

You can also set permission patterns in Markdown agent files:

```markdown
---
description: Code reviewer with selective permissions
mode: subagent
permission:
  edit: deny
  bash:
    "*": ask
    "git diff": allow
    "git log*": allow
    "grep *": allow
  webfetch: deny
---
```

This agent can't edit files or fetch web content (read-only focus), but it can run bash commands with approval, except that `git diff`, `git log`, and `grep` are always allowed since they're read-only operations essential for code review.

Permissions are your primary safety mechanism for controlling what agents can do. Use them thoughtfully to balance capability with risk.

### Mode

The `mode` option determines how the agent can be used. It's a simple setting with three possible values:

**`"primary"`**: The agent appears in the primary agent rotation and can be the main conversational partner. Only primary agents can be the default agent and participate in the Tab-based switching. Build and Plan are both primary agents.

**`"subagent"`**: The agent is only available as a subagent—it can't be switched to as a primary agent, and it won't appear in the Tab rotation. Subagents can only be invoked through @-mentions or automatic invocation by primary agents. Explore and General are subagents.

**`"all"`** (default): The agent can function both as a primary agent and as a subagent. This is the default if no mode is specified, giving maximum flexibility.

```json
{
  "agent": {
    "build": {
      "mode": "primary"
    },
    "explore": {
      "mode": "subagent"
    },
    "orchestrator": {
      "mode": "all"
    }
  }
}
```

The mode affects where and how the agent appears in the UI, what keybinds apply to it, and how it can be invoked. For most custom agents, you'll choose either `primary` or `subagent` based on their intended use.

### Hidden

The `hidden` option (boolean, defaults to `false`) hides subagents from the @-mention autocomplete menu. When `true`, users won't see the agent as an option when typing `@`, but the agent can still be invoked programmatically by primary agents using the Task tool.

```json
{
  "agent": {
    "internal-helper": {
      "mode": "subagent",
      "hidden": true
    }
  }
}
```

This is useful for internal subagents that you want other agents to use, but you don't want end users to manually invoke. Think of it like private methods in programming—they exist for internal orchestration but aren't part of the public interface.

**Important**: `hidden` only applies to `mode: subagent` agents. It has no effect on primary agents.

### Task Permissions

The `permission.task` setting (note the dot notation) controls which subagents an agent can invoke via the Task tool. This is about what other agents this agent can call upon for assistance.

Task permissions use glob patterns for flexible matching:

```json
{
  "agent": {
    "orchestrator": {
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

In this configuration:
- The orchestrator cannot invoke any subagent by default (`"*": "deny"`)
- It can invoke subagents whose names start with `orchestrator-` (`"orchestrator-*": "allow"`)
- It can invoke the `code-reviewer` agent but needs approval (`"code-reviewer": "ask"`)

Permissions values for task invocation:
- `"allow"`: Can invoke without restriction
- `"ask"`: User will be asked to approve the invocation
- `"deny"`: Cannot invoke; the subagent won't even appear in the Task tool's description to the model

When set to `deny`, the subagent is completely invisible to the agent—the AI doesn't even know it's an option. This prevents the model from wasting time considering subagents that aren't available.

**Important**: Task permissions don't restrict what users can do. Users can always invoke any subagent directly via @-mention, regardless of these settings. Task permissions only control what the agent itself can do autonomously.

### Color

The `color` option sets the agent's visual appearance in the OpenCode interface. It's purely cosmetic but can help you distinguish between different agents at a glance.

You can specify colors in two ways:

1. **Hex color codes**: `"#FF5733"`, `"#4CAF50"`, etc.
2. **Theme colors**: `"primary"`, `"secondary"`, `"accent"`, `"success"`, `"warning"`, `"error"`, `"info"`

```json
{
  "agent": {
    "security": {
      "color": "#ff4444"
    },
    "docs": {
      "color": "accent"
    }
  }
}
```

If you don't set a color, the agent uses the default primary color. Using distinctive colors can be helpful when you have many agents and need to quickly identify which one is active in the UI.

### Top P

The `top_p` option is an alternative to temperature for controlling response diversity. Instead of adjusting the sampling temperature, `top_p` uses nucleus sampling—where the AI considers only the most likely tokens whose cumulative probability exceeds `top_p`.

Values range from 0.0 to 1.0:
- Lower values (`0.1–0.5`): More focused, deterministic responses (similar to low temperature)
- Higher values (`0.7–1.0`): More diverse, varied responses

```json
{
  "agent": {
    "explore": {
      "top_p": 0.9
    }
  }
}
```

Most users will primarily use `temperature`. `top_p` is an advanced option for fine-tuning generation behavior, and you generally shouldn't set both in the same agent configuration as they can conflict.

### Additional (Pass-through Options)

Beyond the explicitly defined options, *any other* configuration you include in an agent definition will be passed directly to the AI provider as a model parameter. This is a powerful escape hatch that lets you use provider-specific features without OpenCode needing to explicitly support them.

```json
{
  "agent": {
    "deep-thinker": {
      "model": "openai/gpt-5",
      "reasoningEffort": "high",
      "textVerbosity": "low"
    }
  }
}
```

In this example, `reasoningEffort` and `textVerbosity` aren't standard OpenCode agent options. They're specific to OpenAI's reasoning models and get passed straight through to the provider.

This pass-through mechanism means OpenCode automatically supports new model parameters as providers introduce them—no OpenCode update needed. Whatever options your model accepts can be configured here.

To discover what options your provider supports, check their documentation. Common examples include:

- OpenAI: `reasoningEffort`, `textVerbosity`, `reasoningSummary`, `include`
- Anthropic: `thinking` (with `type` and `budgetTokens`)
- Google: `top_k`, `top_p`, `temperature` (though temperature is already explicit)

These additional options are written directly in the agent object alongside the standard ones:

```json
{
  "agent": {
    "my-agent": {
      "model": "openai/gpt-5",
      "temperature": 0.3,
      "reasoningEffort": "high",
      "textVerbosity": "low"
    }
  }
}
```

---

## Creating Custom Agents

OpenCode makes it straightforward to create your own agents tailored to your specific needs. You have two options: use the interactive command or create the configuration manually.

### Interactive Creation

Run this command in your terminal:

```bash
opencode agent create
```

This launches an interactive wizard that guides you through:

1. **Location choice**: Save globally (available in all projects) or in the current project (only for this project)

2. **Agent name**: A short identifier (letters, numbers, hyphens). This is the name you'll use to reference the agent.

3. **Description**: A brief explanation of what the agent does and when to use it.

4. **Model selection**: Choose which AI model powers this agent, or accept the default.

5. **Tool selection**: Pick which tools the agent should have access to using a checklist interface.

The wizard then generates a Markdown file with a complete configuration and places it in the appropriate directory (`.opencode/agents/` for project-specific, `~/.config/opencode/agents/` for global).

This is the easiest way to create agents and ensures you don't miss any required fields.

### Manual Creation

You can also create agent files manually. For a Markdown agent, create a new `.md` file in `.opencode/agents/` (project) or `~/.config/opencode/agents/` (global).

Example: `.opencode/agents/documentation-writer.md`

```markdown
---
description: Writes and maintains project documentation
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.4
tools:
  write: true
  edit: true
  bash: false
permission:
  edit: ask
---

You are a technical writer specializing in clear, comprehensive documentation.

Your documentation should:
- Start with a clear purpose statement
- Use headings to organize content logically
- Include code examples where relevant
- Explain concepts in simple language
- Follow consistent formatting

When writing documentation:
1. First understand the code or feature
2. Identify the audience (developers, users, administrators)
3. Structure from general to specific
4. Include practical examples
5. Review for clarity and completeness
```

The filename (`documentation-writer.md`) determines the agent name (`documentation-writer`). You can now invoke this agent with `@documentation-writer` in your messages.

For JSON configuration, add an entry to the `agent` section of your `opencode.json`:

```json
{
  "agent": {
    "documentation-writer": {
      "description": "Writes and maintains project documentation",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.4,
      "tools": {
        "write": true,
        "edit": true,
        "bash": false
      },
      "permission": {
        "edit": "ask"
      },
      "prompt": "You are a technical writer specializing in clear, comprehensive documentation..."
    }
  }
}
```

Either format works; use whichever fits your workflow. Markdown files are often easier for team sharing and versioning individual agents, while JSON keeps everything centralized.

---

## Use Cases and Examples

Let's explore practical applications for agents with complete configuration examples.

### Code Review Agent

A specialized agent for reviewing code without making changes. Perfect for pull request analysis, pre-commit reviews, or periodic codebase audits.

```markdown
---
description: Reviews code for quality, security, and best practices
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.2
tools:
  write: false
  edit: false
  bash: true
permission:
  bash:
    "*": ask
    "git status": allow
    "git diff": allow
    "grep *": allow
---

You are a senior code reviewer with expertise in security, performance, and maintainability.

Your review process:
1. Understand the code's purpose and context
2. Check for security vulnerabilities (injection, XSS, authentication issues)
3. Evaluate error handling and edge cases
4. Assess performance implications
5. Review code style and consistency
6. Check for code smells and technical debt

Provide specific line references when possible. Categorize issues by severity (critical, major, minor).

When suggesting improvements:
- Explain the problem clearly
- Provide concrete examples
- Offer specific fixes
- Consider maintainability

Do not make changes—only recommend them.
```

### Debugging Agent

An agent focused on troubleshooting and issue investigation.

```json
{
  "agent": {
    "debugger": {
      "description": "Investigates bugs and performance issues",
      "mode": "subagent",
      "model": "openai/gpt-5",
      "temperature": 0.3,
      "tools": {
        "write": false,
        "edit": false,
        "bash": true,
        "grep": true,
        "read": true
      },
      "permission": {
        "bash": {
          "*": ask,
          "tail *": allow,
          "cat *": allow,
          "ps aux": allow,
          "netstat": allow
        }
      },
      "prompt": "You are a debugging specialist. When investigating issues:\n\n1. Gather information about the problem\n2. Check logs and error messages\n3. Trace execution flow\n4. Identify root cause\n5. Suggest reproducible test cases\n6. Recommend fixes with rationale"
    }
  }
}
```

### Documentation Specialist

An agent dedicated to writing, updating, and organizing documentation.

```markdown
---
description: Creates and maintains project documentation
mode: subagent
model: openai/gpt-5
temperature: 0.5
tools:
  write: true
  edit: true
  read: true
  bash: false
permission:
  edit: ask
---

You are a technical documentation specialist. Your mission is to create clear, comprehensive documentation that serves its intended audience.

Documentation types you handle:
- README files (project overview, setup, usage)
- API documentation (endpoints, parameters, examples)
- Developer guides (architecture, patterns, best practices)
- User guides (step-by-step instructions)
- Code comments (explaining complex logic)

Principles of good documentation:
1. Know your audience (adjust technical level accordingly)
2. Start with the problem, not the solution
3. Use examples liberally
4. Keep it up to date—flag outdated content
5. Structure hierarchically (overview → details)
6. Include both "how" and "why"

When updating existing documentation:
- Preserve what's still relevant
- Flag deprecated information clearly
- Maintain consistent voice and style
- Update cross-references

Output format:
- Use Markdown formatting appropriately
- Include code blocks with language hints
- Add diagrams for complex flows when needed
- Link to related resources
```

### Security Auditor

A focused agent for identifying security vulnerabilities.

```markdown
---
description: Performs security audits and identifies vulnerabilities
mode: subagent
model: anthropic/claude-opus-4
temperature: 0.1
tools:
  write: false
  edit: false
  bash: true
  read: true
  grep: true
permission:
  bash:
    "*": ask
    "grep *": allow
---

You are a security expert conducting a comprehensive security audit.

Your audit covers:
- Input validation and sanitization
- Authentication and authorization mechanisms
- Session management
- Data exposure risks (PII, secrets in code)
- Dependency vulnerabilities (check package manifests)
- Configuration security (hardcoded credentials, exposed ports)
- Injection vulnerabilities (SQL, command, XSS)
- Cryptographic implementation issues

For each finding:
1. Describe the vulnerability clearly
2. Rate severity (Critical/High/Medium/Low)
3. Explain the potential impact
4. Provide concrete remediation steps
5. Reference relevant security guidelines (OWASP, CWE)

Present findings in a structured report format with:
- Executive summary
- Detailed findings by category
- Prioritized remediation plan
- Positive observations (secure practices found)

Focus on actionable intelligence, not just theoretical risks.
```

### Exploratory Research Agent

For broad investigation and information gathering.

```json
{
  "agent": {
    "researcher": {
      "description": "Conducts broad research and synthesizes information",
      "mode": "subagent",
      "model": "openai/gpt-5",
      "temperature": 0.6,
      "tools": {
        "read": true,
        "grep": true,
        "webfetch": true
      },
      "permission": {},
      "prompt": "You are a research specialist who excels at gathering and synthesizing information from diverse sources.\n\nYour approach:\n1. Clarify the research question\n2. Identify all relevant information sources\n3. Systematically explore each source\n4. Take notes on key findings\n5. Synthesize into comprehensive answer\n6. Identify gaps or uncertainties\n\nWhen researching codebases:\n- Map the relevant components\n- Trace data flows and dependencies\n- Find related files and modules\n- Extract patterns and conventions\n\nWhen using web fetch:\n- Verify source credibility\n- Extract only relevant information\n- Note the source and date\n\nOrganize findings hierarchically and highlight key insights."
    }
  }
}
```

### Architecture Planning Agent

For high-level design and planning work.

```json
{
  "agent": {
    "architect": {
      "description": "Designs system architecture and plans implementation",
      "mode": "primary",
      "model": "anthropic/claude-opus-4",
      "temperature": 0.3,
      "tools": {
        "write": false,
        "edit": false,
        "bash": false,
        "read": true,
        "grep": true
      },
      "permission": {
        "edit": "deny",
        "bash": "deny"
      },
      "prompt": "You are a system architect focused on robust, scalable designs.\n\nYour planning process:\n1. Analyze requirements thoroughly\n2. Identify constraints and dependencies\n3. Consider trade-offs (simplicity vs flexibility, performance vs maintainability)\n4. Choose appropriate patterns and technologies\n5. Document the architecture with diagrams and explanations\n6. Create implementation milestones\n7. Identify risks and mitigation strategies\n\nArchitecture artifacts you produce:\n- System component diagrams\n- Data flow diagrams\n- API specifications\n- Database schemas\n- Technology stack recommendations\n- Implementation roadmap\n\nConsider:\n- Scalability boundaries\n- Failure modes and resilience\n- Security implications\n- Operational concerns (monitoring, deployment)\n- Team skill alignment\n\nProvide specific, actionable recommendations with rationale."
    }
  }
}
```

---

## Integration with Config, Providers, and Models

Agents don't exist in isolation—they integrate deeply with OpenCode's configuration system, provider management, and model selection. Understanding these integrations helps you use agents effectively.

### Configuration Hierarchy

Agent configurations participate in OpenCode's layered configuration system. The precedence order is:

1. **Remote config** (from `.well-known/opencode`): Organizational defaults
2. **Global config** (`~/.config/opencode/opencode.json`): Your user preferences
3. **Custom config** (`OPENCODE_CONFIG` env var): Custom overrides
4. **Project config** (`opencode.json` in project): Project-specific settings
5. **`.opencode` directories**: Agents, commands, plugins
6. **Inline config** (`OPENCODE_CONFIG_CONTENT` env var): Runtime overrides

This means:
- You can define agents globally and override them project-by-project
- Organizations can provide agent standards via remote config
- Project configs have the final say for that project

For agents specifically, they can be defined in:
- The `agent` key in any config file at any precedence level
- Markdown files in agents/ directories at any precedence level (`~/.config/opencode/agents/`, `.opencode/agents/`, or custom directories via `OPENCODE_CONFIG_DIR`)

All agent definitions are merged together, with later sources overriding earlier ones for identically-named agents.

### Provider Integration

Agents use models from your configured providers. When you specify a model for an agent like `anthropic/claude-sonnet-4-5`, OpenCode looks for the `anthropic` provider configuration to determine how to connect to Anthropic's API.

The provider documentation shows that you set up providers via:
1. The `/connect` command to add credentials (stored in `~/.local/share/opencode/auth.json`)
2. Provider-specific configuration in the `provider` section of your config

An agent's `model` setting must reference a provider that is configured and available. If you specify a model from a provider that hasn't been connected, the agent won't work—OpenCode will fail to load that agent with an error.

You can also create custom providers for OpenAI-compatible APIs. In that case, your agent can use models from that custom provider:

```json
{
  "provider": {
    "my-company-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Company AI",
      "options": {
        "baseURL": "https://api.mycompany.com/v1"
      },
      "models": {
        "company-model": {
          "name": "Company Model v1"
        }
      }
    }
  },
  "agent": {
    "internal-coder": {
      "model": "my-company-ai/company-model"
    }
  }
}
```

### Model Configuration Integration

Agents' model settings integrate with OpenCode's model system, which includes:

- **Model selection** via `/models` command or `model` config key
- **Model variants** (like `high`, `max`, `low` reasoning settings)
- **Model-specific options** configured in the `provider.models` section
- **Model limits** (context window, output limits)

When an agent specifies `model: "anthropic/claude-sonnet-4-5"`, that model might have default options defined globally for the Anthropic provider. The agent can also override those options with its own settings.

```json
{
  "provider": {
    "anthropic": {
      "models": {
        "claude-sonnet-4-5": {
          "options": {
            "thinking": {
              "type": "enabled",
              "budgetTokens": 8000
            }
          }
        }
      }
    }
  },
  "agent": {
    "planner": {
      "model": "anthropic/claude-sonnet-4-5",
      "thinking": {
        "type": "enabled",
        "budgetTokens": 16000
      }
    }
  }
}
```

In this example, the provider defines a default thinking budget of 8000 tokens for Claude Sonnet. The `planner` agent overrides that with 16000 tokens. The agent-level options take precedence.

### Configuration Merging Example

Let's trace how a complete configuration might come together.

You have:

**Global config** (`~/.config/opencode/opencode.json`):

```json
{
  "model": "anthropic/claude-sonnet-4-5",
  "agent": {
    "plan": {
      "model": "anthropic/claude-haiku-4-5"
    }
  }
}
```

**Project config** (`opencode.json` in project):

```json
{
  "agent": {
    "plan": {
      "temperature": 0.2
    },
    "docs-writer": {
      "description": "Project documentation specialist"
    }
  }
}
```

**Project agents directory** (`.opencode/agents/docs-writer.md`):

```markdown
---
mode: subagent
model: openai/gpt-5
---

You are a documentation writer...
```

The resulting agent configuration that OpenCode actually uses:

- `plan` agent:
  - Model: `anthropic/claude-haiku-4-5` (from global, unchanged by project)
  - Temperature: `0.2` (from project override)
  - Other properties: as defined for the built-in Plan agent (since it exists in OpenCode defaults)

- `docs-writer` agent:
  - Description: "Project documentation specialist" (from project's JSON config)
  - Mode: `subagent` (from Markdown file)
  - Model: `openai/gpt-5` (from Markdown file, overrides default)
  - Prompt: "You are a documentation writer..." (from Markdown file)

This demonstrates the flexibility—you can mix configuration formats and sources, with each layer adding or overriding properties.

---

## Best Practices

Based on experience with OpenCode agents, here are recommendations for effective agent configuration and usage.

### Start Simple

When you're new to agents, begin with the built-in Build and Plan agents. They cover 90% of everyday needs. Build handles implementation; Plan handles analysis. You can accomplish a tremendous amount with just these two.

Only create custom agents when you find yourself repeatedly asking for the same kind of specialized help. If you're always saying "review this for security," a security auditor agent makes sense. If it's a one-time need, just ask directly.

### Clear, Descriptive Agent Names

Agent names are your handle for invoking them. Choose names that are:
- Short but meaningful
- Using hyphens to separate words (kebab-case)
- Intuitive to type and remember
- Distinct from each other

Good names: `security-reviewer`, `doc-writer`, `debug-explorer`, `architect`
Bad names: `agent1`, `foo`, `mycustomagentthatdoesthings`

### Good Prompts Matter

The quality of your agent's responses depends heavily on the prompt. Invest time in writing good prompts. A well-crafted prompt includes:

- **Role definition**: "You are a security auditor with 10 years of experience..."
- **Methodology**: "Your process is: 1) Input validation check, 2) Auth review, 3) Data flow analysis..."
- **Output format**: "Provide findings as: Severity: [rating], Location: [file:line], Issue: [description], Fix: [recommendation]"
- **Constraints**: "Do not modify files. Only analyze and report."
- **Priorities**: "Focus on security over style. Cache performance over minor inefficiencies."

Think of the prompt as the agent's job description and instruction manual. The more specific you are, the more focused the agent's responses will be.

### Temperature Strategy

As a rule of thumb:
- **Analyze, review, plan**: 0.1–0.3 (focused, deterministic, consistent)
- **Code generation**: 0.3–0.5 (balanced creativity)
- **Brainstorming, exploration**: 0.6–0.8 (diverse ideas)

Don't set temperature too high for critical tasks—you want reliability, not randomness. And don't set it too low for creative tasks—you want variety, not the same answer every time.

### Use Permissions for Safety

For agents that you trust but don't want running unchecked, use `permission.ask` for destructive operations. This gives you a chance to review before something happens. For truly safe operations (reading files, grep searches), use `permission.allow` so the agent can work without interruption.

Build is configured with most tools as `allow` because it's your main worker. Plan is configured with modification tools as `ask` because it's for analysis. That pattern makes sense: the more powerful the agent's capabilities, the more oversight you want.

### Tool Restrictions Create Focus

Don't give an agent every tool unless it needs them. A documentation writer doesn't need bash access. A code reviewer doesn't need file write access. By restricting tools, you:
- Make the agent more focused on its specialty
- Reduce potential for accidental damage
- Speed up the agent's decision-making (fewer tool choices)
- Create clearer mental models of what each agent can do

### Agent Color Coding

Use the `color` option to visually distinguish agents. Pick colors that make sense:
- Red/orange for aggressive or destructive agents (build)
- Blue for analytical agents (plan, security)
- Green for creative or generative agents (docs)
- Yellow for exploratory agents (explore, researcher)

Consistent colors help you quickly identify which agent is active just by glancing at the interface.

### Model Selection Strategy

Match models to agent needs:
- **Fast, cheap models** (Haiku, GPT-5 Nano): For simple tasks, quick lookups, subagents that run frequently
- **Mid-tier models** (Sonnet, GPT-5): For general development work
- **High-end models** (Opus, GPT-5 Codex): For complex reasoning, architecture, critical code generation

Consider cost and speed trade-offs. You don't need the most expensive model for everything.

### Version Control for Agent Configs

If you're using Markdown-based agents, you can version them individually in Git. Even if you use JSON configuration, the configuration file itself should be in version control. This gives you:
- History of agent changes
- Ability to roll back problematic configs
- Team consistency (everyone uses the same agents)
- Documentation of why agents were created (through commit messages)

### Testing New Agents

After creating a new agent, test it with simple tasks to verify it behaves as expected. Check that:
- It has the right tool access
- Its prompt produces the right tone and focus
- Permissions work as intended
- The model you selected is appropriate

Run a few test queries and observe the behavior before deploying the agent for serious work.

### Subagent Task Permissions

Use `permission.task` to control which subagents an agent can invoke. This creates a hierarchy of control. For example, you might have a general-purpose "orchestrator" agent that can only invoke your custom, trusted subagents—not arbitrary ones the user might @-mention.

Remember: task permissions control what the *agent* can do autonomously. They don't restrict what *users* can do via @-mentions.

### Keep System Agents Alone

Don't modify or disable the hidden system agents (compaction, title, summary). They're essential for OpenCode's operation and work automatically. If you're experiencing issues with them, it's almost certainly a bug to report, not something to fix by disabling them.

### Documentation

Document your custom agents. In the agent's Markdown file or in a separate README, explain:
- What the agent is for
- When to use it vs other agents
- Any special prompts to use with it
- Known limitations or gotchas

Good documentation helps not just your future self, but also teammates if you share your configuration.

---

## Troubleshooting

When agents don't behave as expected, here are common issues and solutions.

### Agent Not Appearing in Selection

**Symptoms**: You configured an agent but it doesn't show up in the Tab-switch list or @-mention autocomplete.

**Causes and solutions**:
1. **Agent is disabled**: Check `disable: false` or remove the `disable` field.
2. **Wrong mode**: For Tab-switching, the agent must be `mode: "primary"` or `"all"`, not `"subagent"`. Subagents only appear in @-mentions.
3. **Configuration not loaded**: Verify your config file is in a recognized location (`opencode.json` in project root, `~/.config/opencode/agents/`, or `.opencode/agents/`). Run `opencode config show` to see the effective configuration.
4. **Syntax error**: Invalid YAML frontmatter or JSON will cause the agent to be skipped. Validate your configuration format.
5. **File extension**: For Markdown agents, ensure the file ends with `.md`, not `.markdown` or `.txt`.

### Agent Not Invoking Subagents

**Symptoms**: You @-mention a subagent or expect an automatic invocation, but nothing happens.

**Causes and solutions**:
1. **Task permissions**: The primary agent might not have permission to invoke that subagent. Check the primary agent's `permission.task` settings.
2. **Missing description**: Subagents need a `description` for primary agents to recognize when to invoke them automatically. `@`-mentions don't require description, but automatic invocation does.
3. **Wrong name**: Check the exact agent name. If your file is `security-auditor.md`, the agent name is `security-auditor`, not `security` or `auditor`.
4. **Subagent hidden**: Hidden subagents can't be manually invoked via @-mention (only programmatically by other agents).

### Agent Using Wrong Model

**Symptoms**: Your agent is using a different AI model than you configured.

**Causes and solutions**:
1. **Model not available**: If the model ID is misspelled or refers to a provider you haven't connected, OpenCode falls back to the default model. Check your `/connect` setup and the model ID format (`provider/model-id`).
2. **Global override**: No—agent model settings should take precedence. But if the agent's model config is invalid, it may fall back to globals.
3. **Cached configuration**: Restart OpenCode to ensure config changes take effect.

### Tools Not Working

**Symptoms**: An agent can't use a tool that you thought was enabled.

**Causes and solutions**:
1. **Tool disabled in agent config**: Check the agent's `tools` settings. Agent-level overrides take precedence over global tool configuration.
2. **Permission denied**: The tool might be enabled but have `permission: "deny"` set. Check both `tools` and `permission` settings.
3. **Provider limitation**: Some models don't support certain tools. Check if your model choice is compatible with the tools you need.
4. **MCP server offline**: If the tool comes from an MCP server, ensure that server is running and connected.

### Agent Behaviors Unexpected

**Symptoms**: Agent responses are too verbose, too brief, too creative, or too rigid.

**Causes and solutions**:
1. **Adjust temperature**: The single biggest lever on response style. Lower for consistency (0.1–0.3), higher for creativity (0.6–0.8).
2. **Adjust `max steps`**: Too many steps can lead to overthinking; too few can cut off reasoning. Adjust based on task complexity.
3. **Prompt refinement**: The agent might not understand its role properly. Rewrite the prompt with clearer instructions, examples, or constraints.
4. **Model mismatch**: Some models are inherently more verbose or creative. Try a different model that matches your desired behavior.

### Subagent Creating Unnecessary Work

**Symptoms**: Primary agents are spawning subagents for simple tasks that could be handled directly.

**Causes and solutions**:
1. **Subagent descriptions too broad**: If a subagent's description makes it a good match for many tasks, primary agents will invoke it frequently. Make descriptions more specific to limit automatic invocations.
2. **You can disable automatic subagent use**: Unfortunately, there's no direct setting to prevent primary agents from calling subagents. The workaround is to either remove/disable the subagent if you never want it used, or adjust its description to be very narrow.
3. **Manual invocation preference**: Remember you can always @-mention the primary agent directly instead of relying on automatic subagent selection.

### Configuration Precedence Confusion

**Symptoms**: You set an agent option in your project `opencode.json`, but it seems ignored; the agent uses the global configuration instead.

**Causes and solutions**:
1. **Precedence order**: Configs are merged, not replaced. Check that you're not accidentally using a different agent name. An agent defined in both global and project configs will have its properties merged, with project overriding global for specific keys.
2. **Nested structure**: Ensure you're placing the agent under the `"agent"` key correctly.
3. **Run `opencode config show`**: This command displays the fully merged configuration, showing you exactly what settings are in effect. Use it to debug precedence issues.

### Model Selection Issues

**Symptoms**: `/models` command doesn't show expected models, or `model: "anthropic/claude-sonnet-4-5"` gives an error.

**Causes and solutions**:
1. **Provider not connected**: Run `/connect` and ensure you've added credentials for the provider.
2. **Model ID format**: Use `provider/model-id` format. The provider part is the key used in the `provider` config section or the built-in provider name. The model-id is the specific model identifier.
3. **Model not configured**: Some providers need explicit model configuration in the `provider.models` section. Check provider documentation.
4. **Authentication failure**: API key may be invalid or expired. Re-run `/connect` to refresh credentials.

### Custom Provider Not Working

**Symptoms**: You added a custom provider configuration but models don't appear in `/models`.

**Causes and solutions**:
1. **Custom provider not in config**: Ensure the provider ID appears in the `provider` section.
2. **Missing `npm` field**: Custom providers need `npm: "@ai-sdk/openai-compatible"` or similar to specify which AI SDK package to use.
3. **Missing `models` section**: You must list the models available from this provider under `provider.<id>.models`.
4. **Typo in provider ID**: The provider ID you use in agent model references must exactly match the key in the `provider` section.
5. **Network/endpoint issues**: If using a local server (Ollama, LM Studio), ensure it's running and accessible at the configured `baseURL`.

---

## Conclusion

OpenCode agents are a powerful way to customize your AI coding assistant. By understanding the distinction between primary and subagents, mastering the configuration options, and creating thoughtful custom agents, you can build a team of specialists that perfectly matches your workflow.

Remember: the best agent setup is the one that feels natural and productive to you. Start with the built-in agents, experiment with custom ones for repetitive specialized tasks, and continuously refine based on what works. OpenCode's configuration system is designed to be flexible—use that flexibility to create an experience that fits your unique needs.

As you work with agents, you'll discover patterns: certain tasks always go together, certain questions always need the same kind of expertise. Those patterns are opportunities for custom agents. Document your successful agent configurations, share them with your team, and iterate.

OpenCode is a tool that evolves with you. As your projects grow in complexity, your agents can grow with them—specializing, refining, and adapting to serve your development practice better and better.

Happy coding—may your agents be ever helpful and your merges conflict-free.

---

## Appendix

### Agent Configuration Quick Reference

**Minimum required fields for any agent:**
- `description`: Short explanation of purpose
- (For primary agents) `mode: "primary"` or `"all"`
- (For subagents) `mode: "subagent"` or `"all"`

**Common optional fields:**
- `model`: Override default model
- `temperature`: Control randomness (0.0–1.0)
- `steps`: Maximum iteration count
- `tools`: Enable/disable specific tools
- `permission`: Fine-grained action permissions
- `prompt`: Custom system instructions

**Visual customization:**
- `color`: Hex code or theme color name
- `mode`: How the agent can be invoked
- `hidden`: Hide from autocomplete (subagents only)

**Advanced:**
- `permission.task`: Control subagent invocation rights
- `additional options`: Pass-through to provider

### Built-in Agent Summary

| Agent | Mode | Description | Default Tools |
|-------|------|-------------|---------------|
| build | primary | Full development capability | All enabled |
| plan | primary | Analysis and planning only | Write, edit, bash disabled or ask |
| general | subagent | Research and complex tasks | Full access (except todo) |
| explore | subagent | Read-only codebase exploration | Read-only |
| compaction | hidden | Context management | System-only |
| title | hidden | Session title generation | System-only |
| summary | hidden | Session summarization | System-only |

### Migration Notes

- `maxSteps` is deprecated; use `steps` instead
- Legacy `agent/` singular directory names still work but `agents/` is preferred
- The `small_model` config option exists for certain system operations but doesn't correspond to a named agent you can switch to

### Further Reading

- [Configuration](/docs/config/) - Full config schema and precedence
- [Providers](/docs/providers/) - Setting up AI providers
- [Models](/docs/models/) - Model selection and variants
- [Permissions](/docs/permissions/) - Detailed permission system
- [Tools](/docs/tools/) - Available tools and MCP integration
- [Plugins](/docs/plugins/) - Extending with plugins
- [Troubleshooting](/docs/troubleshooting/) - General OpenCode troubleshooting