# OpenCode Agents: A Beginner's Guide

## Quick Glossary
**Agent** - A specialized AI helper that performs specific tasks  
**Build agent** - Your main coding assistant that writes and modifies code  
**Plan agent** - Your analysis assistant that reviews code without changing anything  
**Primary agent** - An agent you talk to directly and switch between  
**Subagent** - A specialist you call for specific tasks using `@name`  
**Configuration** - A file with your agent settings (like an instruction manual)  
**Tools** - Actions an agent can take (write, edit, read, etc.)  
**Permission** - Whether an agent needs approval before using a tool  
**Model** - The AI "brain" powering the agent (Claude Sonnet, GPT-5, etc.)  
**Prompt** - Instructions that tell an agent how to behave  
**Temperature** - Creativity setting: 0 (very predictable) to 1 (very creative)  
**Provider** - The company that makes the AI (Anthropic, OpenAI, etc.)  
**@-mention** - Typing `@agent-name` to summon a specific agent  
**Keybind** - A keyboard shortcut

---

## What Are Agents? (And Why Should You Care?)

Imagine you're working on a project and you have a team of specialized helpers:
- One person who actually builds things (the **Builder**)
- One person who reviews work for mistakes (the **Planner**)
- One person who searches through old files (the **Explorer**)
- One person who researches best practices (the **Researcher**)

That's what OpenCode agents are! Instead of having one generic AI that tries to do everything, you can create or use specialized AI helpers for different kinds of tasks.

**The best part**: You don't need to create any custom agents to get started. OpenCode comes with several built-in agents that handle most everyday needs.

### What You Can Do Right Now (Without Reading Further)

If you have OpenCode open, try these:

1. **Press Tab** (or your switch agent key) to cycle through different agents. Notice the names: `build`, `plan`.
2. **Ask Build** to create a simple file: "Create a hello.js file that prints 'Hello World'"
3. **Switch to Plan** (press Tab) and ask: "Review the hello.js file for any issues"
4. **Call Explore** by typing: `@explore what files are in this project?`

That's 50% of what you need to know. The rest is just learning to customize these helpers for your specific needs.

---

## Part 1: Built-In Agents (What You Get for Free)

OpenCode ships with seven agents. You'll mainly use two, occasionally use two more, and never interact with the last three (they work automatically).

### Primary Agents (Your Main Partners)

These are the agents you switch to and talk to directly.

#### Build Agent (Your Default Helper)
- **What it does**: Writes code, edits files, runs commands - does the actual work
- **When to use**: Everyday coding, creating new features, fixing bugs, making changes
- **Tools**: Can do everything (write, edit, bash, read, grep, webfetch)
- **Example use**: "Add login functionality to my app" or "Refactor this messy function"
- **Switch to it**: Press Tab until you see `build` in the interface

#### Plan Agent (Your Safety Net)
- **What it does**: Analyzes, reviews, plans - but does NOT make changes unless you approve
- **When to use**: Code review, planning features, analyzing problems, generating documentation about existing code
- **Tools**: Can read and analyze, but modification tools require your approval
- **Example use**: "Review my recent changes for security issues" or "Plan out the database schema for this feature"
- **Switch to it**: Press Tab until you see `plan` in the interface

**Pro tip**: Start most tasks with Build. When you want a second opinion, press Tab to switch to Plan and ask it to review what Build did.

### Subagents (Your Specialists)

These aren't in the Tab rotation. You call them by typing `@name` in your message.

#### Explore Agent (Your Detective)
- **What it does**: Searches and explores your codebase without changing anything
- **When to use**: "Where is the authentication code?", "What files contain database connections?", "How is the user module structured?"
- **Perfect for**: Understanding a new codebase, finding where something is defined, mapping out dependencies
- **How to call**: Type `@explore find all files related to payments`

#### General Agent (Your Researcher)
- **What it does**: Conducts deep, multi-step research on complex questions
- **When to use**: "What are the best practices for API rate limiting?", "Research how OAuth 2.0 works", "Find examples of error handling patterns in our codebase"
- **Perfect for**: Questions that require multiple searches and synthesis of information
- **How to call**: Type `@general research the recommended approach for file uploads`

**What about the other three?**
- **Compaction agent**: Automatically summarizes old conversations when they get too long. Invisible.
- **Title agent**: Creates session titles automatically. Invisible.
- **Summary agent**: Creates summaries when you end a session or type `/summary`. Invisible.

You can ignore these - they work automatically in the background.

---

## Part 2: Getting Started - Hands-On Guide

### Switching Between Agents

During an OpenCode session, you can switch between available primary agents using the **Tab** key (this might be different if you customized it).

**What happens when you switch?**
- The conversation continues exactly where you left off
- You're just talking to a different person now
- The new agent can see everything that was discussed so far
- Example: Build writes some code → Switch to Plan → Plan reviews that code → Switch back to Build → Build implements Plan's suggestions

**The fastest workflow**:
1. Build creates something
2. Tab → switch to Plan
3. "Review what we just did"
4. Tab → back to Build
5. "Make the changes suggested"

### Calling Subagents with @-mentions

Instead of switching, you can summon a specialist while staying with your current agent.

**Automatic invocation**: If you ask Build "where is the authentication logic?" it might automatically call Explore because that's Explore's specialty. You'll see a message indicating a subagent was called.

**Manual invocation**: You decide which specialist to use by typing `@` followed by the agent name:
```
@explore find all files that handle user passwords
@general what's the standard way to store API keys securely?
```

**What happens next**:
1. The subagent receives your message as its task
2. It works independently (you may see it create a separate conversation thread)
3. It returns results to your main agent
4. The main agent incorporates those results into its response

**Navigating subagent conversations**: If a subagent creates its own thread, you can jump in:
- Use the default keybinds to navigate between parent and child sessions
- Default: `<Leader>+Down` to enter first child, `Right/Left` to cycle, `Up` to return to parent
- This lets you watch what the subagent is doing in real-time or intervene if needed

---

## Part 3: Do You Need a Custom Agent?

### Start with Built-in Agents

The built-in Build and Plan agents handle 90% of everyday needs. Before creating a custom agent, ask yourself:

**Can I accomplish this by:**
- Just asking Build directly? ✅
- Switching to Plan for analysis? ✅
- Using @explore or @general? ✅

If yes, don't create a custom agent yet. Custom agents are for when you find yourself repeatedly asking for the **same kind of specialized help**.

### Signs You Need a Custom Agent

Create a custom agent when:
- You're always saying "review this for security issues" → create a **security auditor** agent
- You frequently write documentation and want consistent style → create a **documentation writer** agent
- You need a specific workflow (like always running tests before committing) → create a **quality assurance** agent
- You want an agent with just the right tools and permissions for a repetitive task

**If it's a one-time need**, just ask directly. Custom agents are for patterns you see in your work.

### Creating Your First Custom Agent

The easiest way is to use the interactive wizard:

```bash
opencode agent create
```

This asks you:
1. **Where to save it**: Globally (available in all projects) or just for this project
2. **Name**: Short identifier like `security-checker` or `doc-writer` (no spaces, use hyphens)
3. **Description**: What it does in 10-20 words (e.g., "Looks for security vulnerabilities in code")
4. **Model**: Which AI to use (or accept the default)
5. **Tools**: Use arrow keys to select what actions it can perform

The wizard creates the configuration file in the right place with proper formatting. Start here - it prevents mistakes.

---

## Part 4: Configuration Files - Your Agent Instruction Manual

### What Is a Configuration File?

It's a **settings file** that tells OpenCode: "Here's what my agents are like and what they can do."

Think of it like a job description for each agent:
- Name: Security Checker
- Role: Find security problems
- Tools allowed: Can read anything, can't change anything
- Behavior: Be thorough, be critical, list issues by severity

### Where Configuration Lives

| Location | Who Can Use It | Create It With |
|----------|----------------|----------------|
| Project folder: `opencode.json` | Only this project | Edit manually or `opencode agent create` (project option) |
| Global: `~/.config/opencode/opencode.json` | All your projects | Edit manually or `opencode agent create` (global option) |
| Agent files: `.opencode/agents/` (project) or `~/.config/opencode/agents/` (global) | Depends on location | `opencode agent create` (creates these automatically) |

**Get started**: Use `opencode agent create` and it puts files in the right place automatically.

### Two Ways to Write Configurations

Both formats work equally well. Use whichever you prefer.

#### Option 1: JSON (Everything in One File)

```json
{
  "agent": {
    "my-agent": {
      "description": "Does something helpful",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.5,
      "tools": {
        "write": false,
        "edit": false,
        "bash": true
      }
    }
  }
}
```

**Pros**: Everything together, good for version control, easy to share  
**Cons**: Curly braces everywhere, harder to read

#### Option 2: Markdown (One File Per Agent)

Create a file like `.opencode/agents/security-checker.md`:

```markdown
---
description: Finds security vulnerabilities
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.1
tools:
  write: false
  edit: false
  bash: true
  read: true
---

You are a security expert auditing code for vulnerabilities.

Check for:
- SQL injection
- XSS vulnerabilities
- Hardcoded secrets
- Authentication issues

Rate findings by severity and provide fixes.
```

**Pros**: Human-readable, easy to edit, each agent in its own file, supports formatting  
**Cons**: More files to manage

**Which should you choose?**
- Just starting? Use `opencode agent create` - it picks the right format
- Making many custom agents? Markdown files keep them organized
- Sharing configs via Git? Both work, but some people prefer separate files
- Following online examples? They're usually JSON - you can convert later

**You can mix both**: OpenCode loads agents from all sources and combines them. Start with one format; you can always add the other later.

---

## Part 5: Configuration Options Explained

This section explains each setting you can put in an agent configuration. Start with the highlighted ones; the rest you can learn as needed.

### The Essential Options (Start Here)

#### Description (Required)
- **What it does**: A short explanation of what the agent does and when to use it
- **Length**: 10-20 words
- **Example**: "Reviews code for security vulnerabilities" or "Writes and maintains documentation"
- **Where you see it**: In agent selection lists and when auto-summoning subagents

**Good examples**:
- `code-reviewer` - "Reviews code for best practices and potential issues" ✓
- `doc-writer` - "Creates and updates project documentation" ✓
- `security-audit` - "Finds security vulnerabilities and suggests fixes" ✓

**Bad examples**:
- `agent1` - "Does stuff" ✗
- `myagent` - "An AI agent that does things" ✗

#### Mode (Important!)
- **What it does**: Determines how the agent can be used
- **Values**:
  - `"primary"`: Appears in Tab-switch rotation, can be your main conversational partner
  - `"subagent"`: Only available via @-mentions or automatic calls, NOT in Tab-switch
  - `"all"`: Can be both primary and subagent (maximum flexibility)
- **Rule of thumb**:
  - Use `primary` for agents you want to switch to and talk with directly
  - Use `subagent` for specialists you call only when needed (like @explore)
  - Most custom agents you create will be `subagent`

#### Model (Choose Your AI Brain)
- **What it does**: Selects which AI model powers this agent
- **Format**: `provider/model-id` examples:
  - `anthropic/claude-sonnet-4-5`
  - `openai/gpt-5`
  - `opencode/gpt-5.1-codex`
- **If you don't set it**: Agent uses whatever model you currently have selected globally (the one you see when you run `/models`)
- **Why you'd set it**: Different models have different speeds, costs, and capabilities. You might give your architect agent a smarter (but slower/expensive) model, while your quick research agent uses a fast/cheap model.

**Quick guide**:
- **Fast & cheap** (Haiku, GPT-5 Nano): Simple tasks, quick lookups, frequently-called subagents
- **Balanced** (Sonnet, GPT-5): Your everyday coding assistant
- **Smart & powerful** (Opus, GPT-5 Codex): Complex reasoning, architecture, critical code generation

**Don't overthink it**: Use the default model unless you have a specific reason. You can always change it later.

#### Prompt (Your Agent's Instructions)
- **What it does**: Defines the agent's personality, expertise, methodology, and constraints
- **Where**: In JSON, you reference a file: `"prompt": "{file:./prompts/build.txt}"`. In Markdown, the prompt is everything after the `---` section
- **What goes in it**:
  - Role definition: "You are a security auditor with 10 years of experience..."
  - Methodology: "Your process: 1) Check inputs, 2) Review auth, 3) Look for secrets..."
  - Output format: "Provide findings as: Severity: [rating], Location: [file:line], Issue: [description], Fix: [suggestion]"
  - Constraints: "Do not modify files. Only analyze."
  - Priorities: "Focus on security over code style"

**Why it matters**: The prompt is the single biggest factor in how well your agent performs. A well-written prompt gets better, more focused results.

**Bad prompt**: "You are a code reviewer." (Too vague)  
**Good prompt**: "You are a senior code reviewer. Check for: security vulnerabilities, performance issues, maintainability problems, missing error handling. For each issue, explain why it's a problem and provide a specific fix with code examples. Focus on critical issues first."

### The Most Important Settings

#### Temperature (The Creativity Dial)
- **What it does**: Controls how predictable vs. creative the agent is
- **Range**: 0.0 (very predictable) to 1.0 (very creative)
- **Analogy**: Think of it like a slider on a creativity dial

**Real-world examples**:

At **low temperature (0.1-0.3)**:
- You ask: "Name a good function for user login"
- Agent says: `login()`, `authenticateUser()`, `signIn()` - obvious, conventional names
- **Use for**: Code review, security checks, documentation, anything requiring accuracy and consistency

At **medium temperature (0.4-0.6)**:
- Same question
- Agent says: `verifyCredentials()`, `beginSession()`, `userAuthGate()` - reasonable but varied
- **Use for**: General coding help, generating options to choose from

At **high temperature (0.7-0.9)**:
- Same question
- Agent says: `portalToIdentity()`, `theGateKeeperAwakens()`, `grantMeAccess()` - wild, creative, sometimes too weird
- **Use for**: Brainstorming sessions, creative naming, exploring unusual approaches

**The rule of thumb**:
- Reviewing/analyzing → 0.1-0.3 (be consistent and reliable)
- Everyday coding → 0.4-0.6 (balanced)
- Brainstorming → 0.7-0.9 (get variety)

**Common mistake**: Using high temperature for code review → entertaining but unreliable feedback. Using low temperature for brainstorming → same ideas every time.

**Start with**: 0.5. Adjust based on results. Too repetitive? Go up. Too weird? Go down.

#### Tools (What the Agent Can Do)
- **What it does**: Enables or disables specific actions the agent can perform
- **Common tools**:
  - `write`: Create new files
  - `edit`: Modify existing files
  - `bash`: Run system commands (terminal operations)
  - `read`: Read file contents
  - `grep`: Search for patterns across files
  - `webfetch`: Get content from websites
- **Default**: Usually agents inherit global tool settings
- **How to think about it**: You're giving the agent access to different "rooms" in your house. A detective doesn't need to modify files, so you'd disable `write` and `edit`. A builder needs those tools enabled.

**Example configurations**:

**Code Reviewer** (looks but doesn't touch):
```json
"tools": {
  "write": false,
  "edit": false,
  "bash": true,
  "read": true,
  "grep": true
}
```

**Builder Agent** (your main coder):
```json
"tools": {
  "write": true,
  "edit": true,
  "bash": true,
  "read": true,
  "grep": true
}
```

**Explorer** (detective only):
```json
"tools": {
  "write": false,
  "edit": false,
  "bash": false,
  "read": true,
  "grep": true
}
```

**Safety tip**: Don't give agents tools they don't need. Fewer tools = more focused, less chance of accidental damage.

#### Permissions (Your Safety Controls)
- **What it does**: Controls whether the agent needs your approval before using a tool
- **Values**:
  - `"allow"`: Agent does it automatically (no prompt)
  - `"ask"`: Agent asks "Can I do this?" - you approve or deny
  - `"deny"`: Agent cannot use this tool at all

**Analogy**: Parental controls. Some actions are safe and don't need checking (`read` = `allow`). Some are risky and you want to review (`delete` = `ask`). Some are totally forbidden for this agent (`write` for a reviewer = `deny`).

**Example**:
```json
"permission": {
  "edit": "ask",
  "bash": {
    "*": "ask",
    "git status": "allow",
    "rm -rf": "deny"
  }
}
```

Translation:
- When agent wants to edit a file → asks you first ✓
- When agent wants to run most bash commands → asks you first ✓
- When agent wants to run `git status` → allowed automatically (harmless) ✓
- When agent tries `rm -rf` → denied completely (too dangerous) ✓

**Why this matters**: Permissions give you oversight. Use `ask` for operations that could affect your work. Use `allow` for safe, read-only actions. Use `deny` to completely forbid something.

**Built-in defaults**:
- Build: Most tools = `allow` (your main worker)
- Plan: Modification tools = `ask` (analysis mode, you want oversight)

### Other Useful Options

#### Steps (How Long Can It Work?)
- **What it does**: Maximum number of thinking/acting cycles before the agent must stop
- **Default**: Unlimited (within overall session limits)
- **When to use**: To control costs, ensure fast responses, or prevent infinite loops
- **Example**: `"steps": 10` → agent can think and act up to 10 times before giving final answer
- **Typical values**:
  - Simple tasks: 3-5 steps
  - Complex tasks: 10-20 steps
  - Research tasks: 20-50 steps
- **Leave it blank** for most uses - the agent knows when it's done

#### Color (Make It Pretty)
- **What it does**: Sets the agent's visual color in the interface
- **Values**: Hex codes (`"#FF5733"`) or theme names (`"primary"`, `"success"`, `"warning"`, `"error"`)
- **Why**: Helps you quickly identify which agent is active at a glance
- **Example**:
  - Security agent → red (`"#ff4444"`)
  - Documentation → green
  - Explorer → blue
  - Builder → yellow

#### Mode & Hidden (For Advanced Users)
- Already covered in essentials
- `hidden: true` hides subagents from the @-mention menu (they can only be called programmatically). Rarely needed.

### Options You Can Ignore (For Now)

- **top_p**: Alternative to temperature. Use temperature instead unless you know you need this.
- **Additional (pass-through) options**: Provider-specific settings. Only if you're using advanced model features.
- **Task permissions** (`permission.task`): Controls what subagents an agent can call. Advanced orchestration.

---

## Part 6: Complete Example - Building a Documentation Agent

Let's walk through creating a real custom agent from scratch.

### Step 1: Decide You Need It

**Scenario**: You maintain a project with lots of code. You frequently need to:
- Write README files
- Document APIs
- Add code comments explaining complex logic
- Update documentation when code changes

Instead of asking Build to do this every time (and giving inconsistent results), create a specialized **documentation agent**.

### Step 2: Run the Interactive Wizard

```bash
opencode agent create
```

Answer the prompts:

1. **Save globally or for this project?** → Project (for this example)
2. **Agent name?** → `doc-writer` (short, hyphenated, memorable)
3. **Description?** → "Creates and maintains project documentation" (what it does in ~10 words)
4. **Select a model?** → Accept default (whatever your current model is)
5. **Select tools**: Use spacebar to toggle:
   - ✓ write (needs to create docs)
   - ✓ edit (needs to update existing docs)
   - ✓ read (needs to read code to understand it)
   - ✗ bash (documentation doesn't need system commands)
   - ✓ grep (needs to search code)
6. **Finish** → Creates `.opencode/agents/doc-writer.md`

### Step 3: Customize the Prompt

The wizard creates a basic prompt. Edit `.opencode/agents/doc-writer.md`:

```markdown
---
description: Creates and maintains project documentation
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.4
tools:
  write: true
  edit: true
  read: true
  grep: true
  bash: false
permission:
  edit: ask
---

You are a technical writer specializing in clear, comprehensive documentation.

Your documentation should:
- Start with a clear purpose statement (what problem does this solve?)
- Use headings to organize content logically
- Include code examples with language hints
- Explain concepts in simple language (assume reader is competent but not expert)
- Follow consistent formatting
- Link to related resources

Documentation types you handle:
- README files (project overview, setup, usage examples)
- API documentation (endpoints, parameters, response formats)
- Developer guides (architecture, patterns, contribution guidelines)
- Code comments (explain why, not what)
- Inline documentation (JSDoc, docstrings)

When writing documentation:
1. First understand the code/feature by reading it
2. Identify the audience (developers, end users, administrators)
3. Structure from general to specific
4. Include practical, copy-pasteable examples
5. Review for clarity and completeness

When updating existing documentation:
- Preserve what's still relevant
- Flag outdated information clearly
- Maintain consistent voice and style
- Update cross-references and links

Output format:
- Use Markdown appropriately
- Code blocks with language identifiers: ```javascript\n...```
- Tables for options/parameters
- Include diagrams if helpful (describe in text)
```

### Step 4: Test Your Agent

In OpenCode, type:

```
@doc-writer write a README.md for this project explaining what it does and how to install it
```

Watch the agent work. It should read your codebase, understand the project, and produce a good README.

### Step 5: Refine

If the output isn't quite right, adjust the prompt:
- Too verbose? Change `temperature` from 0.4 to 0.3
- Missing something? Add it to the instructions
- Wrong tone? Rewrite the "Your documentation should" section

---

## Part 7: Examples Gallery - Ready-to-Use Agents

Copy these configurations to quickly add specialized agents to your toolkit.

### Code Reviewer
Analyzes code for quality, security, and best practices without making changes.

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
  read: true
  grep: true
permission:
  bash:
    "*": ask
    "git status": allow
    "git diff": allow
    "grep *": allow
---

You are a senior code reviewer. Examine code thoroughly and provide structured feedback.

Review checklist:
1. Security: injection, XSS, authentication, authorization, secrets in code
2. Error handling: missing try-catch, unhandled edge cases
3. Performance: inefficient loops, unnecessary computations, memory leaks
4. Code quality: naming, duplication, complexity, SOLID principles
5. Testing: missing tests, inadequate coverage, flaky tests
6. Maintainability: tightly coupled, magic numbers, unclear logic

For each issue:
- Severity: Critical / High / Medium / Low
- Location: file:line
- Problem: clear explanation
- Suggestion: specific fix with code example when helpful

Provide a summary with:
- Total issues by severity
- Positive observations (what was done well)
- Top 3 priorities to address

Be direct and specific. Don't just say "this is bad" - explain why and show better.
```

**Use it**: `@code-reviewer review the changes in src/auth/` or `@code-reviewer check for security issues in the payment module`

---

### Security Auditor
Finds security vulnerabilities and provides remediation steps.

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

You are a cybersecurity expert conducting a comprehensive security audit.

Audit categories:
1. Input validation: injection (SQL, command, XSS, path traversal), sanitization
2. Authentication: password handling, session management, MFA, credential storage
3. Authorization: privilege escalation, access controls, IDOR
4. Data protection: PII handling, encryption at rest/in transit, secrets management
5. Dependencies: known vulnerabilities in package manifests, outdated versions
6. Configuration: exposed ports, hardcoded credentials, debug mode in prod
7. Cryptography: weak algorithms, improper key management, random number generation
8. Logging: sensitive data in logs, insufficient logging for forensics

For each finding:
- Severity: Critical / High / Medium / Low / Info
- CWE/OWASP reference when applicable
- Description: what is the vulnerability
- Impact: what could happen
- Location: file:line or component
- Remediation: step-by-step fix with code examples

Structure output as:
## Executive Summary
- Overall risk rating
- Key statistics
- Immediate actions required

## Findings by Category
For each category, list all findings with full details

## Positive Observations
Secure practices you found (helps reinforce good behavior)

## Remediation Roadmap
Prioritized by severity with effort estimates
```

**Use it**: `@security-auditor perform a full security audit of the authentication system` or `@security-auditor check for secrets in recent commits`

---

### Debugging Assistant
Investigates bugs, errors, and performance issues.

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
        "read": true,
        "grep": true
      },
      "permission": {
        "bash": {
          "*": "ask",
          "tail *": "allow",
          "cat *": "allow",
          "ps aux": "allow",
          "netstat": "allow",
          "lsof": "allow"
        }
      },
      "prompt": "You are a debugging specialist. Systematic approach:\n\n1. Gather information: error messages, stack traces, symptoms\n2. Check logs: application logs, system logs, error outputs\n3. Reproduce: steps to trigger the issue\n4. Isolate: narrow down to specific component/function\n5. Trace: follow execution flow, watch variable states\n6. Hypothesize: what could cause this\n7. Test hypothesis: validate or refute\n8. Identify root cause\n9. Suggest fix with rationale\n10. Recommend test case to prevent regression\n\nWhen investigating:\n- Look for recent changes (git blame, git log)\n- Check for resource exhaustion (memory, file handles, connections)\n- Consider timing issues, race conditions\n- Check for missing null checks, boundary conditions\n- Review error handling paths\n\nOutput format:\n- Summary of issue\n- Investigation steps taken\n- Root cause identified\n- Recommended fix\n- Test case to verify\n- Preventive measures for future"
    }
  }
}
```

**Use it**: `@debugger why am I getting 'undefined is not a function' in main.js?` or `@debugger the server is slow, help me find the bottleneck`

---

### Architecture Planner
Designs system architecture and plans implementation (read-only).

```markdown
---
description: Designs system architecture and plans implementation
mode: primary
model: anthropic/claude-opus-4
temperature: 0.3
tools:
  write: false
  edit: false
  bash: false
  read: true
  grep: true
permission:
  edit: "deny"
  bash: "deny"
---

You are a system architect designing robust, scalable systems.

Your planning process:
1. Requirements analysis: functional requirements, non-functional (performance, security, reliability)
2. Constraints identification: budget, timeline, team skills, existing infrastructure
3. Consider trade-offs: simplicity vs flexibility, performance vs maintainability, time-to-market vs robustness
4. Choose patterns and technologies: justify each choice
5. Document architecture with diagrams (described in text) and explanations
6. Create implementation roadmap with milestones
7. Identify risks and mitigation strategies

Artifacts you produce:
- System component diagram (describe components and relationships)
- Data flow diagram (how data moves through the system)
- API specifications (endpoints, request/response formats, auth)
- Database schema (tables, relationships, indexes)
- Technology stack recommendations (with alternatives)
- Implementation phases (what to build first, what can wait)
- Risk assessment (what could go wrong and how to prevent it)

Consider:
- Scalability boundaries and capacity planning
- Failure modes and resilience patterns (circuit breakers, retries, fallbacks)
- Security implications at each layer
- Operational concerns: monitoring, logging, deployment, backup/restore
- Team skill alignment and learning curve
- Cost implications (infrastructure, maintenance)

Provide specific, actionable recommendations with clear rationale. Acknowledge trade-offs rather than presenting a single "perfect" solution.
```

**Use it**: Switch to this primary agent and ask: "Design a microservice architecture for an e-commerce platform with 100K users" or "Plan the refactoring of this monolithic app to separate services."

---

### Research Specialist
Conducts broad research and synthesizes information.

```markdown
---
description: Conducts broad research and synthesizes information
mode: subagent
model: openai/gpt-5
temperature: 0.6
tools:
  write: false
  edit: false
  read: true
  grep: true
  webfetch: true
permission:
  webfetch: "ask"
---

You are a research specialist who excels at gathering and synthesizing information from diverse sources.

Your approach:
1. Clarify the research question: identify sub-questions, scope, and intended audience
2. Identify information sources: codebase locations, documentation sites, best practice articles, specification documents
3. Systematically explore each source: search code, read files, fetch authoritative web resources
4. Take structured notes: organize by theme, question, or architecture layer
5. Synthesize: connect dots between sources, identify consensus and contradictions
6. Identify gaps: what remains uncertain or requires additional investigation
7. Present findings: structured by priority, with clear citations and confidence levels

When researching codebases:
- Map relevant components and their relationships
- Trace data flows and dependencies
- Find patterns, conventions, and anti-patterns
- Extract configurations, constants, and business logic
- Understand the "why" behind architectural decisions

When using web fetch:
- Prefer authoritative sources (official documentation, RFCs, well-known blogs)
- Verify information across multiple sources when possible
- Extract only relevant information, not entire articles
- Note the source and date (information freshness matters)
- Be skeptical of outdated content

Organize findings:
- Hierarchical structure (main topics → subtopics → details)
- Highlight key insights and actionable recommendations
- Distinguish facts from opinions
- Include code examples when helpful
- Provide a summary for executives and detailed sections for implementers

Be thorough but concise. Prefer synthesis over raw dump.
```

**Use it**: `@general research the best practices for handling file uploads in web applications` or `@general what authentication methods does this codebase currently use?`

---

### Quick Checklist Agent
For running standard checks (linting, tests, formatting).

```markdown
---
description: Runs project quality checks (lint, test, format)
mode: subagent
model: openai/gpt-5
temperature: 0.2
tools:
  write: false
  edit: false
  bash: true
  read: true
permission:
  bash:
    "*": "ask"
    "npm test": "allow"
    "npm run lint": "allow"
    "npm run format": "allow"
    "git status": "allow"
---

You run quality assurance commands to ensure code health.

Standard check sequence:
1. Linting: catch style issues, potential bugs, code smells
2. Tests: verify existing functionality works
3. Type checking: ensure type safety (if TypeScript)
4. Format: apply consistent code formatting
5. Security scan: check for vulnerabilities (if configured)

Commands for common ecosystems:

JavaScript/TypeScript:
- npm run lint (or yarn lint)
- npm test
- npm run type-check (if TypeScript)
- npm run format

Python:
- flake8 or pylint
- pytest or tox
- black --check (see if formatting needed)

Go:
- go vet
- go test
- go fmt -l (list unformatted files)

After running checks:
- Summarize results: passed/failed, warnings, errors
- If failures: explain what they mean and how to fix
- If all pass: "All quality checks passed ✅"
- Suggest next steps (e.g., "Now you can commit" or "Fix the 3 lint errors before continuing")

Note: You only RUN checks, don't fix automated fixes unless explicitly asked.
```

**Use it**: `@quality-check run all the quality checks for this project` or `@quality-check should I commit these changes?`

---

## Part 8: Advanced Topics (For When You're Ready)

### Configuration Precedence (Which Setting Wins?)

When you define an agent in multiple places, OpenCode merges them. Later sources override earlier ones for the same agent.

**Priority order (from lowest to highest)**:
1. Built-in defaults (OpenCode's built-in agents)
2. Global config: `~/.config/opencode/opencode.json`
3. Global agents: `~/.config/opencode/agents/*.md`
4. Custom config (OPENCODE_CONFIG env var)
5. Project config: `opencode.json` in project root
6. Project agents: `.opencode/agents/*.md`

**Example**: You have a global Plan agent and a project-specific Plan agent with different temperature settings. The project setting wins - the Plan agent in that project uses the project temperature.

**How to see the final result**: Run `opencode config show` - it displays the fully merged configuration.

### Configuration Merging Example

You have:

**Global** (`~/.config/opencode/opencode.json`):
```json
{
  "agent": {
    "plan": {
      "model": "anthropic/claude-haiku-4-5",
      "temperature": 0.1
    }
  }
}
```

**Project** (`opencode.json`):
```json
{
  "agent": {
    "plan": {
      "temperature": 0.2
    }
  }
}
```

**Project agent file** (`.opencode/agents/plan.md`):
```markdown
---
mode: primary
---

You are a thoughtful planning assistant...
```

**Resulting Plan agent**:
- Model: `anthropic/claude-haiku-4-5` (from global, not overridden)
- Temperature: `0.2` (from project override)
- Prompt: "You are a thoughtful planning assistant..." (from project agent file)
- Mode: `primary` (from project agent file)

Don't stress about this. Start simple (use one location), and use `opencode config show` when you need to debug.

### Provider and Model Integration

Agents use models from your configured providers. Before using a model like `anthropic/claude-sonnet-4-5`, you must connect the Anthropic provider:

```bash
# In OpenCode, run:
/connect
# Follow prompts to add your API key
```

The provider is the service (Anthropic, OpenAI, etc.). The model is the specific AI version. The agent configuration references both as `provider/model-id`.

If you specify a model from a provider you haven't connected, the agent won't load and you'll see an error.

### Custom Providers (For Local Models or Internal APIs)

If you have a local model (Ollama, LM Studio) or company AI API, create a custom provider:

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

This is advanced. Stick with built-in providers (Anthropic, OpenAI) until you need something custom.

### Task Permissions (Advanced Orchestration)

`permission.task` controls which subagents an agent can invoke via the Task tool. It's about **what other agents this agent can call**.

Example:
```json
"permission": {
  "task": {
    "*": "deny",
    "orchestrator-*": "allow",
    "code-reviewer": "ask"
  }
}
```

Translation:
- Cannot invoke any subagent by default
- Can invoke subagents starting with `orchestrator-` without asking
- Can invoke `code-reviewer` but needs user approval

**Why use this?** To create agent hierarchies. A management agent might only call your custom, vetted agents - not arbitrary ones users @-mention.

**Important**: This doesn't restrict what *users* can do. Users can always @-mention any agent. Task permissions only restrict what the *agent* does autonomously.

---

## Part 9: Troubleshooting - Plain English

### "I created an agent but can't find it"

**Symptoms**: New agent doesn't appear in Tab-switch list or @-mention suggestions.

**Most likely causes (check in order)**:

1. **Did you restart OpenCode?** OpenCode loads configs at startup. Restart it.
2. **Is the agent mode set correctly?** Subagents don't appear in Tab-switch. Use `mode: "primary"` or `mode: "all"` to make it switchable.
3. **Is the file in the right place?**
   - Markdown agents: `.opencode/agents/your-agent.md` (project) or `~/.config/opencode/agents/` (global)
   - JSON agents: `opencode.json` with agent under `"agent"` key
4. **Syntax error?** YAML frontmatter indentation matters. JSON missing commas? Validate.
5. **Is `disable: true` set?** Remove it or set to `false`.

**Debug**: Run `opencode config show` to see all loaded agents. Is yours listed?

---

### "My agent is using the wrong AI model"

**Symptoms**: Agent uses a different model than you configured.

**Causes**:

1. **Model not available**: Check spelling and provider connection. Format must be `provider/model-id`. Did you connect the provider via `/connect`?
2. **Config not reloading**: Restart OpenCode.
3. **Wrong field**: In JSON, it's `"model"` (not `"models"` or `"aiModel"`).
4. **Another config overriding it**: Use `opencode config show` to see the effective configuration.

---

### "Agent can't use a tool I enabled"

**Symptoms**: Agent fails to use `write`, `edit`, `bash`, or other tools.

**Causes**:

1. **Tool disabled at agent level**: Check agent's `tools` settings. Agent-level overrides global settings.
2. **Permission is `deny`**: Tool may be enabled but denied by `permission`.
3. **Provider limitation**: Some models don't support certain tools. Check model documentation.
4. **MCP server offline**: If tool comes from an MCP server, is it running?

**Check**: In your agent config, ensure:
```json
"tools": {
  "write": true
},
"permission": {
  "write": "allow"  // or "ask", but not "deny"
}
```

---

### "Agent responses are too weird/unpredictable"

**Symptoms**: Agent gives strange answers, goes off-topic, or is inconsistent.

**Fix**: Lower the `temperature`. If it's 0.8, try 0.4. Temperature above 0.6 can cause randomness. Most tasks need 0.1-0.5.

---

### "Agent keeps calling subagents I don't want"

**Symptoms**: Primary agent automatically invokes subagents for simple tasks.

**Cause**: Subagent descriptions are too broad, making them match many tasks.

**Fixes**:
1. Make subagent descriptions more specific
2. Disable the subagent if you never want it used
3. Use @-mentions to explicitly call agents instead of relying on auto-selection
4. There's no setting to disable auto-invocation - adjust descriptions or remove the subagent

---

### "Changes to my config file don't take effect"

**Symptoms**: You edited `opencode.json` or created a new agent file, but OpenCode still uses old settings.

**Cause**: OpenCode reads config at startup.

**Fix**: Restart OpenCode completely.

---

### "How do I know what's actually configured?"

**Command**: `opencode config show`

This displays the fully merged configuration after all files are loaded and merged. Use it to see what settings are actually in effect.

---

### "Agent behaves unexpectedly"

**Checklist**:

1. **Temperature too high?** Lower it (use 0.3-0.5 range)
2. **Prompt unclear?** Rewrite with more specific instructions
3. **Wrong model?** Try a different one (Haiku for speed, Sonnet for balance, Opus for complexity)
4. **Too many/few steps?** Adjust `steps` if agent stops too early or goes on too long
5. **Tools inappropriate?** Review `tools` and `permission`

---

## Part 10: Quick Reference

### Agent Decision Cheat Sheet

| I want to... | Use this agent | How to invoke |
|--------------|----------------|---------------|
| Write/change code | Build | Default, just ask |
| Review/analyze code | Plan | Press Tab → select plan |
| Explore codebase structure | Explore | `@explore your question` |
| Research best practices | General | `@general your question` |
| Get session summary | Summary (hidden) | `/summary` |
| Create documentation | doc-writer (custom) | `@doc-writer ...` |
| Find security issues | security-auditor (custom) | `@security-auditor ...` |

### Temperature Quick Reference

| Task Type | Temperature | Why |
|-----------|-------------|-----|
| Code review, security, documentation | 0.1-0.3 | Need consistency, not creativity |
| General coding help | 0.4-0.6 | Balanced |
| Brainstorming, creative tasks | 0.7-0.9 | Want variety |

### Tools Quick Reference

| Agent Type | Typical Tools | Typical Permissions |
|------------|---------------|---------------------|
| Code Reviewer | write: false, edit: false, bash: true, read: true, grep: true | bash: ask (except git status: allow) |
| Builder | all tools true | Most: allow, destructive: ask |
| Explorer | write: false, edit: false, bash: false, read: true, grep: true | all: allow (it's read-only) |
| Researcher | write: false, edit: false, read: true, grep: true, webfetch: true | webfetch: ask |

### Configuration File Locations

| Scope | Config file | Agent files |
|-------|-------------|-------------|
| Project-specific | `opencode.json` | `.opencode/agents/*.md` |
| Global (all projects) | `~/.config/opencode/opencode.json` | `~/.config/opencode/agents/*.md` |

### Common Commands

- `opencode agent create` - Interactive agent creation wizard
- `opencode config show` - See your effective configuration
- `opencode config validate` - Check for config errors
- `/models` - See available AI models
- `/connect` - Add provider credentials
- Tab - Switch between primary agents (default keybind)
- `@agent-name` - Manually summon a subagent

### Minimal Agent Config (Copy-Paste Template)

```markdown
---
description: What this agent does
mode: subagent  # or "primary"
model: anthropic/claude-sonnet-4-5  # optional, uses default if omitted
temperature: 0.5  # optional, default varies
tools:
  write: false
  edit: false
  bash: false
  read: true
  grep: true
permission:
  edit: "ask"  # or "allow" or "deny"
---

Your prompt goes here. Detailed instructions for the agent.
```

---

## Conclusion

OpenCode agents let you build a team of specialized AI helpers. Start with the built-in Build and Plan agents - they handle most tasks. Switch between them with Tab. Call specialists like `@explore` when needed.

Create custom agents when you repeatedly need the same specialized help. Use `opencode agent create` to get started. Focus on three settings: `temperature` (creativity), `tools` (what it can do), and `permission` (when it needs approval).

Don't overcomplicate. Start simple. Add agents as you discover needs. The configuration system is flexible - you can always refine later.

Happy coding!

---

## Appendix

### Full Built-in Agent Summary

| Agent | Mode | Purpose | When to Use |
|-------|------|---------|-------------|
| build | primary | Main coding assistant | Writing code, making changes, debugging |
| plan | primary | Analysis without changes | Code review, planning, documentation |
| general | subagent | Deep research | Complex questions requiring investigation |
| explore | subagent | Codebase exploration | Finding files, understanding structure |
| compaction | hidden | Auto-summarizes old chat | Invisible, automatic |
| title | hidden | Generates session titles | Invisible, automatic |
| summary | hidden | Creates session summaries | `/summary` command |

### Glossary (Alphabetical)

**@-mention** - Typing `@agent-name` to explicitly call a subagent  
**Agent** - A specialized AI helper with specific capabilities and instructions  
**API** - Application Programming Interface; how software components communicate  
**Bash** - Tool that allows agents to run system commands  
**Configuration** - Settings file that defines how agents behave  
**Edit** - Tool that allows agents to modify existing files  
**Grep** - Tool that searches for patterns across files  
**JSON** - Data format for configuration files (curly braces, key-value pairs)  
**Markdown** - Simple text format with frontmatter for configuration (preferred for agents)  
**Model** - The specific AI engine (Claude Sonnet, GPT-5, etc.)  
**Permission** - Controls whether agent actions require user approval (`allow`/`ask`/`deny`)  
**Primary agent** - An agent you can switch to and talk to directly  
**Prompt** - Instructions that define agent behavior and expertise  
**Provider** - The service supplying AI models (Anthropic, OpenAI, etc.)  
**Read** - Tool that allows agents to view file contents  
**Subagent** - A specialist agent called via @-mention for specific tasks  
**Temperature** - 0-1 setting controlling AI creativity/predictability  
**Tools** - Capabilities an agent can use (write, edit, bash, read, grep, webfetch)  
**Webfetch** - Tool that retrieves content from websites  
**Write** - Tool that allows agents to create new files

### Further Reading (Links to Official Docs)

- Configuration - Full schema and all options
- Providers - Setting up AI service connections
- Models - Understanding different AI models
- Permissions - Detailed permission system
- Tools - Complete tool reference
- Troubleshooting - General OpenCode help

---

**Document version**: 1.0 (Beginner-friendly rewrite)  
**Last updated**: 2025-03-14  
**Intended audience**: Non-technical users, OpenCode beginners, developers new to agent customization
