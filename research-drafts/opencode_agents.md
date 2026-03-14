# OpenCode Agents: Your Complete Guide to Getting Started

---

## What Is OpenCode? (In Plain English)

Let's start with the basics. OpenCode is a coding assistant that lives inside your text editor. Think of it like having a knowledgeable programming partner sitting next to you, ready to help with whatever coding task you're working on.

**But instead of being just one person, you can build a whole team of specialized helpers.**

Imagine you're renovating a house:
- You need an architect to plan the work
- You need electricians and plumbers for specialized tasks  
- You need a general contractor who actually builds
- You need an inspector to check the work

OpenCode agents are like that team. Each agent has a specific role, expertise, and set of tools they can use. This guide will help you understand how to use the agents that come with OpenCode, and how to create your own custom agents when you need specialized help.

**The key insight**: One-size-fits-all AI help is okay, but having a team of specialists is much more powerful for getting consistent, high-quality results.

---

## Your First 10 Minutes with OpenCode Agents

Before reading anything else, open OpenCode and try these:

1. Press the **Tab** key (this cycles through your available agents). You should see names like `build` and `plan` appear.
2. Without switching, type: `Create a simple Python script that says "Hello World" and saves it to hello.py`
3. Watch what happens. The `build` agent should create the file for you.
4. Now press **Tab** until you see `plan` in the interface.
5. Type: `Review the hello.py file we just created and suggest any improvements`
6. Press Tab again to go back to `build`.
7. Type: `Make the changes that Plan suggested`

That's the core workflow. You now understand 80% of how agents work:
- **Build** does the work
- **Plan** reviews the work
- You switch between them with Tab

Don't worry about everything else in this document yet. Come back to it when you've done those steps and want to learn more.

---

## Part 1: Understanding the Built-In Agent Team

OpenCode gives you seven agents right out of the box. You'll use two constantly, two occasionally, and three work automatically in the background.

### The Two You Use Every Day

#### Build Agent - Your Main Builder

The Build agent is your default helper. It writes code, edits files, creates new things, and makes changes to your project.

**What it does**: Actually creates and modifies code
**When to use it**: Everyday coding tasks - new features, bug fixes, refactoring, adding functionality
**Can it**: Write files, edit files, run commands, search code, fetch web content - basically everything
**How to talk to it**: Just ask. It's the default. Or press Tab until you see `build`.

*Example*: "Add user authentication to this app" or "Extract this duplicated code into a reusable function"

#### Plan Agent - Your Quality Control

The Plan agent analyzes and reviews but doesn't make changes unless you specifically ask it to. It's your safety net.

**What it does**: Reviews, analyzes, plans - but doesn't modify
**When to use it**: Code review, planning features, analyzing problems, understanding code
**Can it**: Read files and analyze them. Modification tools require your approval
**How to talk to it**: Press Tab until you see `plan`.

*Example*: "Review my recent changes for security issues" or "What's wrong with this function?"

**Smart workflow**: Start with Build to create something, switch to Plan (Tab) to review it, switch back to Build to implement the improvements. This back-and-forth is the heart of effective agent use.

### The Two You Call Occasionally

These aren't in the Tab rotation. You call them by typing `@name` in your messages.

#### Explore Agent - Your Detective

Explore searches through your codebase to find things and understand structure.

**What it does**: Finds files, maps dependencies, understands how components relate
**When to use it**: "Where is the payment code?", "What files handle user authentication?", "How is the database structured?"
**Perfect for**: New codebases, finding where something is defined, understanding code organization
**How to call it**: `@explore find all files related to user management`

#### General Agent - Your Researcher

General conducts deeper research requiring multiple steps and information gathering.

**What it does**: Multi-step research, synthesizing information from multiple places, answering complex questions
**When to use it**: "What are best practices for API security?", "How do other teams in our codebase handle authentication?", "Research database connection pooling strategies"
**How to call it**: `@general what's the standard approach for file upload validation here?`

---

## Part 2: How Agent Communication Actually Works

### Switching vs. Calling: Two Different Patterns

**Switching (Tab key)**: You change which agent you're talking to directly. The conversation continues. It's like walking over to a different colleague's desk and having a conversation with them.

**Calling (@-mention)**: You ask your current agent to get help from a specialist. It's like saying "Hey Build, I need you to ask Explore to find something for us."

**When to use which?**

| Situation | Best approach | Why |
|-----------|---------------|-----|
| You want a second opinion on Build's code | Switch to Plan | Direct conversation with reviewer |
| You need Build to create something | Stay with Build | It's the builder |
| You need to find where something is in the codebase | Call @explore | Explorer finds things faster |
| You need deep research on a topic | Call @general | General does thorough investigation |
| You're planning architecture | Switch to Plan or create architecture agent | Analytical thinking needed |

### What Happens When You Call a Subagent

1. You type `@explore find all database files`
2. Explore gets that as its task and works independently
3. You might see Explore create its own mini-conversation (a "subtask")
4. Explore finishes and returns results to your main agent
5. Your main agent uses those results to answer you

**You can watch subtasks in action**: If a subagent creates its own thread, use keyboard shortcuts to jump in and see what it's doing. Default: `Ctrl+H` (or your configured key) to enter child sessions, arrow keys to navigate between them. This is useful if a subagent seems stuck or you want to guide it.

### Automatic Subagent Calling

Sometimes you don't need to manually call subagents. If you ask Build "where is the login code?", Build might automatically call Explore because finding things is Explore's specialty. You'll see a message indicating this happened.

**Don't fight this** - let the system do its job. But if you want more control, manually specify `@explore` to ensure the right agent handles the task.

---

## Part 3: Should You Create a Custom Agent?

### Start With What You Have

**Important**: You don't need custom agents to use OpenCode effectively. The built-in Build and Plan agents handle 90% of everyday coding tasks. Use `@explore` and `@general` for the rest.

Create a custom agent only when you notice a pattern in your work: you keep asking for the same type of specialized help and want consistent, focused results.

### Signs You Actually Need a Custom Agent

Create a custom agent when you find yourself repeatedly doing these:

- "Always review for security issues" → **Security Auditor**
- "Write documentation in a specific format" → **Documentation Writer**
- "Run these quality checks before committing" → **Quality Gatekeeper**
- "Check if my code follows our team's style guide" → **Style Enforcer**
- "Optimize this for performance" → **Performance Tuner**

**If it's a one-time question**, just ask directly. Custom agents are for recurring needs where you want consistency.

---

## Part 4: Creating Your First Custom Agent

### The Easy Way: Use the Wizard

OpenCode has an interactive wizard that holds your hand through agent creation. Here's how:

```
opencode agent create
```

The wizard asks you five questions:

1. **Save location** - "Globally (all projects) or just for this project?"
   - Choose "project" while learning. You can always change later.
   
2. **Agent name** - Short identifier, no spaces. Use hyphens: `security-checker`, `doc-writer`, `test-runner`
   - Keep it descriptive and short
   
3. **Description** - What it does in 10-20 words
   - Example: "Reviews code for security vulnerabilities" (good)
   - Example: "Does security stuff" (bad - too vague)
   
4. **Choose a model** - Which AI powers it? Accept the default unless you have a reason.
   - You can change this later in the config file
   
5. **Select tools** - What actions can it take? Use spacebar to toggle:
   - `write` - create files
   - `edit` - modify files
   - `bash` - run terminal commands
   - `read` - read files
   - `grep` - search patterns
   - `webfetch` - get web content

**Safety principle**: Give agents only the tools they absolutely need. A reviewer doesn't need `write` or `edit`. A documentation writer probably doesn't need `bash`. Fewer tools = more focused = safer.

The wizard creates the configuration file in the right location automatically. Start here - it prevents formatting mistakes.

---

## Part 5: Understanding Agent Configuration Files

### What Is a Configuration File?

It's a settings file that tells OpenCode: "Here's what this agent is like and what it can do."

Think of it like filling out a job description:
- Name: Security Checker
- Job: Find security problems in code
- Tools: Can read anything, but can't modify files
- Behavior: Be thorough, rate findings by severity, always suggest fixes
- Working style: Use temperature 0.1 (very consistent, not creative)

### Where Configuration Files Live

| Location | Available To | Create With |
|----------|--------------|-------------|
| Project folder: `.opencode/agents/your-agent.md` | Only this project | `opencode agent create` (project option) |
| Global folder: `~/.config/opencode/agents/your-agent.md` | All your projects | `opencode agent create` (global option) |
| Project file: `opencode.json` (in project root) | Only this project | Edit manually or wizard |
| Global file: `~/.config/opencode/opencode.json` | All your projects | Edit manually or wizard |

**Recommendation**: Use the wizard (`opencode agent create`). It creates files in the right place with proper formatting. If you need to edit later, you'll know where to look.

### Two Ways to Write Configurations

Both formats work equally well. The wizard gives you both options. Here's how they differ:

#### Option 1: JSON (Everything in One File)

In this format, you put all your agent definitions in a single `opencode.json` file using JSON format (curly braces, quotes, commas).

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

**When to use JSON**:
- You like having everything in one place
- You're sharing configs via Git and want version control on a single file
- You're copying examples from documentation (many examples use JSON)
- You programmatically generate agent configs

**Downsides**: Harder to read, curly braces everywhere, one syntax error breaks everything.

#### Option 2: Markdown (One File Per Agent)

In this format, each agent gets its own `.md` file in the `.opencode/agents/` folder. The file starts with YAML frontmatter (the part between `---` lines), then your prompt instructions.

Create `.opencode/agents/security-checker.md`:

```markdown
---
description: Finds security vulnerabilities in code
mode: subagent
model: anthropic/claude-sonnet-4-5
temperature: 0.1
tools:
  write: false
  edit: false
  bash: true
  read: true
  grep: true
---

You are a cybersecurity expert auditing code for vulnerabilities.

Your audit covers:
- SQL injection
- XSS vulnerabilities
- Hardcoded secrets
- Authentication issues

For each finding:
1. State severity: Critical, High, Medium, Low, or Info
2. Show location: file and line number
3. Explain the risk
4. Provide a specific fix

Be thorough but concise.
```

**When to use Markdown**:
- You want human-readable files
- You have many custom agents and want them organized separately
- You want to write detailed prompts with formatting (headings, lists, etc.)
- You prefer editing agent instructions in a natural way

**Downsides**: More files to manage, though that rarely becomes a problem.

**You can mix both**: OpenCode loads agents from all sources and combines them. Start with one format (the wizard helps you choose). Switch later if needed.

---

## Part 6: Agent Configuration Settings Explained

This section explains each setting you can use. Start with the highlighted essentials; learn the rest as you need them.

### Essential Settings (Use These First)

#### description - What Your Agent Does

**Required**: Every agent needs this.
**Length**: 10-20 words is ideal
**Example good**: "Reviews code for security vulnerabilities and suggests fixes"
**Example bad**: "A code agent that does security things"

**Where you see it**: In agent selection lists, when agents auto-summon subagents, in `/agents` command output.

Make it clear and descriptive. Future-you will thank you when you have 10 custom agents and need to remember what each does.

#### mode - How Your Agent Is Used

**Critical setting** with three options:

- `"primary"` - Appears in Tab rotation. You switch to it and talk directly. Use for agents you want as your main partners.
- `"subagent"` - Only available via @-mentions or automatic calls. NOT in Tab rotation. Use for specialists you call when needed (like @explore).
- `"all"` - Can be both primary and subagent. Maximum flexibility.

**Rule of thumb**:
- Build and Plan should be `primary` (you switch to them)
- Specialists like security-checker should be `subagent` (you call them when needed)
- Most custom agents you create will be `subagent`

#### model - Which AI Powers Your Agent

**Format**: `provider/model-name`
**Examples**: 
- `anthropic/claude-sonnet-4-5`
- `openai/gpt-5`
- `opencode/gpt-5.1-codex`

If you don't set this, the agent uses whatever model you have selected globally (see `/models` command).

**Why you'd set it per-agent**: Different models for different purposes:
- **Fast & cheap** (Haiku, GPT-5 Nano): Quick lookups, frequent subagents
- **Balanced** (Sonnet, GPT-5): Everyday coding (default for most)
- **Smart & expensive** (Opus, GPT-5 Codex): Complex architecture, critical code

**Don't overthink**: Use the default model. Change only if you have a specific reason (cost, speed, or capability needs). You can adjust per-agent later.

#### prompt - Your Agent's Instructions

**This is the most important setting**. The prompt defines your agent's expertise, personality, methodology, and constraints. A great prompt creates a great agent. A vague prompt creates confusion.

**Prompt goes where**:
- In JSON: `"prompt": "{file:./prompts/build.txt}"` (reference to separate file) OR inline: `"prompt": "You are a builder..."`
- In Markdown: Everything after the `---` frontmatter section

**What makes a good prompt**:

1. **Clear role**: "You are a senior security auditor with 10 years of experience" (good) vs "You review code" (bad)

2. **Specific methodology**: "Your process: 1) Check input validation, 2) Review authentication, 3) Look for secrets, 4) Rate findings by severity" (good)

3. **Output format**: "For each issue: Severity: [rating], Location: [file:line], Problem: [description], Fix: [code example]" (good) vs "find issues" (bad)

4. **Constraints**: "Do not modify files. Only analyze and report." (important for reviewers)

**Bad prompt example**:
```
You are a code reviewer. Check code and give feedback.
```

**Good prompt example**:
```
You are a senior code reviewer with expertise in security and performance.

Your review covers:
1. Security: injection attacks, XSS, authentication flaws, secrets in code
2. Error handling: missing try-catch, unhandled edge cases
3. Performance: inefficient loops, unnecessary computations
4. Code quality: naming, duplication, complexity, maintainability

For each issue found:
- Severity (Critical/High/Medium/Low)
- Exact location (file:line)
- Clear explanation of the problem
- Specific fix with code example

Output format:
## Summary (overall assessment, 2-3 sentences)

## Issues Found
List each issue with severity, location, explanation, and fix.

## Positive Observations
What was done well (encourage good practices)

Be direct and specific. Don't just say "this is bad" - explain why and show better.
```

**Why prompt matters more than model**: A well-crafted prompt on a cheaper model often outperforms a vague prompt on an expensive model. Invest time in your prompt.

---

### Important Settings (Get These Right)

#### temperature - The Creativity Dial

**Range**: 0.0 (very predictable, consistent) to 1.0 (very creative, unpredictable)

This is the single most misunderstood setting. Here's what it actually does:

**Temperature 0.1-0.3 (Low - Consistent and Reliable)**:
- Output is predictable, follows conventions, stays on track
- Perfect for: code review, security scanning, documentation, anything requiring accuracy
- Same prompt gives similar results every time
- Example: For "good function name for login", you get conventional names: `login()`, `authenticate()`, `signIn()`

**Temperature 0.4-0.6 (Medium - Balanced)**:
- Some variation, but still reasonable
- Perfect for: everyday coding help, generating options to choose from
- Good default for builders
- Example: Same prompt gives: `verifyCredentials()`, `beginSession()`, `startLoginFlow()`

**Temperature 0.7-0.9 (High - Creative/Experimental)**:
- Wild, varied, sometimes too weird, goes off-topic
- Perfect for: brainstorming, creative naming, exploring unusual approaches
- Not suitable for production work or analysis
- Example: Same prompt gives: `portalToIdentity()`, `theGateKeeperAwakens()`, `grantMeAccess()`

**Common mistakes**:
- Using high temperature (0.8) for code review → entertaining but unreliable
- Using low temperature (0.2) for brainstorming → same ideas every time

**Start with 0.5**. Adjust based on results:
- Too repetitive? Increase temperature slightly
- Too weird or off-topic? Decrease temperature
- For analysis/review tasks: 0.1-0.3
- For coding assistance: 0.4-0.6
- For creative work: 0.7-0.9

#### tools - What Actions the Agent Can Take

**Simple on/off switches for each capability**:

- `write` - Create new files
- `edit` - Modify existing files
- `read` - Read file contents
- `grep` - Search for patterns across files (like a smart find)
- `bash` - Run system commands (terminal operations)
- `webfetch` - Retrieve content from websites

**Default**: If you don't specify tools, agents inherit global settings. But you should explicitly set tools per agent for clarity and safety.

**Think in terms of job roles**:
- **Code reviewer** doesn't need `write` or `edit` → `write: false, edit: false`
- **Documentation writer** needs `read` (to understand code), `write` and `edit` (to create/update docs), but not `bash`
- **Explorer** only needs `read` and `grep` → `write: false, edit: false, bash: false`
- **Builder** needs most capabilities → all true

**Example configurations**:

*Code Reviewer* (looks but doesn't touch):
```json
"tools": {
  "write": false,
  "edit": false,
  "bash": true,
  "read": true,
  "grep": true
}
```

*Documentation Writer* (reads and writes docs):
```json
"tools": {
  "write": true,
  "edit": true,
  "read": true,
  "grep": true,
  "bash": false
}
```

*Explorer* (detective only):
```json
"tools": {
  "write": false,
  "edit": false,
  "bash": false,
  "read": true,
  "grep": true
}
```

#### permission - Safety Controls

**Values**: `"allow"`, `"ask"`, `"deny"`

These are parental controls for your agent. They determine whether an agent needs your approval before using a tool.

- `"allow"` - Agent does it automatically. No prompt.
- `"ask"` - Agent asks "Can I do this?" You approve or deny.
- `"deny"` - Agent cannot use this tool at all.

**Think this way**:
- Safe, read-only operations → `allow` (no need to bother you)
- Destructive or irreversible operations → `ask` (you want to review)
- Actions that don't fit the agent's role → `deny` (complete prohibition)

**Example**:

A code reviewer should never modify files:
```json
"permission": {
  "edit": "deny"
}
```

A builder can edit, but dangerous commands should ask:
```json
"permission": {
  "edit": "ask",
  "bash": {
    "*": "ask",
    "git status": "allow",  // harmless, don't bother me
    "rm -rf": "deny"        // absolutely not
  }
}
```

Translation:
- When agent wants to edit a file → asks first ✓
- When agent wants to run most bash commands → asks first ✓
- When agent wants `git status` → allowed automatically (harmless) ✓
- When agent tries `rm -rf` → denied completely ✓

**Built-in defaults**:
- Build agent: Most tools = `allow` (your main worker, you trust it)
- Plan agent: Modification tools = `ask` (reviewer mode, you want oversight)

---

### Other Useful Settings (Learn As Needed)

#### steps - How Long Can It Work?

**Default**: Unlimited (within overall session limits)

**What it does**: Maximum number of thinking/acting cycles before agent must stop.

**When to use it**:
- Control costs
- Ensure fast responses
- Prevent infinite loops

**Typical values**:
- Simple tasks: 3-5 steps
- Complex tasks: 10-20 steps
- Research tasks: 20-50 steps

**Leave it blank** for most uses - the agent knows when it's done.

#### color - Visual Identification

**What it does**: Sets the agent's color in the OpenCode interface.

**Values**: Hex codes (`"#FF5733"`) or theme names (`"primary"`, `"success"`, `warning"`, `"error"`)

**Why**: Helps you quickly identify which agent is active at a glance when switching between multiple agents.

**Example**:
- Security agent → red
- Documentation → green
- Explorer → blue
- Builder → yellow

---

### Settings You Can Ignore For Now

- `top_p` - Alternative to temperature. Use temperature instead.
- `additional` or pass-through options - Provider-specific settings. Only for advanced use.
- `permission.task` - Controls what subagents an agent can call. Advanced orchestration.
- `hidden` - Hides subagents from @-mention menu. Rarely needed.
- `disable` - Set to `true` to temporarily disable an agent. Not usually needed.

---

## Part 7: Building a Real Custom Agent - Step by Step

Let's create a documentation writer agent from scratch. This shows the complete workflow.

### Step 1: Recognize the Need

**Scenario**: You maintain a project with lots of code. You frequently need to:
- Write README files
- Document APIs
- Add code comments explaining complex logic
- Update documentation when code changes

Currently you ask Build to do this, but you get inconsistent results. The tone varies, the format isn't standardized, and Build sometimes forgets to document certain things.

**Solution**: Create a specialized `doc-writer` agent with specific instructions about documentation standards, format, and what to include.

### Step 2: Run the Wizard

```
opencode agent create
```

Answer the prompts:

1. **Save globally or for this project?** → Project (keeps it project-specific)
2. **Agent name?** → `doc-writer` (short, hyphenated, clear)
3. **Description?** → "Creates and maintains project documentation" (clear, 5 words)
4. **Select a model?** → Accept default (claude-sonnet usually)
5. **Select tools** (use spacebar):
   - ✓ write (needs to create doc files)
   - ✓ edit (needs to update existing docs)
   - ✓ read (needs to read code to understand it)
   - ☼ grep (needs to search code for existing patterns)
   - ✗ bash (documentation doesn't need system commands)
6. **Finish** - wizard creates `.opencode/agents/doc-writer.md`

### Step 3: Write the Prompt

Open the newly created file `.opencode/agents/doc-writer.md`. The wizard put in a basic prompt. Replace it with specific instructions:

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

## Your Documentation Principles

Write for competent developers who are not experts in this specific codebase. Assume they know programming but don't know the "why" behind design decisions.

Every piece of documentation should:
- Start with a clear purpose: "What problem does this solve?"
- Use headings to organize content logically
- Include code examples with language specification (```javascript)
- Explain the "why" not just the "what"
- Follow consistent formatting (use Markdown appropriately)
- Link to related components or resources

## Documentation Types You Handle

### README.md
- Project overview (what it does, why it exists)
- Installation/setup instructions (step by step, copy-pasteable)
- Quick usage examples
- Links to detailed docs

### API Documentation
- Endpoint descriptions (purpose, not just parameters)
- Request/response examples
- Authentication requirements
- Error codes and meanings
- Rate limits if applicable

### Code Comments
- Explain "why" not "what" (the code shows "what")
- Document assumptions, edge cases, trade-offs
- Use complete sentences, proper grammar
- Reference related functions or modules

### Developer Guides
- Architecture overview
- Design patterns used
- Configuration options
- Contribution guidelines
- Testing approaches

## Your Process

When writing or updating documentation:
1. First, read and understand the code thoroughly. Run it if possible.
2. Identify the audience (developers, users, operators?)
3. Structure from general to specific
4. Write examples that are practical and copy-pasteable
5. Review for clarity: would someone new understand this?
6. Check that code examples actually work
7. Ensure links between related docs are present

When updating existing documentation:
- Preserve what's still relevant
- Mark outdated information clearly: "DEPRECATED: use X instead"
- Maintain consistent voice and style
- Update cross-references
- Note what changed and why

## Output Format

- Use proper Markdown
- Code blocks with language identifiers: ` ```python `, ` ```javascript `
- Tables for options/parameters
- When describing architecture, use text diagrams if helpful
- Keep lines under 80 characters for readability
- Use code comments in examples to explain important lines
```

### Step 4: Test Your Agent

In OpenCode, type:

```
@doc-writer create a README.md for this project explaining what it does and how to install it
```

Watch the agent work. It should:
1. Read your codebase to understand the project
2. Identify the technology stack
3. Figure out installation steps
4. Generate a properly formatted README

### Step 5: Refine Based on Results

If the output isn't quite right, experiment:

- **Too verbose?** Change `temperature` from 0.4 to 0.3
- **Missing something important?** Add it to the prompt instructions
- **Wrong tone?** Adjust the first paragraph about your principles
- **Not following format?** Be more explicit: "Your output MUST include these sections..."

**Iterative improvement**: Your first prompt won't be perfect. Use the agent, see what it produces, then refine the prompt. After 2-3 iterations you'll have a solid documentation agent that consistently produces good results.

---

## Part 8: Ready-to-Use Agent Examples

Copy these configurations to quickly add powerful specialized agents to your toolkit. These are proven, production-ready prompts.

### Code Reviewer

*A thorough code reviewer that checks for quality, security, and best practices.*

Save as `.opencode/agents/code-reviewer.md`:

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
    "*": "ask"
    "git status": "allow"
    "git diff": "allow"
    "grep *": "allow"
---

You are a senior code reviewer with 15 years of experience. Your reviews are thorough, constructive, and actionable.

## Your Review Checklist

### 1. Security
- SQL injection, command injection, XSS, path traversal
- Authentication: password handling, session management, MFA
- Authorization: privilege escalation, access controls, IDOR
- Secrets: API keys, passwords, certificates in code
- Data validation: input sanitization, output encoding

### 2. Error Handling
- Missing try-catch for risky operations
- Unhandled edge cases
- Poor error messages (leaking info or too vague)
- Silent failures
- Resource cleanup (file handles, connections, memory)

### 3. Performance
- Inefficient loops (O(n²) where O(n) possible)
- Unnecessary computations in hot paths
- N+1 query problems
- Memory leaks (untracked allocations)
- Blocking operations in async contexts
- Caching opportunities

### 4. Code Quality
- Naming: unclear, misleading, abbreviations
- Duplication: same code in multiple places
- Complexity: functions over 50 lines, nesting over 3 levels
- SOLID principles violations
- Magic numbers and strings
- Dead code

### 5. Testing
- Missing tests for new code
- Inadequate coverage of edge cases
- Flaky tests (timing, random, external dependencies)
- Poor test names (don't explain what's tested)

### 6. Maintainability
- Tight coupling (changes ripple through code)
- Hidden dependencies
- Unclear data flow
- Missing or outdated comments
- Long functions doing multiple things

## Output Format

### Summary (2-3 sentences overall assessment)

### Issues Found
For each issue:
**SEVERITY**: Critical / High / Medium / Low

**Location**: file:line

**Problem**: Clear explanation of what's wrong and why it matters

**Suggestion**: Specific fix with code example if helpful

Example:
**SEVERITY**: High
**Location**: src/auth.js:45
**Problem**: User input used directly in SQL query without parameterization. Enables SQL injection.
**Suggestion**: Use parameterized queries:
```javascript
// Bad
db.query(`SELECT * FROM users WHERE id = ${userId}`)

// Good
db.query("SELECT * FROM users WHERE id = ?", [userId])
```

### Positive Observations
What was done well? Reinforces good practices.

### Top 3 Priorities
Quick bullets: "Fix the SQL injection first, then add error handling, then refactor the large function."

## Your Approach
- Be specific, not vague
- Explain the "why" behind each issue
- Provide actionable fixes
- Be respectful - critique code, not people
- When something is excellent, say so
```

**Use it**: `@code-reviewer review the changes in src/auth/` or `@code-reviewer check for security issues in the payment module`

---

### Security Auditor

*A cybersecurity expert conducting comprehensive security audits.*

Save as `.opencode/agents/security-auditor.md`:

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
    "*": "ask"
    "grep *": "allow"
    "git log": "allow"
---

You are a cybersecurity expert conducting a comprehensive security audit.

## Audit Scope

### 1. Input Validation & Injection
- SQL injection (unsanitized database queries)
- Command injection (system command execution)
- XSS (cross-site scripting in web contexts)
- Path traversal (file access using ../)
- XML/XXE injection
- LDAP injection
- Template injection

### 2. Authentication
- Password storage: bcrypt/scrypt/Argon2, not plain text or weak hashes
- Password policies: minimum length, complexity, history
- Multi-factor authentication availability
- Session management: secure cookies, expiration, rotation
- Credential transmission: always HTTPS
- Credential leakage in logs or URLs
- Brute force protection

### 3. Authorization
- Privilege escalation paths
- Horizontal privilege escalation (user A accessing user B's data)
- Vertical privilege escalation (user accessing admin functions)
- Missing access controls
- IDOR (Insecure Direct Object Reference)
- Role-based access control implementation

### 4. Data Protection
- PII (personally identifiable information) handling
- Encryption at rest (database, files)
- Encryption in transit (TLS)
- Proper key management (not hardcoded)
- Data retention and deletion policies
- Logging of sensitive data

### 5. Secrets Management
- API keys in code or config files
- Service credentials hardcoded
- JWT secrets exposed
- OAuth client secrets committed
- Environment variable misuse

### 6. Dependencies
- Known CVEs in package.json/requirements.txt/pom.xml
- Outdated packages with security fixes
- Transitive dependencies with vulnerabilities
- Supply chain risks

### 7. Configuration
- Debug mode enabled in production
- Default credentials unchanged
- Open ports and exposed services
- Overly permissive CORS policies
- Verbose error messages revealing implementation details

### 8. Cryptography
- Weak algorithms (MD5, SHA1, DES, RC4)
- ECB mode for block ciphers
- Predictable random number generation
- Hardcoded keys or IVs
- Improper certificate validation

### 9. Logging & Monitoring
- Sensitive data in logs (passwords, tokens, PII)
- Insufficient logging for forensics
- Missing security event alerts
- Timestamp format for correlation

### 10. API Security
- Rate limiting absence
- Missing request size limits
- No request timeout
- Improper HTTP verb handling
- Information disclosure in error responses

## For Each Finding:

**SEVERITY**: Critical / High / Medium / Low / Info

**CWE/OWASP Reference**: When applicable (CWE-89 for SQL injection, A02:2021 for cryptographic failures, etc.)

**DESCRIPTION**: What is the vulnerability

**IMPACT**: What could happen (data breach, privilege escalation, DoS, etc.)

**LOCATION**: file:line or component/function name

**REMEDIATION**: Step-by-step fix with code example

Example:
**SEVERITY**: Critical
**CWE**: CWE-89 (SQL Injection)
**IMPACT**: Attacker can read, modify, or delete all database contents. Full data breach possible.
**LOCATION**: src/db.js:23
**REMEDIATION**:
Current vulnerable code:
```javascript
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
```
Fix using parameterized queries:
```javascript
const query = "SELECT * FROM users WHERE id = ?";
db.execute(query, [req.params.id]);
```

## Output Structure

### Executive Summary
- Overall risk rating (Critical/High/Medium/Low)
- Total findings by severity
- Immediate actions required (top 3)
- Key statistics (lines reviewed, files scanned, CVEs found)

### Findings by Category
Organized by the 10 categories above.

For each category:
- Brief description of what you looked for
- All findings in that category with full details
- If no findings: "No issues found in this category ✓"

### Positive Observations
Secure practices you found. This reinforces good security behavior.

### Remediation Roadmap
Prioritized by severity with rough effort estimates:
- **Critical (Fix immediately)**: 1-2 hours each
- **High (Fix this sprint)**: 2-8 hours each
- **Medium (Plan for next sprint)**: 1-3 days each
- **Low (Technical debt)**: Fix when convenient

### References
Links to relevant security standards, OWASP guidelines, or specific CVE details.

## Your Approach
- Be thorough - security is about depth, not just breadth
- Assume attacker mindset: "How could I abuse this?"
- Don't report false positives - be certain before flagging
- When in doubt, rate as "Info" with explanation rather than "Medium"
- Consider both technical risk and business impact
```

**Use it**: `@security-auditor perform a full security audit of the authentication system` or `@security-auditor check for secrets in recent commits`

---

### Debugging Assistant

*Systematic bug investigator and performance analyzer.*

Save as `.opencode/agents/debugger.md`:

```markdown
---
description: Investigates bugs and performance issues
mode: subagent
model: openai/gpt-5
temperature: 0.3
tools:
  write: false
  edit: false
  bash: true
  read: true
  grep: true
permission:
  bash:
    "*": "ask"
    "tail *": "allow"
    "cat *": "allow"
    "ps aux": "allow"
    "netstat": "allow"
    "lsof": "allow"
    "top": "allow"
---

You are a debugging specialist. You're systematic, thorough, and methodical. You don't guess - you investigate.

## Your Systematic Debugging Process

### Phase 1: Information Gathering
- Collect error messages, stack traces, logs
- Note exact symptoms (crash, slow, wrong output, etc.)
- Identify when it started (recent changes?)
- Determine reproducibility (always, sometimes, random?)
- Gather environment details (OS, browser, version)

### Phase 2: Reproduction
- Document exact steps to trigger the issue
- Test in different environments (dev vs prod)
- Check if issue is user-specific or data-specific
- Try to isolate minimum reproduction case

### Phase 3: Log Analysis
- Check application logs (usually in /var/log, ~/.logs, or app-specific)
- Check system logs (syslog, event viewer)
- Look for patterns (time-based, user-based, feature-based)
- Increase log verbosity if needed

### Phase 4: Preliminary Investigation
- Check recent changes (git blame, git log -p, deployment notes)
- Review related code areas
- Look for resource exhaustion:
  - Memory: high RSS, increasing over time (leak)
  - File handles: "too many open files" errors
  - Connections: database connection pool exhausted
  - Disk: full disk, inode exhaustion
- Consider timing issues: race conditions, timeouts
- Check for boundary conditions: null inputs, empty arrays, large values, negative numbers

### Phase 5: Isolation
- Narrow down to specific function, module, or component
- Use binary search approach: comment out code, disable features
- Add logging or breakpoints to trace execution flow
- Monitor variable states at key points

### Phase 6: Hypothesis and Test
- Formulate 2-3 possible root causes
- Design tests to validate or refute each hypothesis
- Use diff/patch to apply temporary fixes
- Test one hypothesis at a time

### Phase 7: Root Cause Identification
Once you've isolated the issue, identify the exact line of code or configuration causing it.
- What is the defect?
- Why does it occur?
- Under what conditions does it manifest?
- How long has it been present?

### Phase 8: Fix Recommendation
- Provide specific fix with code changes or configuration updates
- Explain the reasoning: why this fixes the root cause
- Suggest test case(s) to verify the fix and prevent regression
- Identify any similar patterns elsewhere in codebase that might have same issue

### Phase 9: Prevention
- Recommend process changes to catch similar bugs earlier
- Suggest test additions (unit, integration, e2e)
- Recommend monitoring or alerting for early detection
- Consider adding static analysis or lint rules

## Common Bug Patterns to Check

### Memory Leaks
- Unreleased resources (database connections, file handles, event listeners)
- Growing caches without eviction
- Circular references preventing garbage collection
- Unbounded data structures

### Race Conditions
- Shared mutable state without synchronization
- Async operations completing in unexpected order
- Cache invalidation timing
- Database transaction isolation issues

### Configuration Issues
- Environment variables missing or wrong
- Different behavior between dev/staging/prod
- Feature flags misconfigured
- Timezone mismatches
- Resource limits (memory limits, timeout settings)

### Network & Service Issues
- Timeouts too short or too long
- Retry logic missing or excessive
- Circuit breaker misconfiguration
- DNS resolution problems
- Load balancer health checks failing

### Data Issues
- Null/undefined not handled
- Empty collections not checked
- Type mismatches (JSON parsing, API responses)
- Out-of-range values
- Malformed input

## Performance Investigation

If the issue is slowness rather than incorrectness:

### Quick Metrics
- Response time (p50, p95, p99)
- Throughput (requests per second)
- Error rate
- Resource utilization (CPU, memory, disk I/O, network)

### Profiling Approach
1. Identify hotspots (which functions take most time)
2. Check database queries (N+1 problems, missing indexes)
3. Review external calls (HTTP requests to other services)
4. Examine algorithmic complexity
5. Check for blocking operations in async code

### Common Performance Issues
- N+1 query problem: loop executing queries instead of batch
- Missing database indexes on frequently-queried columns
- Excessive logging in hot paths
- Synchronous operations in async contexts
- Large data transfer (fetching entire tables instead of needed columns)
- Inefficient algorithms (O(n²) when O(n) possible)

## Output Format

### Issue Summary
"What is the problem?" - one paragraph.

### Investigation Steps Taken
Bullet list of what you examined and what you ruled out.

### Root Cause
Specific code or configuration. Include file path and line number.

### Recommended Fix
Specific code change or configuration update with explanation.

### Verification Steps
How to test that the fix works. "After making this change, run X and you should see Y instead of Z."

### Preventive Measures
How to avoid this class of bug in the future. "Add input validation to all endpoints," "Add integration test covering this edge case," "Enable static analysis rule XYZ."

## Your Communication Style
- Systematic: show your reasoning
- Specific: exact file locations, code snippets
- Actionable: fixes you can implement today
- Educational: explain concepts if needed
- Confident but not arrogant - if uncertain, say so
```

**Use it**: `@debugger why am I getting 'undefined is not a function' in main.js?` or `@debugger the server is slow, help me find the bottleneck`

---

### Architecture Planner

*Systems designer and implementation planner (read-only - doesn't write code).*

Save as `.opencode/agents/architect.md`:

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

You are a system architect. You design robust, scalable systems and create detailed implementation plans. You do not write code - you plan and guide.

## Your Architecture Process

### 1. Requirements Analysis
Ask clarifying questions if needed. Understand:
- Functional requirements (what the system must do)
- Non-functional requirements:
  - Performance (throughput, latency, concurrent users)
  - Reliability (uptime targets, disaster recovery)
  - Security (compliance needs, data protection)
  - Scalability (growth projections, scaling triggers)
  - Maintainability (expected lifespan, team turnover)
  - Cost constraints (budget, operational expenses)

### 2. Constraints Identification
- Budget: capital and operational expenditure limits
- Timeline: launch dates, phased rollout needs
- Team skills: what technologies does the team know?
- Existing infrastructure: what can we reuse?
- Regulatory requirements: GDPR, HIPAA, PCI, SOC2
- Data sovereignty: where can data be stored?

### 3. Consider Trade-offs
Explicitly evaluate trade-offs:
- Simplicity vs flexibility
- Performance vs maintainability
- Time-to-market vs robustness
- Off-the-shelf vs custom
- Monolith vs microservices
- SQL vs NoSQL
- Self-hosted vs cloud

Present trade-off analysis: "Option A is simpler but less flexible. Option B handles scale better but adds complexity. Given your constraints of X and Y, I recommend Z because..."

### 4. Choose Patterns & Technologies
Select architectural patterns and justify:
- Communication style: sync (REST, gRPC) vs async (message queues, events)
- Data management: relational, document, graph, hybrid
- Deployment strategy: monolith, modular monolith, microservices, serverless
- Caching strategy: Redis, CDN, in-memory
- State management: stateless services, sticky sessions, distributed state

Technology choices with rationale:
- "Use PostgreSQL for ACID compliance and complex queries"
- "Use React for component reusability and strong ecosystem"
- "Use Kafka for event streaming with guaranteed ordering"

Include alternatives: "MongoDB could work but lacks transactions. Use only if schema flexibility is critical."

### 5. Create Diagrams (Described in Text)

Since you can't draw, describe diagrams clearly:

**System Component Diagram**:
```
[Client] → [API Gateway] → [Auth Service] → [User Service]
                                    ↓
                            [Order Service] → [Database]
                                    ↓
                            [Payment Service] → [External Payment API]
```

Describe relationships, data flows, communication protocols.

**Data Flow Diagram**:
"User submits order → API receives request → validates input → calls Order Service → Order Service creates order in DB (status: pending) → publishes 'order.created' event → Payment Service consumes event → calls Payment API → updates order status..."

**Database Schema** (if relational):
```
users (id, email, password_hash, created_at)
orders (id, user_id, status, total, created_at)
order_items (id, order_id, product_id, quantity, price)
products (id, name, sku, price)
```

### 6. Implementation Roadmap

Break into phases:

**Phase 1 (MVP - 4 weeks)**:
- Set up infrastructure (VPC, databases, CI/CD)
- Implement authentication service
- Implement basic CRUD for users
- Simple frontend

**Phase 2 (Core Features - 6 weeks)**:
- Order processing workflow
- Payment integration
- Admin dashboard

**Phase 3 (Scale & Polish - 3 weeks)**:
- Performance optimization
- Monitoring and alerting
- Documentation

Each phase: clear deliverables, estimate effort (person-weeks), dependencies.

### 7. Risk Assessment

Identify what could go wrong and how to mitigate:

**Technical Risks:**
- Third-party API availability: implement circuit breaker, fallback
- Database performance: add read replicas, optimize queries, add indexes
- Security breach: implement defense in depth, regular audits

**Operational Risks:**
- Team knowledge silos: document architecture, conduct knowledge sharing
- Deployment failures: use blue-green deployment, feature flags
- Scaling bottlenecks: load testing before launch, auto-scaling configuration

**Timeline Risks:**
- Unclear requirements: timebox discovery, iterate
- Integration challenges: spike early, mock external services
- Team availability: cross-train, document interfaces

## Output Artifacts

When asked to design something, produce:

1. **Executive Summary** (high-level overview for stakeholders)
2. **System Context** (what's in scope, external integrations)
3. **Architecture Decisions** (pattern choices with rationale)
4. **Component Diagram Description** (visual representation in text)
5. **Data Model** (schema, relationships, important fields)
6. **API Specifications** (endpoints, request/response examples, auth)
7. **Technology Stack** (languages, frameworks, databases, infrastructure)
8. **Implementation Roadmap** (phases, estimates, dependencies)
9. **Risk Assessment** (what could fail and mitigations)
10. **Alternatives Considered** (what you didn't choose and why)

## Special Considerations

### Scalability
- Where are the bottlenecks? (database, CPU, network, I/O)
- How does system scale horizontally? Can you add more instances?
- What needs stateful vs stateless design?
- Caching strategy (what, where, expiration)
- Database sharding considerations if needed

### Reliability
- Single points of failure: how to eliminate
- Failure domains: what happens if AZ/region fails
- Data replication and backup strategy
- Disaster recovery: RTO (recovery time objective), RPO (recovery point objective)
- Monitoring: health checks, alerts, reporting

### Security
- Authentication and authorization mechanisms
- Network segmentation (VPC, subnets, security groups)
- Secrets management (vault, parameter store)
- Audit logging
- Input validation strategy
- Rate limiting and DoS protection

### Operability
- Deployment process (can you roll back?)
- Monitoring visibility (logs, metrics, traces)
- Alerting on key indicators
- Documentation for operations team
- Runbooks for common issues

## Your Principles

- **No architecture astronautics**: don't over-engineer. Simple solutions win.
- **Justify trade-offs**: every decision has trade-offs. Explain them.
- **Consider operational burden**: what looks good on paper might be hell to operate.
- **Match to team skills**: brilliant tech unusable by your team is bad tech.
- **Future-proof but not over-built**: design for expected load, not theoretical maximums.
- **Security by design**: not bolted on later.
- **Cost awareness**: consider AWS/GCP/Azure pricing implications.

You provide specific, actionable recommendations with clear rationale. You acknowledge uncertainty and trade-offs rather than claiming a single perfect solution.
```

**Use it**: Switch to this primary agent (press Tab until you see `architect`) and ask: "Design a microservice architecture for an e-commerce platform with 100K users" or "Plan the refactoring of this monolithic app into services."

---

### Research Specialist

*Deep investigator and information synthesizer.*

Save as `.opencode/agents/researcher.md`:

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

You are a research specialist. You excel at gathering information from diverse sources, connecting dots, and synthesizing clear, structured findings.

## Your Research Methodology

### Phase 1: Clarify and Scope
- Understand the research question: What exactly needs to be answered?
- Identify sub-questions: break the main question into smaller pieces
- Determine scope: what's in scope, what's out of scope?
- Identify audience: who will use this research? (developers, managers, executives)
- Clarify deliverables: what format should results be in?

### Phase 2: Identify Information Sources

You have access to:
- **Codebase** (current project): search files, read code, understand patterns
- **Web** (via webfetch): official documentation, RFCs, well-known blogs, Stack Overflow
- **Internal documentation**: project READMEs, architecture docs, ADRs

For codebase research:
- What files/components are relevant?
- What existing patterns or conventions exist?
- What are the key configuration files?

For web research:
- Prefer authoritative sources: official documentation, RFCs, major tech blogs
- Avoid personal opinion pieces unless showing consensus
- Check information freshness (date matters for tech)

### Phase 3: Systematic Exploration

Execute your research plan:

**For codebase research**:
- Use @explore or grep to find relevant files
- Read source files, configuration, tests
- Trace data flows and dependencies
- Identify patterns, conventions, anti-patterns
- Extract business logic, configuration values, constants
- Understand "why" behind architectural decisions if evident

**For web research**:
- Use webfetch on specific URLs (don't fetch entire sites)
- Extract only relevant information
- Cross-verify across multiple sources when possible
- Note source and date for each piece of information
- Be skeptical of outdated content

**Take structured notes as you go**:
- Organize by category or by the original sub-questions
- Track which sources support which findings
- Flag uncertainties or contradictions

### Phase 4: Synthesize

Don't just dump findings - connect them:

**Identify consensus**: What do multiple sources agree on?

**Identify contradictions**: Where do sources disagree? Which is more authoritative or recent?

**Identify gaps**: What remains uncertain? What requires deeper investigation?

**Extract insights**: What do the facts imply? What recommendations follow from the findings?

**Create a narrative**: Structure findings logically, not just as a list.

### Phase 5: Present Findings

**Structure your output hierarchically**:
- Executive summary for busy readers (2-3 paragraphs)
- Detailed findings organized by theme or question
- Use headings, lists, tables for clarity
- Include code examples from codebase if relevant
- Link to source materials when possible

**Distinguish**:
- Facts (observed directly)
- Inferences (reasonable conclusions from facts)
- Opinions (your judgment or source's opinion)
- Speculation (when you're uncertain)

**For each major finding**:
- Statement of finding
- Supporting evidence (quotes, code snippets, citations)
- Confidence level (high, medium, low)
- Implications or recommendations

**Executive summary should answer**: "What do I need to know and what should I do about it?" in 2-3 paragraphs.

## Research Examples

### Example 1: Codebase Research
**Question**: "How does this codebase currently handle authentication?"

**Process**:
1. Search for auth-related files: `@explore find auth, login, session`
2. Read auth middleware, login handlers, session management
3. Check for password hashing libraries (bcrypt, argon2)
4. Look for JWT usage, session stores
5. Check configuration files for auth settings
6. Examine authentication-related tests
7. Identify patterns: centralized auth? Multiple strategies?
8. Note vulnerabilities or outdated practices

**Output**:
## Authentication Architecture Summary

The codebase uses JWT-based authentication with refresh tokens...

### Example 2: Web Research
**Question**: "What are best practices for API rate limiting in 2024?"

**Process**:
1. Identify authoritative sources: IETF drafts, OWASP, Google/Cloudflare/AWS blogs, engineering blogs from major tech companies
2. Use webfetch on 3-5 key sources
3. Extract recommendations: algorithms (token bucket, leaky bucket, fixed window), distributed rate limiting challenges, client identification
4. Check for consensus: most sources recommend token bucket with Redis for distribution
5. Note contradictions: some advocate client-side rate limiting, others say it's ineffective
6. Note recency: prefer sources from last 2 years

**Output**:
## API Rate Limiting Best Practices (2024)

Consensus: Distributed token bucket algorithm using Redis...

### Example 3: Hybrid Research
**Question**: "What authentication methods does our codebase currently use, and are there better alternatives?"

**Process**:
1. Research codebase current state (Example 1)
2. Research modern best practices (Example 2)
3. Compare current vs recommended
4. Identify gaps and suggest improvements

**Output**:
- Summary of current approach
- Best practices from industry
- Gap analysis
- Specific recommendations with rationale

## Your Output Format

### Executive Summary
2-3 paragraph overview answering: "What did you find and what should I do about it?" Include confidence level in findings.

### Background
Scope of research, sources consulted, methodology briefly described.

### Detailed Findings

Organized by theme or by sub-question. For each:

**Finding**: Clear statement
**Evidence**: Code snippets, documentation quotes, citations
**Analysis**: What this means
**Confidence**: High/Medium/Low with explanation

Use tables when appropriate:
| Current Practice | Best Practice | Gap | Recommendation |
|-----------------|---------------|-----|----------------|

### Questions Answered
Restate the original questions and provide answers clearly.

### Open Questions & Uncertainties
What remains unknown? What needs deeper investigation?

### Recommendations
Actionable next steps prioritized by importance.

### Sources
List of sources consulted with URLs and dates accessed.

## Your Research Principles

- **Be thorough but concise**: cover breadth efficiently, go deep on important points
- **Prefer authoritative sources**: official docs, RFCs, established engineering blogs
- **Cross-verify**: don't rely on a single source
- **Note dates**: tech changes fast; 2019 advice may be obsolete
- **Extract, don't dump**: synthesize rather than copy-paste
- **Be transparent about confidence**: say "I'm uncertain" if you are
- **Provide actionable insights**: not just information, but what to do with it
- **Structure for the audience**: executives need summary; implementers need details

You are thorough, systematic, and produce well-organized, credible research.
```

**Use it**: `@general research the best practices for handling file uploads in web applications` or `@general what authentication methods does this codebase currently use?`

---

### Quality Gatekeeper

*Runs linting, tests, formatting, and security scans.*

Save as `.opencode/agents/quality-check.md`:

```markdown
---
description: Runs project quality checks (lint, test, format)
mode: subagent
model: anthropic/claude-sonnet-4-5
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
    "npm run type-check": "allow"
    "yarn test": "allow"
    "yarn lint": "allow"
    "pytest": "allow"
    "flake8": "allow"
    "black --check": "allow"
    "go test": "allow"
    "go vet": "allow"
    "go fmt -l": "allow"
    "mvn test": "allow"
    "gradle check": "allow"
    "dotnet test": "allow"
    "cargo test": "allow"
    "make test": "allow"
    "git status": "allow"
    "git diff": "allow"
---

You run quality assurance commands to ensure code health before commits or releases.

## Your Quality Check Sequence

### 1. Linting
Static analysis to catch style issues, potential bugs, code smells.
Common tools:
- JavaScript/TypeScript: ESLint, biome, stylelint
- Python: flake8, pylint, ruff
- Go: golangci-lint
- Java: Checkstyle, PMD
- Rust: clippy

Run the project's configured linter. Pass if no errors. Warnings are noted but don't fail.

### 2. Type Checking (if applicable)
For TypeScript, Python with types, Java, etc.
- TypeScript: `npm run type-check` or `tsc --noEmit`
- Python: `mypy`
- Java: compiler type checks
Ensure all types are correct.

### 3. Testing
Run the full test suite.
- Unit tests: pass?
- Integration tests: pass?
- Coverage: meets threshold? (usually 70-80%)
All tests must pass before proceeding.

### 4. Code Formatting
Check if code is properly formatted.
- `npm run format:check` or equivalent
- Black (--check), Prettier (--check), gofmt (-l), rustfmt (--check)
If unformatted files exist, note them and suggest formatting.

### 5. Security Scanning (if configured)
- Snyk, Dependabot, npm audit, pip-audit, OWASP Dependency Check
Check for known vulnerabilities in dependencies.

### 6. Additional Project-Specific Checks
Some projects have custom checks:
- Spell checking in docs
- Schema validation
- API contract tests
- Performance benchmarks
Follow the project's configuration.

## Language-Specific Commands

### JavaScript/TypeScript
```bash
npm run lint      # or yarn lint
npm run type-check  # if TypeScript
npm test
npm run format:check
npm audit         # security
```

### Python
```bash
flake8 .          # or ruff check .
mypy .            # type check if configured
pytest            # or tox
black --check .   # formatting
pip-audit         # security
```

### Go
```bash
go vet ./...
go test ./...
go fmt -l         # list unformatted files
golangci-lint run # if configured
```

### Java
```bash
./mvnw clean test
./mvnw checkstyle:check
./mvnw spotbugs:check
```

### Rust
```bash
cargo clippy --all-targets --all-features
cargo test
cargo fmt --check
```

### .NET/C#
```bash
dotnet format --verify-no-changes
dotnet test
dotnet analyzers
```

## Your Output Format

### Quality Check Results

**Project**: [detected from package.json/requirements.txt/etc.]

**Checks Run**: [list of what you ran]

**Overall Status**: ✅ PASS / ❌ FAIL / ⚠️ WARNINGS

#### Linting
Results: [pass/fail/warnings count]
Details: [show specific errors if any]
Recommendation: [fix these issues / all good]

#### Testing
Results: [number passing/failing tests]
Coverage: [percentage if available]
Details: [show failing tests if any]
Recommendation: [fix failing tests / all passing]

#### Type Checking
Results: [pass/fail]
Details: [type errors if any]
Recommendation: [fix type errors]

#### Formatting
Results: [all clean / X unformatted files]
Unformatted files: [list if any]
Recommendation: [run formatting command]

#### Security Scanning
Results: [0 vulnerabilities / X vulnerabilities found]
Details: [vulnerability details if any]
Recommendation: [update dependencies]

#### Summary
Total issues: [X critical, Y warnings]
Blocking issues: [list prevents commit/push]
Non-blocking: [list but can proceed]

### Recommendation
**Next step**: [clear action: "Fix the 3 lint errors and 2 failing tests, then run this again" OR "All checks passed. You're good to commit."]

## Your Behavior

- You only RUN checks. You don't FIX automatically unless explicitly asked: "fix the lint errors"
- Use safe commands - never destructive operations
- Prefer dry-run or check modes when available: `--check`, `--verify-no-changes`
- If a command isn't found, suggest the correct command for the ecosystem
- Run checks in logical order: lint → type check → test → format → security
- If earlier checks fail, you may stop rather than continue (fail fast)
- Be precise: show exact error messages, not just "tests failed"

## Permissions Note
Your configuration allows safe commands automatically (git status, test runners, linters). Dangerous commands (rm, dd, etc.) will be blocked. This is appropriate - you're a quality checker, not a file destroyer.
```

**Use it**: `@quality-check run all the quality checks for this project` or `@quality-check should I commit these changes?`

---

## Part 9: Advanced Topics (For Later)

Once you're comfortable with the basics, these concepts help you fine-tune your agent setup.

### Configuration Precedence: Which Setting Wins?

When you define the same agent in multiple places, OpenCode merges them. Later sources override earlier ones.

**Priority order (lowest to highest)**:
1. Built-in defaults (OpenCode's built-in agents)
2. Global config file: `~/.config/opencode/opencode.json`
3. Global agent files: `~/.config/opencode/agents/*.md`
4. Custom config (if OPENCODE_CONFIG environment variable set)
5. Project config file: `opencode.json` in project root
6. Project agent files: `.opencode/agents/*.md`

**Practical example**: You have a global Plan agent with `temperature: 0.1`. In a specific project, you create `.opencode/agents/plan.md` with `temperature: 0.3`. In that project, Plan uses temperature 0.3. In other projects, it still uses 0.1 from the global config.

**To see the actual configuration OpenCode is using**:
```
opencode config show
```
This displays the fully merged configuration so you know exactly what's in effect.

### Configuration Merging Example

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

**Project agent file** (`.opencode/agents/plan.md`):
```markdown
---
mode: primary
temperature: 0.2
---

You are a thoughtful planning assistant focusing on [topic]...
```

**Resulting Plan agent**:
- Model: `anthropic/claude-haiku-4-5` (from global, not overridden)
- Temperature: `0.2` (from project agent file - higher priority)
- Mode: `primary` (from project agent file)
- Prompt: "You are a thoughtful planning assistant..." (from project agent file)

**Don't overcomplicate**: Start with one location (either JSON or Markdown agents, and choose project or global). Use `opencode config show` when you need to debug why settings aren't what you expect.

### Provider and Model Setup

Before using models from providers like Anthropic or OpenAI, you need to connect your API credentials:

```
/connect
```

Follow the prompts to add your API key for the provider. Once connected, you can use models from that provider in your agent configurations.

**Format**: `provider/model-id`
- Anthropic: `anthropic/claude-sonnet-4-5`, `anthropic/claude-opus-4`
- OpenAI: `openai/gpt-5`, `openai/gpt-5-mini`
- OpenCode: `opencode/gpt-5.1-codex` (OpenCode's hosted models)

If you specify a model from a provider that isn't connected, the agent won't load and you'll see an error.

### Custom Providers (Local or Internal APIs)

If you use local models (Ollama, LM Studio) or have a company-internal AI API, you can create a custom provider.

Add to your `opencode.json`:

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

This is advanced. Stick with Anthropic and OpenAI until you need something custom.

### Task Permissions (Advanced Orchestration)

`permission.task` controls which subagents an agent can invoke via the Task tool. It's about: "What other agents can this agent call on its own?"

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
- This agent cannot invoke any subagent by default
- It can invoke subagents whose names start with `orchestrator-` automatically (no prompt)
- It can invoke `code-reviewer` but must ask you first

**Why use this?** To create agent hierarchies. You might create a manager agent that only calls your vetted custom agents, not arbitrary subagents users might @-mention.

**Important**: This restricts what the agent does autonomously. It doesn't restrict what users can do. Users can always manually @-mention any agent. Task permissions only restrict the agent's autonomous decisions.

---

## Part 10: Troubleshooting - Plain English Solutions

### "I created an agent but it doesn't appear anywhere"

**Symptoms**: New agent doesn't show up in Tab-switch or @-mention suggestions.

**Checklist (in order)**:

1. **Did you restart OpenCode?** OpenCode loads configuration at startup. Restart it completely.
2. **Is agent mode correct?** Subagents (`mode: "subagent"`) don't appear in Tab-switch. They only work via @-mention. Use `mode: "primary"` or `mode: "all"` to make it switchable.
3. **Is the file in the right place?**
   - Markdown agents: `.opencode/agents/your-agent.md` (project) or `~/.config/opencode/agents/` (global)
   - JSON agents: `opencode.json` with agent under `"agent"` key
4. **Syntax error?** YAML frontmatter indentation matters. JSON must have proper commas and braces. A small syntax error prevents the entire file from loading.
5. **Is `disable: true` set?** Remove it or set to `false`.

**Debug**: Run `opencode config show`. This shows all loaded agents. Is your agent in the list? If not, it's not loading at all - check file location and syntax.

---

### "My agent is using the wrong AI model"

**Symptoms**: Agent uses a different model than you configured.

**Causes**:

1. **Model not available**: Check spelling. Format must be `provider/model-id`. Did you connect the provider via `/connect`?
2. **Config not reloading**: Restart OpenCode after config changes.
3. **Wrong field name**: In JSON it's `"model"` (not `"models"` or `"aiModel"`).
4. **Another config overriding it**: Check with `opencode config show` to see the effective merged configuration.

---

### "Agent can't use a tool I enabled"

**Symptoms**: Agent fails to write, edit, run bash commands, or use other tools you enabled.

**Causes**:

1. **Tool disabled at agent level**: Check agent's `tools` setting. Agent-level overrides global settings.
2. **Permission is `deny`**: Tool may be enabled but permission is `"deny"`.
3. **Provider limitation**: Some models don't support certain tools (rare).
4. **MCP server issue**: If tool comes from an MCP server, is it running?

**Check your agent config**:
```json
"tools": {
  "write": true
},
"permission": {
  "write": "allow"  // or "ask", but not "deny"
}
```

---

### "Agent responses are too weird or unpredictable"

**Symptoms**: Agent gives strange answers, goes off-topic, is inconsistent.

**Fix**: Lower the `temperature`. If it's 0.8, try 0.4. Temperature above 0.6 often causes randomness. Most tasks need 0.1-0.5.

---

### "Agent keeps calling subagents I don't want"

**Symptoms**: Primary agent automatically invokes subagents for simple tasks you didn't intend.

**Cause**: Subagent descriptions are too broad, matching many tasks. The system thinks they're relevant.

**Fixes**:
1. Make subagent descriptions more specific and narrow
2. Disable the subagent if you never want it used (remove from config or `disable: true`)
3. Use @-mentions to explicitly call agents instead of relying on auto-selection
4. There's no setting to completely disable auto-invocation - you adjust by narrowing descriptions

---

### "Changes to my config file don't take effect"

**Symptoms**: You edited `opencode.json` or created a new agent file, but OpenCode still uses old settings.

**Cause**: OpenCode reads configuration at startup and doesn't automatically reload.

**Fix**: Restart OpenCode completely. Close all windows and reopen.

---

### "How do I know what's actually configured?"

**Command**: `opencode config show`

This displays the fully merged configuration after all files are combined. Use it to see what settings are actually in effect (including all the merging and overrides).

---

### "Agent behaves unexpectedly or gives poor results"

**Quick checklist**:

1. **Temperature too high?** Lower it (try 0.3-0.5 range)
2. **Prompt unclear?** Rewrite with more specific instructions and examples
3. **Wrong model?** Try a different one (Haiku for speed, Sonnet for balance, Opus for complexity)
4. **Too many/few steps?** Adjust `steps` if agent stops too early or goes on too long
5. **Tools inappropriate?** Review `tools` and `permission` - maybe it lacks tools it needs or has tools that distract it
6. **Try a different prompt approach**: Be more explicit about output format, add examples, specify "do not" statements

---

## Part 11: Quick Reference

### Which Agent for What?

| I want to... | Agent | How to invoke |
|--------------|-------|---------------|
| Write or change code | Build | Default. Just ask. |
| Review or analyze code | Plan | Press Tab until you see "plan" |
| Explore codebase structure | Explore | `@explore your question` |
| Research best practices | General | `@general your question` |
| Get session summary | Summary (hidden) | `/summary` |
| Create documentation | doc-writer (custom) | `@doc-writer ...` |
| Find security issues | security-auditor (custom) | `@security-auditor ...` |

---

### Temperature Quick Reference

| Task Type | Temperature | Why |
|-----------|-------------|-----|
| Code review, security, documentation | 0.1-0.3 | Need consistency, not creativity |
| General coding help | 0.4-0.6 | Balanced |
| Brainstorming, creative naming | 0.7-0.9 | Want variety |

---

### Tools by Agent Type

| Agent Type | Typical Tools | Typical Permissions |
|------------|---------------|---------------------|
| Code Reviewer | write: false, edit: false, bash: true, read: true, grep: true | bash: ask (except git status: allow) |
| Builder | all tools true | Most: allow, destructive: ask |
| Explorer | write: false, edit: false, bash: false, read: true, grep: true | all: allow (read-only is safe) |
| Researcher | write: false, edit: false, read: true, grep: true, webfetch: true | webfetch: ask |

---

### Configuration File Locations

| Scope | Config file | Agent files |
|------|-------------|-------------|
| Project-specific | `opencode.json` (in project root) | `.opencode/agents/*.md` |
| Global (all projects) | `~/.config/opencode/opencode.json` | `~/.config/opencode/agents/*.md` |

---

### Common Commands

- `opencode agent create` - Interactive wizard to create a new agent
- `opencode config show` - Display your merged configuration
- `opencode config validate` - Check your config files for errors
- `/models` - See available AI models
- `/connect` - Add provider API credentials
- **Tab** - Cycle through primary agents (default keybind)
- `@agent-name` - Manually summon a subagent
- `/summary` - Get a summary of the current session

---

### Minimal Agent Config (Copy-Paste Template)

```markdown
---
description: What this agent does (10-20 words)
mode: subagent  # or "primary"
model: anthropic/claude-sonnet-4-5  # optional
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

Your detailed prompt goes here. Explain the agent's role, methodology, output format, and any constraints.
```

---

## Part 12: Built-In Agent Summary

| Agent | Mode | Purpose | How to Use |
|-------|------|---------|------------|
| build | primary | Your main coding assistant | Just ask; it's the default |
| plan | primary | Analysis and review (no changes unless asked) | Press Tab until you see "plan" |
| general | subagent | Deep research on complex topics | `@general …` |
| explore | subagent | Find things in codebase | `@explore …` |
| compaction | hidden | Auto-summarizes old conversations | Invisible, automatic |
| title | hidden | Generates session titles | Invisible, automatic |
| summary | hidden | Creates session summaries | `/summary` |

---

## Conclusion

OpenCode agents let you build a specialized AI team for your coding workflow.

**Start simple**: Use the built-in Build and Plan agents. Switch between them with Tab. Call specialists like @explore when needed.

**Create custom agents only when needed**: When you find yourself repeatedly asking for the same type of specialized help.

**Focus on three key settings**:
1. `temperature` - predictability vs creativity
2. `tools` - what the agent can do (give only what's needed)
3. `permission` - when the agent needs your approval

**Remember**: The prompt is the most important setting. A clear, specific prompt on a mid-tier model beats a vague prompt on the best model.

**Iterate**: Your first agent config won't be perfect. Use it, observe the results, refine the prompt. After 2-3 iterations you'll have it dialed in.

**Stay safe**: Use `ask` permissions for destructive operations. Give agents only the tools they need. Don't enable `write` for reviewers.

**Don't overcomplicate**: Start with one project-specific agent in Markdown format. Add more as needs emerge. Use `opencode config show` to understand what's actually configured.

You now have everything you need to effectively use and customize OpenCode agents. Start with the 10-minute hands-on at the beginning of this guide, then explore further as needed. Happy coding!

---

## Appendix

### Glossary of Terms

**@-mention** - Typing `@agent-name` to explicitly call a subagent for specialized help.

**Agent** - A specialized AI helper with specific capabilities, instructions (prompt), and permissions. Think of it as a team member with a defined role.

**API** - Application Programming Interface. How different software components communicate. You don't need to understand this deeply to use agents.

**Bash tool** - Allows the agent to run terminal/shell commands (like `ls`, `git status`, `npm install`).

**Configuration** - Settings file(s) that define agents: their descriptions, models, prompts, tools, and permissions.

**Edit tool** - Allows agent to modify existing files (make changes without overwriting whole files).

**Grep tool** - Search for text patterns across multiple files. Like "find in files" but the agent controls the search.

**JSON** - Data format using curly braces and key-value pairs. One way to write agent configurations.

**Markdown** - Simple text formatting language (headings, lists, code blocks). Also used for agent configs with YAML frontmatter.

**Model** - The specific AI engine that powers an agent (e.g., claude-sonnet-4-5, gpt-5). Different models have different capabilities, speeds, costs.

**Permission** - Controls whether agent actions need user approval: `allow` (automatic), `ask` (prompt user), or `deny` (never allowed).

**Primary agent** - An agent that appears in the Tab-switch rotation. You talk to it directly. Build and Plan are primary agents.

**Prompt** - The instructions that define an agent's behavior, expertise, methodology, and output format. The most important config setting.

**Provider** - The company that supplies AI models (Anthropic, OpenAI, OpenCode, etc.). Must be connected via `/connect`.

**Read tool** - Allows agent to view the contents of files.

**Subagent** - A specialized agent invoked via @-mention or automatically. Not in Tab rotation (typically). Explore and General are subagents.

**Temperature** - Setting from 0.0 to 1.0 that controls AI randomness. Low = consistent and predictable. High = creative and variable. Most tasks: 0.4-0.6.

**Tools** - Capabilities an agent can use: write, edit, read, grep, bash, webfetch. Enable only what the agent needs.

**Webfetch tool** - Allows agent to retrieve content from websites (for research, documentation lookup, etc.).

**Write tool** - Allows agent to create new files.

---

### Further Reading

These topics are covered in OpenCode's official documentation:

- **Configuration schema** - Complete reference for all configuration options
- **Providers** - How to set up API connections for different AI services
- **Models** - Understanding available models and their characteristics
- **Permissions system** - Detailed guide to permission syntax and patterns
- **Tools reference** - Complete list of tools and what they do
- **MCP integration** - How agents use Model Context Protocol servers for additional tools
- **Troubleshooting guide** - General OpenCode help and common issues

---

**Document version**: 2.0 (Beginner-friendly, restructured)  
**Last updated**: 2025-03-14  
**Intended audience**: Non-technical users, OpenCode beginners, developers new to agent customization