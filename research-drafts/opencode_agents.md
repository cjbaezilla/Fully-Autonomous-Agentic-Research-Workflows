# OpenCode Agents: Your Complete Guide to Getting Started

---

## What Is OpenCode? (In Plain English)

Let us start with the absolute fundamentals. OpenCode is a coding assistant that lives directly inside your text editor. Think of it like having a knowledgeable programming partner sitting next to you at your desk, ready to help with whatever coding task you are working on at any moment. But here is where it becomes truly powerful: instead of being just one person with general knowledge, you can build a whole team of specialized helpers, each with their own expertise and strengths.

Imagine you are renovating a house. You would not rely on just one person for everything. You would need an architect to plan the work and create blueprints. You would need electricians and plumbers for specialized technical tasks. You would need a general contractor who actually builds and coordinates the work. You would need an inspector to check the work quality and ensure everything meets code. OpenCode agents are exactly like that team. Each agent has a specific role, particular expertise, and a dedicated set of tools they can use. This guide will help you understand how to use the agents that come with OpenCode right out of the box, and more importantly, how to create your own custom agents when you need specialized help that goes beyond the built-in options.

The key insight that makes OpenCode so effective is this: one-size-fits-all AI help is okay, but having a team of specialists is much more powerful for getting consistent, high-quality results that match your specific needs. When you work with a specialized agent that has been given precise instructions and only the tools it needs, you get focused, reliable assistance that understands the nuances of its particular domain.

---

## Understanding the Basics: Models, AI Assistants, and Providers

Before diving deeper into how OpenCode agents work, let's clarify some fundamental concepts that are often confusing when you're new to AI-powered tools. We're talking about three key ideas: what a "model" is, how that differs from an "agent," what an AI coding assistant actually is, and what people mean by "providers." These terms come up constantly, and understanding them will make everything else in this guide make much more sense.

### What's the Difference Between "Model" and "Agent"?

This is one of the most common points of confusion, and it's important to get clear on it because these are fundamentally different things, even though they work together.

Think of it this way: a model is like the engine in a car. It's the raw power source, the thing that actually does the thinking and generates responses. Models like Claude Sonnet, GPT-4, or other AI engines are massive neural networks trained on enormous amounts of text. They can write code, answer questions, analyze documents, and perform many other tasks. But on their own, they're just engines sitting there, waiting for someone to give them a job to do and tell them how to do it.

An agent, on the other hand, is like the complete car with a driver. It's a package that includes the model (the engine), plus a set of instructions (the driver's directions), plus specific capabilities (the steering wheel, pedals, and tools), plus a defined role (are we driving to the grocery store or racing on a track?). The agent is what you actually interact with in OpenCode. When you type a message to the build agent, you're not talking directly to a model-you're talking to an agent that is using a model behind the scenes, following specific instructions, and wielding particular tools to get work done.

Here's a concrete example to make this crystal clear. The model "claude-sonnet-4-5" is a specific AI engine created by Anthropic. You could use that same model for writing poetry, analyzing medical documents, or generating marketing copy-the model itself is just a general-purpose reasoning engine. But when you configure an OpenCode agent with model: "anthropic/claude-sonnet-4-5" and a prompt that says "You are a security auditor who finds vulnerabilities in code" and tools that include read and grep but not write, you've created a specific agent whose job is security review. That same model, with a different prompt and different tools, becomes a documentation writer or a debugging assistant. The model is the capability; the agent is how that capability is specialized, directed, and made safe for a specific purpose.

The relationship can be visualized as a stack. At the foundation is the model-the raw AI reasoning engine. On top of that sits the agent configuration, which includes the prompt (instructions about role and behavior), tools (what the agent can do), permissions (when it needs approval), and temperature (how creative or predictable it is). The agent is what you actually use-it's a configured instance of a model with a specific job description and capabilities.

Why does this distinction matter in practice? Because when you're creating a custom agent, you primarily think about the agent's role and instructions, not about swapping models. The model is an implementation detail-you choose it based on cost, speed, and capability needs, but most of your work goes into crafting a great prompt and selecting appropriate tools. You could use the same excellent agent prompt with different models and get similar results, maybe with slight variations in quality or speed. But you could use the same model for dozens of different agents, each with completely different behaviors and specialties. The specialization happens at the agent level through configuration, not at the model level.

| Aspect | Model | Agent |
|--------|-------|-------|
| **What it is** | A raw AI reasoning engine (Claude, GPT, etc.) | A configured assistant with a specific role and tools |
| **Analogy** | The engine in a car | The complete car with a driver and a destination |
| **What you interact with** | You don't interact with models directly in OpenCode | You talk to agents directly |
| **What it determines** | General capability level, speed, cost | Specific behavior, expertise, allowed actions |
| **Can you have the same model in multiple agents?** | Yes-the same model can power many agents | Each agent gets its own model configuration |
| **What you configure** | You select which model to use from available options | You configure prompt, tools, permissions, temperature |
| **Changes affect** | Performance characteristics (speed, quality) | Behavior, safety, capabilities, output format |
| **Example** | anthropic/claude-sonnet-4-5 | A security auditor that uses Sonnet and only reads code |

The practical takeaway is this: when you're creating an agent, focus on the agent design-what job does this agent need to do, what instructions will make it effective, what tools should it have, how creative should it be? The model selection is important but secondary. Use the default model initially, make your agent excellent through good prompt design, and only change models if you have specific reasons related to cost, speed, or capability requirements.

### What Is an AI Coding Assistant?

An AI coding assistant is a software tool that uses artificial intelligence to help you write, understand, modify, and debug code. Instead of having to figure everything out on your own or search through endless documentation, you have a knowledgeable partner that can assist with programming tasks in natural language. You describe what you want in plain English (or your native language), and the AI helps make it happen.

OpenCode is a specific type of AI coding assistant that works directly inside your text editor. But to understand what makes it special, let's first understand what AI coding assistants do in general.

The core capabilities of an AI coding assistant include several categories. First, code generation-this is what most people think of first. You say "create a function that validates email addresses" and the AI writes working code. But AI coding assistants do much more than just write new code from scratch. They also explain code-you can point to a complex function and ask "what does this do?" and get a clear explanation. They can translate code from one language to another. They can refactor code-take working code and reorganize it to be cleaner, more efficient, or follow better patterns without changing what it does. They can find and fix bugs-you describe a problem or show an error, and they help identify the cause and suggest a fix. They can write tests-generate unit tests, integration tests, or edge case coverage for existing code. They can document code-add comments, write function documentation, or create README files that explain how a project works. They can search through codebases to find where things are defined and understand how different parts relate. And they can answer questions about programming concepts, best practices, and how to use specific APIs or frameworks.

What makes modern AI coding assistants effective is that they understand context. They can see the files you have open, they can understand the structure of your project, they can read configuration files and dependencies. This context awareness means they can give relevant, specific help rather than generic advice. When you ask "how should I handle authentication in this project?" a good AI assistant will look at how authentication is currently implemented (if at all), what frameworks and libraries you're using, and give advice that fits your actual codebase rather than just reciting general principles.

OpenCode takes this further by introducing the concept of specialized agents. Instead of having one assistant that tries to do everything reasonably well, OpenCode lets you build a team where each member has a specific specialty. You have a build agent that creates and modifies code, a plan agent that reviews and analyzes, and specialists for security, documentation, debugging, research, and more. This mirrors how real software teams work-you don't ask the same person to both build new features and perform security audits; you have specialists who excel in their domains.

The workflow with an AI coding assistant is conversational and iterative. You don't just give one command and get perfect results. You have a back-and-forth: you ask for something, the AI produces code or analysis, you review it, you might ask for changes or clarification, you might switch to a different agent for a second opinion, you might call in a specialist for a particular aspect. It's a collaborative partnership. The AI is not autonomously building entire applications without your guidance-it's assisting you, doing the tedious parts, offering suggestions, and helping you be more productive. You remain in control, making decisions and reviewing work.

This collaborative aspect is crucial. AI coding assistants are not replacements for developers. They're force multipliers. They handle the mechanics of code generation, reduce the time spent on routine tasks, help overcome "blank page" syndrome by giving you something to start from, and surface information you might not know to look for. But they don't replace your judgment, your understanding of the problem domain, your architecture decisions, or your responsibility for the final code. You're the lead developer; the AI is your capable assistant who can work fast but needs direction and oversight.

The technology behind AI coding assistants is based on large language models-neural networks trained on vast amounts of text, including lots of source code. These models learn patterns of how code is written, how problems are solved, what good code looks like versus bad code, and how to translate natural language descriptions into working code. The models don't "understand" code the way humans do-they're detecting statistical patterns and generating likely completions. But because they've seen so much code, their suggestions are often remarkably accurate and useful.

What limits AI coding assistants? They can make mistakes-they might generate code that looks correct but has subtle bugs, they might misunderstand requirements, they might use outdated APIs or insecure patterns. They have knowledge cutoffs-they don't know about very recent library versions or events after their training data. They can be confidently wrong, presenting incorrect information as if it's certain. They don't replace your need to understand what you're doing-you still need to review, test, and validate their output. They're not good at tasks requiring deep reasoning about business logic you haven't explained clearly. And they can't actually run code to see if it works-they're predicting what code should exist, not verifying it executes correctly. That's where the cycle of building and reviewing comes in.

The best way to use an AI coding assistant is to treat it as a skilled but inexperienced junior developer who works very fast. You give clear direction, you review their work carefully, you catch errors they make, you ask them to redo things that aren't right. Over time, as you learn how to communicate with them effectively and as the models improve, this partnership becomes increasingly productive. But the partnership requires your engagement-the AI won't figure out what you really need without your guidance.

### What Is a "Provider"?

When you hear people talk about "providers" in the context of AI coding assistants like OpenCode, they're referring to the companies that create and host the AI models you can use. Anthropic, OpenAI, Google (with Gemini), and OpenCode itself are all providers. They're the ones who train massive neural networks on enormous datasets, who operate the computer clusters that run these models, and who charge for access to their models through APIs.

Understanding providers is important because they affect cost, performance, features, and how you configure OpenCode. Each provider offers multiple models with different capabilities, price points, and characteristics. And you need credentials (API keys) from each provider to access their models.

Let's break down what providers do and why they matter. A provider is essentially a service that gives you access to AI models over the internet. You don't run these models on your own computer (except for local options like Ollama, but that's a different story). You send a request over the internet to the provider's servers, their servers run your request through their AI model, and they send back the result. This is called cloud-based AI, and it's what most people use because running these models yourself requires extremely powerful and expensive hardware.

The provider determines several important characteristics. Different providers have different pricing structures-some charge per token (roughly per word), some have monthly subscriptions, some have hybrid models. Prices vary dramatically between models from the same provider and between providers. Some providers offer models optimized for specific tasks-code generation, creative writing, mathematical reasoning, vision, etc. Some providers have different speed characteristics-some models respond in seconds, others take minutes for complex reasoning. Some providers have usage limits or rate limits on how many requests you can make in a given time period. And each provider has its own API format, authentication methods, and sometimes unique features.

When you use OpenCode, you need to connect it to one or more providers via their API keys. This is done through the /connect command. You enter your API key for Anthropic, for OpenAI, or whichever providers you have accounts with. Once connected, OpenCode can use models from those providers. In your agent configuration, you specify which model to use in the format provider/model-id. For example, anthropic/claude-sonnet-4-5 means use the Claude Sonnet model from Anthropic. openai/gpt-5 means use GPT-5 from OpenAI. The provider name is the prefix before the slash, and it must match a provider you've connected.

The provider/model naming format is consistent throughout OpenCode. You'll see it in agent configuration files, in the /models command output, in error messages. This format makes it explicit which company's model you're using and which specific model variant. It's important because different providers' models behave differently, even if they have similar capabilities. A prompt engineered for Claude might need adjustment for GPT. Temperature scales might have different effects. Cost structures differ.

Here's why you need to understand providers when configuring agents. Suppose you create a security auditing agent. You might want to use the most capable model available because security analysis requires thorough reasoning. That might mean using anthropic/claude-opus-4 from Anthropic. But Opus is expensive and slower. So you might have a different agent for quick tasks that uses anthropic/claude-haiku-4-5, which is cheaper and faster but less thorough. Both are from the same provider (Anthropic), but they're different models with different trade-offs. Or you might have some agents using OpenAI models and others using Anthropic models, depending on your preferences and which providers you've connected to.

Providers also differ in what models they offer and their specialties. Anthropic is known for models that are particularly good at safety, instruction following, and reasoning tasks. OpenAI's models are widely used and have strong all-around capabilities, with some variants optimized for specific tasks. OpenCode's own models are optimized for coding tasks. Each provider may release new models at different cadences, may have different terms of service, may offer different levels of support, and may have different data privacy policies. For professional use, these factors matter.

The common providers you'll encounter are Anthropic (claude models), OpenAI (GPT models), and OpenCode (their own hosted models optimized for coding). Sometimes people also use local providers like Ollama or LM Studio to run models on their own hardware, which requires custom provider configuration. But for most users, the main providers are the major cloud services.

In practice, you don't need to worry too much about the provider distinction once you've connected your API keys. You just specify the model in the format provider/model-id, and OpenCode handles the rest. But understanding providers helps you make informed choices about which models to use for which agents, helps you understand your costs and how they're calculated, and helps you troubleshoot when something doesn't work (like using a model from a provider you haven't connected).

---

## Your First 10 Minutes with OpenCode Agents

Before you read anything else in this document, we strongly encourage you to open OpenCode and try these simple steps. This hands-on experience will give you a concrete understanding of how agents work, which will make the rest of this guide much clearer.

First, press the Tab key on your keyboard. This action cycles through your available agents. You should see names like build and plan appear in the interface as you press Tab multiple times. The build agent is your default helper, so it is usually what you see first. Without switching agents, type this message: "Create a simple Python script that says 'Hello World' and saves it to hello.py." Watch what happens. The build agent should spring into action and create that file for you, complete with the proper Python code that prints the greeting.

Now press Tab again until you see plan appear in the interface. This switches your conversation to the plan agent. Type: "Review the hello.py file we just created and suggest any improvements." The plan agent will read the file you just created and provide thoughtful feedback about how it could be better. You might see suggestions about adding error handling, using proper shebang lines, or following Python conventions.

Press Tab once more to go back to the build agent. Type: "Make the changes that Plan suggested." The build agent should now take the feedback from the plan agent and implement those improvements, editing the hello.py file to address the issues that were identified.

That simple back-and-forth is the core workflow of OpenCode agents. You now understand 80% of how agents work in practice. The build agent does the actual work of creating and modifying code. The plan agent reviews the work and provides analysis and suggestions. You switch between them using the Tab key. Do not worry about everything else in this document yet. Come back to it after you have completed those steps and want to learn more about the full capabilities.

---

## Part 1: Understanding the Built-In Agent Team

OpenCode gives you seven agents right out of the box when you first install it. You will use two of these constantly in your everyday work, you will call two others occasionally when you need their special skills, and three work automatically in the background without you needing to think about them.

### The Two You Use Every Day

These are the workhorses of your OpenCode experience. You will interact with them more than any other agents.

The build agent is your main builder and default helper. It writes code, edits files, creates new things from scratch, and makes changes to your existing project. In practical terms, the build agent actually creates and modifies code files based on your instructions. You use it for everyday coding tasks such as adding new features, fixing bugs, refactoring code to make it cleaner, and adding functionality to your project. The build agent can write files, edit files, run commands in your terminal, search through your codebase, and even fetch content from websites if needed. It basically has all the capabilities you would expect from a hands-on coding assistant. You talk to the build agent simply by asking it to do something. Since it is the default agent, you often do not even need to switch to it. Just type your request and the build agent responds. If you have switched to another agent, press Tab until you see build in the interface. An example of how you might use it: "Add user authentication to this application" or "Extract this duplicated code into a reusable function that I can call from multiple places."

The plan agent is your quality control specialist. It analyzes and reviews code and plans but does not make changes unless you specifically ask it to. Think of the plan agent as your safety net and second pair of eyes. Its role is to review, analyze, and plan out approaches without modifying anything. You use it for code review when you want someone to check your work, for planning features before you start building them, for analyzing problems to understand their root cause, and for understanding complex code that someone else wrote. The plan agent can read files and analyze them thoroughly, but its modification tools require your explicit approval before it can change anything. You talk to the plan agent by pressing Tab until you see plan in the interface. An example of how you might use it: "Review my recent changes for security issues before I push to production" or "What is wrong with this function? It is not working as I expected."

There is a smart workflow pattern that emerges from having these two agents. Start with the build agent to create something or make a change. Then switch to the plan agent to review what was built. Then switch back to the build agent to implement the improvements that the plan agent suggested. This back-and-forth conversation between builder and reviewer is the heart of effective agent use and mimics how human development teams work together.

### The Two You Call Occasionally

These agents are not in the Tab rotation that you cycle through. Instead, you call them by typing their name preceded by the @ symbol in your messages. They are specialists you bring in when you need their particular expertise.

The explore agent is your detective and codebase navigator. It searches through your codebase to find things and understand the overall structure. Its specialty is finding files, mapping dependencies between different parts of the code, and understanding how components relate to each other. You use it when you need to know where something is located, such as "Where is the payment processing code?" or "What files handle user authentication in this project?" or "How is the database structured and which tables relate to each other?" The explore agent is perfect for new codebases where you are still finding your way around, for finding where a particular function or feature is defined, and for understanding the overall organization of a project. You call it by typing something like @explore find all files related to user management or @explore show me the dependency graph for the payment module.

The general agent is your researcher who conducts deeper investigations requiring multiple steps and information gathering. Its specialty is multi-step research, synthesizing information from multiple places, and answering complex questions that require investigation rather than simple file operations. You use it when you need to know best practices for a particular technology, when you need to understand how other teams in your codebase handle a certain problem, or when you need to research and compare different approaches to a technical challenge. For example, you might ask @general what are the standard approaches for file upload validation in web applications or @general research database connection pooling strategies and tell me which one works best for high-traffic sites. The general agent will conduct a thorough investigation and come back with synthesized findings.

---

## Part 2: How Agent Communication Actually Works

Understanding how to interact with agents is crucial for effective use. There are two distinct patterns for getting agents to help you, and each serves a different purpose.

Switching, which you do with the Tab key, means you change which agent you are talking to directly. The conversation continues in the same thread, but now you are addressing a different specialist. It is exactly like walking over to a different colleague's desk and having a conversation with them instead of the person you were just talking to. When you switch to the plan agent, you are now having a conversation with the plan agent, and it remembers what you were discussing. Switching is how you use primary agents that are meant to be your main partners in the workflow.

Calling, which you do with the @-mention syntax, means you ask your current agent to get help from a specialist behind the scenes. It is like saying "Hey build, I need you to ask explore to find something for us." You stay in the same conversation with the build agent, but it will summon the explore agent as a subagent to handle the part of the task that requires exploration. The explore agent works independently and returns its findings to the build agent, which then incorporates those findings into its response to you. You use calling when you need your primary agent to leverage specialist capabilities without you having to switch away.

When should you use which approach? If you want a second opinion on build's code, switch to plan because you want to have a direct conversation with the reviewer. If you need build to create something, stay with build because it is the builder and that is its primary role. If you need to find where something is in the codebase, call @explore because finding things is explore's specialty and it can do that faster than build figting through the code itself. If you need deep research on a topic, call @general because general is designed for thorough investigation. If you are planning architecture, switch to plan or consider creating a dedicated architecture agent because that requires analytical thinking and planning expertise. The pattern emerges: switch when you want to adopt the perspective of that specialist for a while, call when you want your current agent to leverage a specialist for a specific subtask.

When you call a subagent, several things happen in sequence. First, you type something like @explore find all database files. The explore agent receives that as its task and works independently, often creating its own mini-conversation that OpenCode calls a subtask. You might actually see this subtask appear if you use keyboard shortcuts to navigate between conversation threads. The explore agent finishes its exploration and returns the results to your main agent. Your main agent then uses those results to form its complete answer to you. You can watch subtasks in action by using the Ctrl+H keyboard shortcut by default to enter child sessions, and then arrow keys to navigate between them. This is useful if a subagent seems stuck or if you want to provide guidance and steer its investigation.

Sometimes you do not even need to manually call subagents. The system can automatically invoke them when it determines that a specialist would be helpful. If you ask build "where is the login code?" the build agent might automatically call explore because finding things in the codebase is explore's specialty and it makes sense to delegate that part of the request. You will usually see a message indicating that this happened. Do not fight this automatic behavior - let the system do its job. But if you want more explicit control, manually specify @explore to ensure the right agent handles that part of the task and you know exactly what is happening.

---

## Part 3: Should You Create a Custom Agent?

Before you rush to create custom agents, it is important to understand that you do not need custom agents to use OpenCode effectively. The built-in build and plan agents handle 90% of everyday coding tasks. The @explore and @general subagents cover most of the remaining scenarios. You can be very productive with just these four agents and the Tab-switching workflow.

Create a custom agent only when you notice a clear pattern in your work: you find yourself repeatedly asking for the same type of specialized help, and you want consistent, focused results every time rather than having to explain the same requirements over and over. Custom agents are force multipliers for recurring needs.

There are specific signs that indicate you actually need a custom agent. Create a custom agent when you find yourself repeatedly doing these things: always asking for security reviews and wanting consistent security-focused results (this suggests a security auditor agent), frequently writing documentation in a specific format (this suggests a documentation writer agent), running the same quality checks before every commit (this suggests a quality gatekeeper agent), checking if your code follows your team's style guide repeatedly (this suggests a style enforcer agent), or always optimizing for performance and wanting systematic performance analysis (this suggests a performance tuner agent). If it is a one-time question or a task you do only once, just ask directly. Custom agents are for recurring needs where you want consistency and specialization. They save you time in the long run by codifying your expertise and requirements into a dedicated assistant.

---

## Part 4: Creating Your First Custom Agent

The easiest way to create a custom agent is to use the interactive wizard that OpenCode provides. This wizard holds your hand through the entire agent creation process and ensures you do not make formatting mistakes. Here is how you start it: open your terminal or command line and type opencode agent create and press Enter.

The wizard will ask you five questions to configure your new agent. The first question asks where you want to save the agent configuration. You can choose globally, which means the agent will be available in all your projects, or you can choose just for the current project, which means the agent will only exist in the project you are currently working on. While you are learning, we recommend choosing the project option because it keeps things contained and you can always change to global later if you find the agent is useful across multiple projects. There is no penalty for starting with project-specific and upgrading later.

The second question asks for the agent name. This is a short identifier with no spaces. Use hyphens to separate words, such as security-checker or doc-writer or test-runner. Keep it descriptive but short. You will type this name when you call the agent with @-mention, so you want it to be memorable and easy to type. Avoid special characters beyond hyphens and stick to lowercase letters for consistency.

The third question asks for a description. This is a brief explanation of what the agent does, ideally in 10 to 20 words. A good description might be "Reviews code for security vulnerabilities and suggests specific fixes" or "Creates and maintains project documentation following standards." A bad description would be "A code agent that does security things" because it is too vague and does not tell you or the system what the agent actually specializes in. The description appears in agent selection lists and helps you remember what each agent does when you have many custom agents.

The fourth question asks you to choose a model. This determines which AI powers your agent. The wizard will suggest a default model, and you should accept that default unless you have a specific reason to choose something else. You can always change the model later by editing the configuration file. Most users are fine with the default, which is typically a good balance of capability and cost.

The fifth question asks you to select tools. Tools are the actions your agent can take. You use the spacebar to toggle each tool on or off. The available tools include write, which allows creating new files; edit, which allows modifying existing files; bash, which allows running terminal commands; read, which allows reading file contents; grep, which allows searching for patterns across files; and webfetch, which allows retrieving content from websites. You should give agents only the tools they absolutely need to do their job. This is a safety principle: a reviewer does not need write or edit capabilities because it should not be modifying files. A documentation writer probably needs read, write, and edit but does not need bash because it does not run system commands. Fewer tools mean more focused behavior and better safety because the agent cannot accidentally do things outside its intended purpose.

The wizard creates the configuration file in the correct location automatically based on your answers. This prevents formatting errors and placement mistakes that commonly happen when creating configuration files by hand. Start with the wizard - it is the reliable way to create your first agent.

---

## Part 5: Understanding Agent Configuration Files

A configuration file is the settings file that tells OpenCode everything about an agent: what the agent is like, what it can do, how it behaves, and what instructions it follows. Think of it like filling out a job description for an employee. You are specifying the job title, the responsibilities, the tools the employee gets to use, how they should behave, and what working style they should adopt.

The configuration file might include fields such as: the agent's name, which is security-checker; the job description, which is find security problems in code; the tools the agent can use, which might be can read anything but cannot modify files; the behavior guidelines, which could be be thorough, rate findings by severity, always suggest fixes; and the working style, which might use temperature 0.1 for very consistent and reliable output rather than creative variations.

Configuration files live in specific locations depending on whether they are available to just one project or to all your projects. Project-specific configuration lives in a folder called .opencode inside your project directory. The agents you create for just that project go in .opencode/agents/your-agent.md. Global configuration that applies to all your projects lives in your operating system's configuration directory, typically ~/.config/opencode/agents/your-agent.md on Unix-like systems or the equivalent in AppData on Windows. There is also the option of having a single opencode.json file either in your project root or in the global config directory that contains definitions for multiple agents. The wizard helps you choose and creates files in the right place. As a general recommendation, use the wizard because it creates files in the right location with proper formatting. If you need to edit later, you will know exactly where to look.

There are two ways to write agent configurations, and both work equally well. The wizard gives you the option to choose either format. Here is how they differ. The choice between JSON and Markdown (with YAML frontmatter) is not just about syntax-it affects readability, maintainability, collaboration, and your day-to-day experience managing agent configurations. Let's explore the practical considerations in depth.

**JSON Format: The Structured Single-File Approach**

JSON (JavaScript Object Notation) is a data interchange format that uses strict syntax: curly braces for objects, square brackets for arrays, quotes around all string keys and values, and commas to separate elements. In OpenCode, you put all your agent definitions in a single opencode.json file. The structure looks like this:

```json
{
  "agent": {
    "security-checker": {
      "description": "Reviews code for security vulnerabilities",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.2,
      "tools": {
        "write": false,
        "edit": false,
        "bash": false,
        "read": true,
        "grep": true,
        "webfetch": false
      },
      "permission": {
        "edit": "ask"
      },
      "prompt": "You are a senior security auditor..."
    }
  }
}
```

JSON's main advantage is having everything in one place. This appeals to people who like consolidation-one file to rule them all. If you're using version control like Git, you can track changes to all your agents in a single commit, see diffs across multiple agents, and manage agent definitions as a cohesive unit. JSON is also the format you'll see most often in programmatic contexts and API documentation, so if you're generating agent configurations from scripts or tools, JSON is more straightforward.

However, JSON has significant drawbacks for humans editing it by hand. The syntax is unforgiving-a missing comma, an extra trailing comma, mismatched braces, or unquoted keys will cause the entire file to fail parsing, and error messages are often cryptic about where the problem is. The structure becomes deeply nested with all those braces, making it visually hard to scan. Multi-line strings are awkward-you either have to concatenate lines or deal with escaped newlines. And prompts become long single-line strings or ugly concatenated messes that are difficult to read and edit.

**Markdown with YAML Frontmatter: The Human-Friendly Separate-Files Approach**

In the Markdown format, each agent gets its own .md file in the .opencode/agents/ directory. The file starts with YAML frontmatter-the section between triple-dashed lines-which contains structured settings. Everything after the closing triple-dash is the prompt, written in regular Markdown with full formatting capabilities. Here's what it looks like:

```yaml
---
description: "Reviews code for security vulnerabilities"
mode: "subagent"
model: "anthropic/claude-sonnet-4-5"
temperature: 0.2
tools:
  write: false
  edit: false
  bash: false
  read: true
  grep: true
  webfetch: false
permission:
  edit: "ask"
---

You are a senior security auditor with 15 years of experience. Your task is to find security vulnerabilities in code.

**Your methodology:**
1. Check input validation for injection attacks
2. Review authentication mechanisms
3. Examine authorization controls
4. Look for secrets in code
5. Verify encryption usage

**Output format for each finding:**
- **Severity:** Critical/High/Medium/Low
- **Location:** file:line
- **Problem:** Clear description
- **Fix:** Specific code example

Always provide a CWE or OWASP reference when applicable.
```

The advantages of Markdown format are substantial for day-to-day human editing. Files are readable and editable in any text editor. YAML is much more forgiving than JSON about formatting-you don't need quotes around keys (though you can use them), you can add comments to explain settings, and the structure is visually clear with indentation. Most importantly, the prompt lives in its natural format-you can use proper paragraphs, headings, bullet lists, code examples, and all the formatting that makes instructions clear. You can open the file and immediately understand what the agent does and how it works. Each agent has its own file, so you can edit one agent without worrying about breaking others. This separation also makes it obvious which file corresponds to which agent-the filename is the agent name.

The downsides are having more files to manage and the need to ensure YAML syntax is correct (though YAML is more forgiving than JSON, it still has rules about indentation). If you have dozens of agents, you'll have dozens of files, but in practice most people have only a handful of custom agents, so this isn't a problem. One file per agent also makes collaboration easier-multiple people can edit different agents without merge conflicts.

**A Detailed Comparison**

Let's examine the two formats across several practical dimensions to help you decide which fits your workflow.

| Dimension | JSON Format | Markdown/YAML Format |
|-----------|-------------|---------------------|
| **File organization** | Single opencode.json file containing all agents | One .md file per agent in .opencode/agents/ |
| **Human readability** | Low-dense syntax, hard to scan prompts | High-clean structure, readable prompts with formatting |
| **Edit safety** | One syntax error breaks all agents | Syntax error only affects that single agent file |
| **Prompt editing** | Awkward-multi-line strings or concatenation | Natural-just write in Markdown with full formatting |
| **Version control** | Single file-one commit changes all agents | Multiple files-agents tracked separately |
| **Comments** | Not supported in JSON | YAML supports comments with # |
| **Code examples in prompts** | Hard to include cleanly | Easy with Markdown code blocks |
| **Programmatic generation** | Very easy-JSON is standard data format | Possible but requires YAML + Markdown handling |
| **Splitting large prompts** | Possible via external file references | Not applicable-prompt is in same file |
| **Discoverability** | Need to search within the JSON file | Agent name = filename, easy to browse |
| **Wizard support** | Wizard can create either format | Wizard can create either format |
| **Migration between formats** | Manual conversion required | Manual conversion required |

**Decision Framework: Which Format Should You Choose?**

Ask yourself these questions to find your best fit.

Choose JSON format if: you plan to generate agent configurations programmatically from scripts or other tools; you strongly prefer having all agent definitions consolidated in one location for easy overview; you're sharing configurations with a team that expects a single file; or you're copying examples from documentation that predominantly uses JSON. JSON also works better if you need to include agent definitions in a repository where you want to minimize file count (though this is rarely a compelling reason).

Choose Markdown format if: you'll be editing prompts frequently and want them to be readable and well-formatted; you prefer the clarity of one file per agent; you want to use Markdown formatting like headings, lists, and code blocks in your prompts; you want to be able to add comments explaining your configuration choices; or you appreciate being able to quickly glance at a file and understand what that agent does without parsing JSON structure. Markdown format is generally more pleasant for human-centered workflows.

**Real-World Implications of Your Choice**

The format you choose affects your everyday workflow in tangible ways. With JSON, when you need to tweak an agent's prompt, you open one file, navigate to the right agent object in the nested structure, find the "prompt" field (which might be a single long line or escaped newlines), and edit within JSON syntax constraints. With Markdown, you open a file named after the agent, scroll past the YAML settings (which are clearly labeled and scannable), and edit the prompt in clean Markdown with proper formatting. The difference in ease-of-use over many small edits is substantial.

With JSON, if you accidentally delete a comma or close brace, none of your agents load and you get a generic error. You have to carefully check the entire file to find the syntax error. With Markdown, if you have a YAML indentation problem in one agent file, only that agent fails to load; the others still work. This isolation keeps you productive rather than bringing everything to a halt.

Collaboration differs too. With JSON, multiple people editing the same file need to coordinate to avoid merge conflicts-if two people add agents to the same opencode.json, they have to manually merge their changes. With Markdown files, two people can add different agents by creating separate files and there's no conflict at all (unless they edit the same agent file, but that's a much more localized conflict).

The prompt presentation itself matters. In JSON, your prompt is a string value. If you want to include examples with code formatting, you need to escape things carefully or include literal newlines that can be hard to read in the JSON file. In Markdown, your prompt is regular text where you can use **bold**, bullet lists, numbered lists, code blocks with triple backticks, and all of Markdown's features. This makes prompts much easier to write and maintain because you can organize them clearly with headings like "Your methodology:", "Output format:", "Important rules:". The agent actually sees the Markdown as plain text, but the structure helps you as the author.

**Mixing Formats: The Best of Both Worlds?**

One pleasant surprise is that OpenCode doesn't force you to choose one format exclusively. You can mix both approaches. You can have your global configuration in opencode.json with some agents defined there, and also have project-specific agents in .opencode/agents/*.md files. OpenCode loads all sources and combines them, with project-level agents overriding global ones when there are conflicts. This means you could start with the wizard using Markdown format for ease of editing, but keep a few stable agents in JSON if you prefer it that way. There's no penalty for mixing.

**Practical Recommendations**

For most users, especially those new to OpenCode, we strongly recommend starting with Markdown format. The human-friendly experience makes it easier to learn, easier to edit, and harder to break. The wizard defaults to Markdown in many installations because it's the more beginner-friendly option. You can always migrate to JSON later if you have a compelling reason, but there's rarely a need-Markdown works perfectly well even for large setups with dozens of agents.

If you do choose JSON, be meticulous about syntax. Use a JSON validator if you're unsure. Consider using an editor with JSON syntax highlighting and validation. Keep your agents' prompt strings separate in external files if they're large; reference them with `"prompt": "{file:./prompts/security-auditor.txt}"` to keep the JSON manageable. And be prepared to deal with escaping and formatting issues when prompts contain quotes or special characters.

Regardless of format, use the wizard to create your first agents. The wizard handles the formatting details correctly and places files in the right locations. You can then edit the generated files, confident that the structure is correct. Once you understand the format, you can edit by hand with confidence.

---

## Your First 10 Minutes with OpenCode Agents

The first option is JSON format, where you put all your agent definitions in a single opencode.json file using JSON syntax with curly braces, quotes, and commas. In this format, you might have a JSON object that contains an agent key, and under that you define your custom agent with its properties. The JSON looks something like this:

```json
{
  "agent": {
    "security-checker": {
      "description": "Reviews code for security vulnerabilities",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.2,
      "tools": {
        "write": false,
        "edit": false,
        "bash": false,
        "read": true,
        "grep": true,
        "webfetch": false
      },
      "permission": {
        "edit": "ask"
      },
      "prompt": "You are a senior security auditor..."
    }
  }
}
```

Or you can reference a separate prompt file:

```json
{
  "agent": {
    "doc-writer": {
      "description": "Creates and maintains project documentation",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.4,
      "tools": {
        "write": true,
        "edit": true,
        "read": true,
        "grep": true,
        "bash": false,
        "webfetch": false
      },
      "prompt": "{file:./prompts/doc-writer.txt}"
    }
  }
}
```

You would use JSON format if you like having everything in one place rather than many separate files, if you are sharing configurations via Git and want version control on a single file, if you are copying examples from documentation since many examples use JSON, or if you need to generate agent configurations programmatically. The downsides are that JSON is harder to read with all the curly braces everywhere, and one small syntax error like a missing comma can break the entire file.

The second option is Markdown format, where each agent gets its own .md file in the .opencode/agents/ folder. The file starts with YAML frontmatter, which is the part between the triple-dashed lines, and then your prompt instructions follow in regular Markdown. The frontmatter contains the structured settings like description, mode, model, temperature, tools, and permissions. Everything after the closing triple-dash is the prompt - the natural language instructions that define how the agent behaves.

Here is a complete example of a Markdown agent configuration file:

```yaml
---
description: "Reviews code for security vulnerabilities"
mode: "subagent"
model: "anthropic/claude-sonnet-4-5"
temperature: 0.2
tools:
  write: false
  edit: false
  bash: false
  read: true
  grep: true
  webfetch: false
permission:
  edit: "ask"
---

You are a senior security auditor with 15 years of experience. Your task is to find security vulnerabilities in code.

**Your methodology:**
1. Check input validation for injection attacks
2. Review authentication mechanisms
3. Examine authorization controls
4. Look for secrets in code
5. Verify encryption usage

**Output format for each finding:**
- **Severity:** Critical/High/Medium/Low
- **Location:** file:line
- **Problem:** Clear description
- **Fix:** Specific code example

Always provide a CWE or OWASP reference when applicable.
```

You would use Markdown format if you want human-readable files that you can open and understand at a glance, if you have many custom agents and want them organized separately in their own files, if you want to write detailed prompts with formatting like headings and lists (which you can do in Markdown), or if you prefer editing agent instructions in a natural way without worrying about JSON syntax. The downside is more files to manage, though that rarely becomes a problem in practice. The good news is that you can mix both formats if you want. OpenCode loads agents from all sources and combines them. Start with one format that feels comfortable. You can always switch later if your needs change.

---

## Part 6: Agent Configuration Settings Explained

This section explains each setting you can use in an agent configuration file. We will start with the essential settings that everyone should understand first. Then we will cover important settings that you should get right for your agents to behave properly. Finally we will mention other useful settings that you can learn as you become more advanced.

### Essential Settings (Use These First)

The description setting tells everyone what your agent does. It is required for every agent. The ideal length is 10 to 20 words. A good example would be "Reviews code for security vulnerabilities and suggests fixes." A bad example would be "A code agent that does security things" because it is too vague and does not convey specific expertise. You see the description in agent selection lists when you press Tab to switch agents. You see it when agents automatically summon subagents, and you see it in the output of the /agents command that lists all available agents. Make your description clear and descriptive because future you will thank you when you have ten custom agents and need to remember what each one does without opening each file.

The mode setting is critical because it determines how your agent can be used. There are three options. Primary means the agent appears in the Tab rotation. You switch to it directly and talk to it as a main partner. Build and plan are primary agents. Subagent means the agent is only available via @-mentions or automatic calls by other agents. It does not appear in the Tab rotation. This is for specialists you call when needed, like @explore and @general. The all mode means the agent can be both primary and subagent, giving maximum flexibility. As a rule of thumb, build and plan should be primary because you switch to them directly. Specialists like security-checker should be subagent because you call them when you need their specific skill. Most custom agents you create will be subagent rather than primary because you already have build and plan as your primary coding partners.

The model setting determines which AI engine powers your agent. The format is provider/model-name. Examples include anthropic/claude-sonnet-4-5 or openai/gpt-5 or opencode/gpt-5.1-codex. If you do not set this, the agent uses whatever model you have selected globally, which you can see with the /models command. You would set a specific model per-agent if you want different models for different purposes. For example, you might use fast and cheap models like Haiku or GPT-5 Nano for quick lookups and frequent subagents because speed matters more than depth. You might use balanced models like Sonnet or GPT-5 for everyday coding assistance because they offer good all-around capability. And you might use the smartest and most expensive models like Opus or GPT-5 Codex for complex architecture work or critical code where maximum reasoning ability is worth the cost. Do not overthink this - use the default model that the wizard suggests. Change only if you have a specific reason related to cost, speed, or capability needs. You can always adjust per-agent later as you learn more.

The prompt setting is the most important setting of all. The prompt defines your agent's expertise, personality, methodology, and constraints. A great prompt creates a great agent. A vague prompt creates confusion and inconsistent results. The prompt goes in different places depending on your configuration format. In JSON, you can either reference a separate file with "prompt": "{file:./prompts/build.txt}" or put the prompt inline with "prompt": "You are a builder...". In Markdown format, everything after the --- frontmatter section is the prompt. What makes a good prompt? First, a clear role: "You are a senior security auditor with 10 years of experience" is good, while "You review code" is bad because it is too vague. Second, specific methodology: "Your process: 1) Check input validation, 2) Review authentication, 3) Look for secrets, 4) Rate findings by severity" gives the agent a clear procedure to follow. Third, output format: "For each issue: Severity: [rating], Location: [file:line], Problem: [description], Fix: [code example]" tells the agent exactly how to structure its response. Fourth, constraints: "Do not modify files. Only analyze and report" is crucial for reviewers so they know their limits.

A bad prompt example would simply say "You are a code reviewer. Check code and give feedback." This gives no guidance about what to check for, how to structure the feedback, or what constitutes good feedback. A good prompt example provides comprehensive guidance. It might say "You are a senior code reviewer with expertise in security and performance. Your review covers: 1) Security: injection attacks, XSS, authentication flaws, secrets in code; 2) Error handling: missing try-catch, unhandled edge cases; 3) Performance: inefficient loops, unnecessary computations; 4) Code quality: naming, duplication, complexity, maintainability. For each issue found: provide Severity (Critical/High/Medium/Low), Exact location (file:line), Clear explanation of the problem, and Specific fix with code example. Output format should have three sections: Summary with overall assessment, Issues Found with detailed entries, and Positive Observations encouraging good practices. Be direct and specific. Do not just say 'this is bad' - explain why and show better." This prompt gives the agent a clear role, specific checklist, output format, and guidance on tone.

Here is why prompt matters more than model: a well-crafted prompt on a cheaper model often outperforms a vague prompt on an expensive model. The model provides capability, but the prompt directs that capability. Investing time in your prompt is the single most effective way to improve your agent's performance.

### Important Settings (Get These Right)

The temperature setting is the creativity dial, and it is the single most misunderstood setting. It controls how predictable or creative the AI's output is. The range goes from 0.0, which is very predictable and consistent, to 1.0, which is very creative and unpredictable.

**Temperature Examples:**

Temperature between 0.1 and 0.3 is low, meaning consistent and reliable. At this range, the output is predictable, follows conventions, and stays on track. This is perfect for code review, security scanning, documentation, and anything requiring accuracy. The same prompt given to the agent will produce very similar results every time. For example, if you ask for "a good function name for login," you will get conventional names like `login`, `authenticate`, or `signIn`.

Temperature between 0.4 and 0.6 is medium, offering a balance with some variation but still reasonable results. This is perfect for everyday coding help and for generating options to choose from. This is a good default for builders who need creative problem solving but still want reasonable results. Using the same example, you might get `verifyCredentials`, `beginSession`, or `startLoginFlow`.

Temperature between 0.7 and 0.9 is high, meaning creative and experimental. At this range, the output is wild, varied, sometimes too weird, and may go off-topic. This is perfect for brainstorming sessions, creative naming, and exploring unusual approaches. It is not suitable for production work or analysis where you need reliability. The same example might yield `portalToIdentity`, `theGateKeeperAwakens`, or `grantMeAccess` - interesting but not practical for real code.

**Common mistakes people make** include using high temperature like 0.8 for code review, which produces entertaining but unreliable results that miss serious issues. Or using low temperature like 0.2 for brainstorming, which produces the same ideas every time and limits creative exploration. Start with 0.5 as a baseline and adjust based on your results. If the agent is too repetitive and giving the same suggestions every time, increase temperature slightly. If the agent is too weird or goes off-topic, decrease temperature. For analysis and review tasks, aim for 0.1 to 0.3. For general coding assistance, aim for 0.4 to 0.6. For creative work like brainstorming, aim for 0.7 to 0.9.

The tools setting is simply on-off switches for each capability. The available tools are write, which allows creating new files; edit, which allows modifying existing files; read, which allows reading file contents; grep, which allows searching for patterns across files like a smart find that works across your entire codebase; bash, which allows running system commands in the terminal; and webfetch, which allows retrieving content from websites. The default if you do not specify tools is that agents inherit global settings, but you should explicitly set tools per agent for clarity and safety. Think in terms of job roles when deciding tools.

**Tool Configuration Examples:**

A code reviewer needs read access to examine code, grep to search for patterns across files, and bash to run git commands or linters. But it should NEVER modify code, so write and edit are false. Example:

```yaml
tools:
  write: false
  edit: false
  read: true
  grep: true
  bash: true
  webfetch: false
```

A builder (like the build agent) needs all capabilities since it creates and modifies code, runs commands, and searches the codebase:

```yaml
tools:
  write: true
  edit: true
  read: true
  grep: true
  bash: true
  webfetch: true
```

An explorer only needs to search and read code to understand structure:

```yaml
tools:
  write: false
  edit: false
  read: true
  grep: true
  bash: false
  webfetch: false
```

A documentation writer needs read to understand code, write and edit to create and update docs, but doesn't need bash:

```yaml
tools:
  write: true
  edit: true
  read: true
  grep: true
  bash: false
  webfetch: false
```

When you look at the example configurations in this guide, you will see exactly what typical tool sets look like for different agent types.

The permission setting provides safety controls for your agent. It determines whether an agent needs your approval before using a tool. The values are allow, which means the agent does it automatically with no prompt from you; ask, which means the agent asks "Can I do this?" and you must approve or deny; and deny, which means the agent cannot use this tool at all. Think of these as parental controls for your agent. Safe, read-only operations can be set to allow because there is no risk - reading files and searching code is harmless. Destructive or irreversible operations should be set to ask because you want to review before something potentially harmful happens. Actions that do not fit the agent's role at all should be set to deny, providing complete prohibition.

**Permission Configuration Examples:**

A code reviewer should never modify files, so you would set edit to deny. It can read files and grep safely, so those can be allow. Bash commands that could be destructive should ask:

```yaml
permission:
  read: "allow"
  grep: "allow"
  bash: "ask"
  write: "deny"
  edit: "deny"
```

A builder needs more flexibility but still requires approval for dangerous commands. You might allow safe operations automatically but ask for destructive ones:

```yaml
permission:
  read: "allow"
  grep: "allow"
  write: "allow"
  edit: "ask"
  bash:
    default: "ask"
    allow: ["git status", "git diff", "npm test", "pytest"]
    deny: ["rm -rf", "dd", "mkfs", ":(){ :|:& };:"]
```

A read-only agent like an explorer has all permissions set to allow since it only reads:

```yaml
permission:
  read: "allow"
  grep: "allow"
  write: "deny"
  edit: "deny"
  bash: "deny"
```

You can get quite granular with permissions, specifying different levels for different commands. The built-in agents have sensible defaults: the build agent has most tools set to allow because it is your main worker and you trust it; the plan agent has modification tools set to ask because it is a reviewer and you want oversight on any changes it suggests.

---

### Other Useful Settings (Learn As Needed)

#### The "Steps" Setting: Understanding Agent Thinking Cycles

The steps setting controls how long an agent can work before it must stop. Think of it as giving your agent an allowance for how many thinking turns it can take. Each "step" is one complete cycle of thinking and action-the agent reads information, decides what to do, and takes that action. For example, if you ask an agent to research vacation spots: Step 1 might search for destinations in Mexico; Step 2 reads about the top results; Step 3 checks flight prices; Step 4 writes a summary. Each cycle uses computational resources and counts toward the limit.

**What You See During Execution**

When your agent runs, you'll typically see a progress indicator like "Step 3/10" or "Thinking step 7 of 15." This shows how many steps have been used and how many remain. You might also see brief descriptions like "searching database" or "analyzing results." This transparency helps you track progress and understand if the agent is making headway or appears stuck.

**What Happens When the Limit is Reached**

When an agent hits its step limit, it stops immediately and reports back with whatever it has accomplished so far. It does not throw an error. Instead, it says something like "I've used my allocated thinking turns and here's what I found." This means you might get partial results if the task was more complex than the allowed steps. You can then decide whether to give it more steps to continue or accept the current output.

**The Trade-Off: Speed vs. Thoroughness**

There's a balance between thoroughness and responsiveness. More steps mean the agent can think longer and produce more complete results, but it takes longer and costs more. Fewer steps mean faster, cheaper responses but risk incomplete work. Simple tasks like formatting might need only 3-5 steps. Research with multiple sources often needs 10-20 steps. Deep analysis across many files could require 30-50 steps or more.

**When to Adjust the Step Limit**

Increase the step limit when the agent consistently stops before finishing-common with research tasks, analyzing large documents, comparing multiple options, or when the agent itself suggests it needs more time. Decrease the step limit for simple, routine tasks where you want faster responses, or when testing agent behavior to see if it can solve problems efficiently without excessive cycles.

**Troubleshooting**

If the agent stops too early with incomplete results, increase the step limit. If the agent seems to loop without making progress, the issue is usually flawed instructions rather than step count-review the prompt. You can always adjust the limit: start moderate, observe behavior, then fine-tune based on the quality and completeness of results.

**Cost Implications**

Each step requires computational work, and most AI services charge based on tokens processed. More steps generally mean higher costs, though they also typically yield better results. Think of it like budgeting: you get what you pay for, but avoid overspending on unnecessary thinking cycles. When cost is a concern, start with fewer steps and increase only if needed.

**Important Distinction**

Don't confuse the "steps" setting with the methodology steps the agent might describe in its prompts (like "First I'll search, then analyze"). Those are descriptive planning steps that show the agent's approach. The configured "steps" setting is a hard limit on how many thinking cycles it can actually perform, regardless of its plan.

**Quick Reference Table**

Task Type | Typical Steps | Adjust UP if... | Adjust DOWN if...
--- | --- | --- | ---
Simple formatting/Q&A | 3-5 | Answer incomplete | Task truly simple
Basic research (2-3 sources) | 5-10 | Need more depth | Need faster response
Complex research (many sources) | 10-20 | Results superficial | Cost/time constraints
Multi-file analysis | 20-30 | Missing details | Too slow for needs
Deep investigation | 30-50 | Need exhaustive coverage | Cost too high

For most uses, you can leave it blank (unlimited) and let the agent decide when it's done. Use explicit limits when you need to control costs, ensure fast responses, or prevent infinite loops from flawed instructions.

The color setting sets the agent's color in the OpenCode interface. It helps you quickly identify which agent is active at a glance when you are switching between multiple agents. The value can be a hex color code like #FF5733 or a theme name like primary, success, warning, or error. You might give your security agent a red color to indicate danger, your documentation agent a green color for go, your explorer a blue color, and your builder a yellow color. This visual cue makes it easy to see at a glance what you are working on.

There are some settings you can safely ignore for now as a beginner. These include top_p, which is an alternative to temperature used by some providers - just use temperature instead. The additional field or pass-through options are provider-specific settings only for advanced use cases. Permission.task controls what subagents an agent can call and is part of advanced orchestration patterns. Hidden is used to hide subagents from the @-mention menu and is rarely needed. Disable set to true temporarily disables an agent without deleting it, but you usually do not need this.

---

## Part 7: Building a Real Custom Agent - Step by Step

Let us create a documentation writer agent from start to finish. This concrete example will show you the complete workflow and how all the pieces fit together.

### Step 1: Recognize the Need

Imagine you maintain a project with lots of code, like a web application or a library. You frequently need to write README files that explain what the project does. You need to document APIs so other developers know how to use them. You need to add code comments explaining complex logic so future maintainers understand why things are done a certain way. You need to update documentation whenever code changes so the docs stay accurate. Currently you ask the build agent to do this, but you get inconsistent results. The tone varies from formal to casual depending on what the build agent feels like. The format is not standardized - one README might use headings this way while another uses different formatting. And the build agent sometimes forgets to document certain important things like installation prerequisites or configuration options. The solution is to create a specialized doc-writer agent with specific instructions about documentation standards, required format, and exactly what to include in every type of document. This agent becomes your dedicated technical writer who always produces consistent, comprehensive documentation.

### Step 2: Run the Wizard

Open your terminal in your project directory and type opencode agent create. The wizard starts and asks you questions. Answer as follows. First, save globally or for this project? Choose project because you want this agent to travel with this specific project and have documentation standards tailored to it. Second, agent name? Enter doc-writer. Keep it short, use a hyphen, make it clear. Third, description? Enter "Creates and maintains project documentation" - clear, concise, five words that capture the purpose. Fourth, select a model? Accept the default, which is usually claude-sonnet for a good balance. Fifth, select tools. Use the spacebar to toggle these: write should be checked because the agent needs to create doc files; edit should be checked because it needs to update existing docs; read should be checked because it needs to read code to understand what it is documenting; grep can be checked if you want it to search for existing documentation patterns; bash should be left unchecked because documentation writing does not need system commands. When you finish, the wizard creates .opencode/agents/doc-writer.md automatically in the right place.

### Step 3: Write the Prompt

Open the newly created file .opencode/agents/doc-writer.md in your editor. The wizard put in a basic prompt, but you need to replace it with specific instructions that define exactly how you want documentation written. Here is what you might write in that file after the YAML frontmatter:

You are a technical writer specializing in clear, comprehensive documentation for software projects. Write for competent developers who are not experts in this specific codebase. Assume they know programming but do not know the why behind design decisions. Every piece of documentation should start with a clear purpose answering "what problem does this solve?" Use headings to organize content logically. Include code examples with language specification using triple backticks. Explain the why not just the what. Follow consistent formatting using Markdown appropriately. Link to related components or resources where relevant.

Now specify the documentation types you want this agent to handle. For README files, include project overview stating what it does and why it exists, installation or setup instructions that are step by step and copy-pasteable, quick usage examples that show the basics, and links to more detailed documentation. For API documentation, include endpoint descriptions that explain purpose not just parameters, request and response examples showing actual JSON or data structures, authentication requirements spelled out clearly, error codes and what they mean, and any rate limits that apply. For code comments, explain the why not the what because the code itself shows the what. Document assumptions, edge cases, and trade-offs made in the implementation. Use complete sentences and proper grammar. Reference related functions or modules so developers can see connections. For developer guides, cover architecture overview at a high level, design patterns that are used and why, configuration options with examples, contribution guidelines for people who want to help, and testing approaches so they understand how to verify their changes.

Define the agent's process. When writing or updating documentation: first read and understand the code thoroughly, run it if possible to see behavior; second, identify the audience - are you writing for developers, users, or operators; third, structure content from general to specific concepts; fourth, write examples that are practical and copy-pasteable working code; fifth, review for clarity - would someone new to this codebase understand everything; sixth, check that code examples actually work by verifying syntax and logic; seventh, ensure links between related docs are present so people can navigate.

When updating existing documentation: preserve what is still relevant from the old version; mark outdated information clearly with DEPRECATED and suggest the new approach; maintain consistent voice and style throughout; update cross-references that point to other documents; note what changed and why in a changelog section. For output format, use proper Markdown throughout. Use code blocks with language identifiers like python or javascript. Use tables for options and parameters. When describing architecture, use text diagrams if helpful - you can represent diagrams with ASCII art or numbered lists showing flow. Keep lines under 80 characters for readability in any text editor. Use code comments in examples to explain important lines.

You would also set the temperature to 0.4 because documentation needs some variation but not too much creativity. You would set tools appropriately: write true, edit true, read true, grep true, bash false. And permissions: edit ask because you want approval before documentation gets modified, but writes can happen automatically since creating new docs is low risk.

### Step 4: Test Your Agent

In OpenCode, type @doc-writer create a README.md for this project explaining what it does and how to install it. Watch the agent work through its process. It should first read your codebase to understand the project structure and what it actually does. It should identify the technology stack by looking at package.json or requirements files. It should figure out installation steps by examining configuration and dependency files. Then it should generate a properly formatted README following all the instructions you provided. Check the output against your expectations. Is the tone consistent? Does it include all the sections you specified? Are code examples properly formatted? Does it explain the why behind things and not just list features?

### Step 5: Refine Based on Results

Your first prompt will not be perfect. That is expected and normal. Use the agent, see what it produces, then refine the prompt based on what you observe. If the output is too verbose and wordy, change temperature from 0.4 to 0.3 to make it more concise. If it is missing something important that you expected, add that requirement to the prompt instructions explicitly. If the tone is wrong - too casual or too formal - adjust the first paragraph where you define your principles and voice. If it is not following your desired format, be more explicit: "Your output MUST include these sections in this exact order: 1) Overview, 2) Installation, 3) Usage, 4) Configuration, 5) Contributing." Be specific about required structure. This iterative improvement is how you get a solid documentation agent. After two or three iterations of using it, seeing results, and refining the prompt, you will have an agent that consistently produces excellent documentation tailored to your project's needs.

---

## Part 8: Ready-to-Use Agent Examples

This section provides complete, proven agent configurations that you can copy into your project to immediately add powerful specialized agents to your toolkit. These are production-ready prompts that have been carefully crafted for effectiveness. Each example includes the full configuration file content that you can copy and paste.

### Code Reviewer

This agent is a thorough code reviewer that checks for quality, security, and best practices. It does not modify code - it only analyzes and reports. Save the following as `.opencode/agents/code-reviewer.md`:

```yaml
---
description: "Thorough code reviewer for quality, security, and best practices"
mode: "subagent"
model: "anthropic/claude-sonnet-4-5"
temperature: 0.2
tools:
  write: false
  edit: false
  read: true
  grep: true
  bash: true
permission:
  read: "allow"
  grep: "allow"
  bash: "ask"
  write: "deny"
  edit: "deny"
---

You are a senior code reviewer with 15 years of experience. Your reviews are thorough, constructive, and actionable. You follow a comprehensive checklist covering six major areas.

**1. Security Issues:**
- SQL injection, command injection, XSS, path traversal
- Authentication: password handling, session management, MFA
- Authorization: privilege escalation, missing access controls
- Secrets: API keys, passwords, certificates in code
- Data validation: input sanitization, output encoding

**2. Error Handling:**
- Missing try-catch around risky operations
- Unhandled edge cases
- Poor error messages (leaking info or too vague)
- Silent failures
- Resource cleanup issues

**3. Performance:**
- Inefficient loops (O(n²) vs O(n))
- Unnecessary computations in hot paths
- N+1 query problems
- Potential memory leaks
- Blocking operations in async contexts
- Caching opportunities

**4. Code Quality:**
- Naming clarity (unclear, misleading, abbreviations)
- Duplication (same code in multiple places)
- Complexity (functions >50 lines, nesting >3 levels)
- SOLID violations
- Magic numbers/strings
- Dead code

**5. Testing:**
- Test coverage for new code
- Edge case coverage
- Flaky tests
- Clear test names

**6. Maintainability:**
- Tight coupling
- Hidden dependencies
- Unclear data flow
- Missing/outdated comments
- Long functions doing too much

**For each issue found:**
- Severity: Critical / High / Medium / Low / Info
- Location: file:line
- Problem: Clear explanation
- Fix: Specific code example when helpful

**Output format:**
1. **Summary** - Overall assessment (2-3 sentences)
2. **Issues Found** - Detailed entries with severity, location, problem, fix
3. **Positive Observations** - What was done well
4. **Top 3 Priorities** - Most urgent issues

Your approach should be:
- Specific, not vague
- Explain the "why" behind each issue
- Provide actionable fixes that can be implemented today
- Critique code, not people
- Acknowledge excellence when you see it

### Security Auditor

This agent is a cybersecurity expert conducting comprehensive security audits. It is even more thorough than the code reviewer and focuses exclusively on security vulnerabilities. Save as `.opencode/agents/security-auditor.md`:

```yaml
---
description: "Cybersecurity expert performing comprehensive security audits"
mode: "subagent"
model: "anthropic/claude-opus-4"  # Use the most capable model for security
temperature: 0.1  # Very low for consistent, reliable security analysis
tools:
  write: false
  edit: false
  read: true
  grep: true
  bash: true
permission:
  read: "allow"
  grep: "allow"
  bash: "ask"
  write: "deny"
  edit: "deny"
---

You are a cybersecurity expert conducting a comprehensive security audit. Your audit scope covers ten major categories of security concerns.

**1. Input Validation & Injection Attacks:**
- SQL injection (unsanitized queries)
- Command injection (system command execution)
- XSS (cross-site scripting)
- Path traversal (../ sequences)
- XML external entity injection
- LDAP injection
- Template injection

**2. Authentication Issues:**
- Password storage (use bcrypt/scrypt/Argon2, never plain text or MD5)
- Password policies (length, complexity, history)
- Multi-factor availability
- Session management (secure/HttpOnly cookies, expiration, ID rotation)
- HTTPS-only credentials
- Credential leakage in logs/URLs
- Brute force protection

**3. Authorization Problems:**
- Privilege escalation (horizontal & vertical)
- Missing access controls
- Insecure direct object references (predictable IDs)
- Improper RBAC implementation

**4. Data Protection:**
- PII handling procedures
- Encryption at rest
- Encryption in transit (TLS)
- Key management (no hardcoded keys, use vaults)
- Data retention/deletion policies
- Sensitive data in logs

**5. Secrets Management:**
- API keys in code/config files
- Hardcoded service credentials
- JWT secrets in client-side code
- OAuth client secrets committed
- Environment variable misuse

**6. Dependencies:**
- Known CVEs in package.json/requirements.txt/pom.xml
- Outdated packages with security fixes
- Transitive dependency vulnerabilities
- Supply chain risks

**7. Configuration Issues:**
- Debug mode in production
- Default credentials unchanged
- Open ports/exposed services
- Overly permissive CORS
- Verbose error messages

**8. Cryptography:**
- Weak algorithms (MD5, SHA1, DES, RC4)
- ECB mode (insecure)
- Predictable random numbers
- Hardcoded keys/IVs
- Improper certificate validation

**9. Logging & Monitoring:**
- Sensitive data in logs (passwords, tokens)
- Sufficient forensic logging
- Missing security alerts
- Timestamp formats for correlation

**10. API Security:**
- Missing rate limiting
- Missing request size limits
- Improper timeouts
- Unnecessary HTTP verbs
- Information disclosure in errors

**For each finding:**
- Severity: Critical / High / Medium / Low / Info
- Reference CWE or OWASP category (e.g., CWE-89, A02:2021)
- Description of vulnerability
- Impact: what could happen (data breach, privilege escalation, DoS)
- Exact location
- Remediation: step-by-step fix with code examples

**Output structure:**
1. **Executive Summary** - Overall risk rating, findings by severity, immediate actions, statistics
2. **Findings** - Organized by the 10 categories above
3. **Positive Observations** - Secure practices found
4. **Remediation Roadmap** - Prioritized by severity with effort estimates
5. **References** - Security standards and guidelines

Your approach should be:
- Thorough (depth over breadth)
- Assume an attacker mindset
- No false positives (be certain before flagging)
- Rate as "Info" if uncertain (don't inflate severity)
- Consider both technical risk AND business impact

### Debugging Assistant

This agent is a systematic bug investigator and performance analyzer. It follows a methodical debugging process rather than guessing. Save as `.opencode/agents/debugger.md`:

```yaml
---
description: "Systematic bug investigator and performance analyzer"
mode: "subagent"
model: "anthropic/claude-sonnet-4-5"
temperature: 0.3  # Low for methodical, consistent investigation
tools:
  write: false
  edit: false
  read: true
  grep: true
  bash: true
permission:
  read: "allow"
  grep: "allow"
  bash: "ask"  # Ask before running debug commands
  write: "deny"
  edit: "deny"
---

You are a debugging specialist. You are systematic, thorough, and methodical. You do not guess - you investigate. You follow a nine-phase debugging process.

**Phase 1: Information Gathering**
- Collect error messages, stack traces, logs
- Note exact symptoms (crash, slow, wrong output)
- When did it start? (recent changes/deployments?)
- Is it reproducible? (always/sometimes/random?)
- Environment details (OS, browser, versions)

**Phase 2: Reproduction**
- Document exact steps to trigger
- Test in different environments (dev vs prod)
- Is it user-specific or data-specific?
- Create minimum reproduction case

**Phase 3: Log Analysis**
- Check application logs (common locations: /var/log, ~/.logs, app-specific)
- Check system logs
- Look for patterns (time of day, specific user, feature usage)
- Increase verbosity if needed

**Phase 4: Preliminary Investigation**
- Check recent changes (git blame, git log -p, deployment notes)
- Review related code areas
- Resource exhaustion? (memory growth → leak, file handles, DB pool, disk full)
- Timing issues? (race conditions, timeouts)
- Boundary conditions? (nulls, empty arrays, large/negative values)

**Phase 5: Isolation**
- Narrow down to specific function/module/component
- Binary search approach (comment out/disable features)
- Add logging/breakpoints to trace execution

**Phase 6: Hypothesis & Test**
- Formulate 2-3 possible root causes
- Design tests to validate/refute each hypothesis
- Use diff/patch to apply temporary fixes one at a time

**Phase 7: Root Cause Identification**
- Exact line of code or configuration causing the issue
- What is the defect?
- Why does it occur?
- Under what conditions does it manifest?
- How long has it been present?

**Phase 8: Fix Recommendation**
- Specific fix with code changes or config updates
- Reasoning: why this fixes the root cause
- Test cases to verify and prevent regression
- Similar patterns elsewhere that might have same issue

**Phase 9: Prevention**
- Process changes to catch similar bugs earlier
- Test additions (unit/integration/e2e)
- Monitoring/alerting for early detection
- Static analysis or lint rules to catch this class of error

**Common Bug Patterns to Check:**

*Memory leaks:* unreleased resources (DB connections, file handles), growing caches without eviction, circular references, unbounded data structures.

*Race conditions:* shared mutable state without sync, async operations completing out of order, cache invalidation timing, DB transaction isolation issues.

*Configuration issues:* missing/wrong env vars, dev vs prod differences, misconfigured feature flags, timezone mismatches, resource limits.

*Network/service issues:* timeout settings, missing retry logic, circuit breaker misconfiguration, DNS problems, load balancer health checks.

*Data issues:* null handling, empty collection handling, JSON parsing type mismatches, out-of-range values, malformed input.

**For Performance Issues:**
- Collect metrics: response time (p50/p95/p99), throughput (req/sec), error rate, resource utilization (CPU/memory/disk/network)
- Profile to identify hotspots
- Check DB queries (N+1, missing indexes)
- Review external service calls
- Examine algorithmic complexity
- Check for blocking in async code

**Output format:**
- **Issue Summary** - What the problem is (1 paragraph)
- **Investigation Steps** - What was examined, what was ruled out
- **Root Cause** - Specific file:line with explanation
- **Recommended Fix** - Code change with reasoning
- **Verification** - How to test the fix works
- **Prevention** - How to avoid similar bugs in future

Your communication style is:
- Systematic (show your reasoning)
- Specific (exact locations, code snippets)
- Actionable (fixes you can implement today)
- Educational (explain concepts when needed)
- Confident but not arrogant (say when uncertain)

### Architecture Planner

This agent is a systems designer and implementation planner. It is read-only and does not write code - it plans and guides. Save as .opencode/agents/architect.md:

This agent is a system architect. It designs robust, scalable systems and creates detailed implementation plans. It does not write code - it plans and guides. It has a primary mode because you switch to it directly when you need architectural guidance.

The agent follows a seven-step architecture process. Step one is requirements analysis. If anything is unclear, it asks clarifying questions. It seeks to understand functional requirements - what the system must actually do. It identifies non-functional requirements including performance targets for throughput and latency and concurrent users, reliability targets for uptime and disaster recovery, security needs for compliance and data protection, scalability requirements for growth projections and scaling triggers, maintainability considerations for expected lifespan and team turnover, and cost constraints for both capital and operational expenses.

Step two is constraints identification. It documents budget limitations, timeline constraints including launch dates and phased rollout needs, team skills and what technologies the team actually knows, existing infrastructure that can be reused, regulatory requirements such as GDPR or HIPAA or PCI or SOC2, and data sovereignty rules about where data can be stored.

Step three is considering trade-offs. The agent explicitly evaluates important trade-offs: simplicity versus flexibility, performance versus maintainability, time-to-market versus robustness, off-the-shelf solutions versus custom-built, monolithic versus microservices architecture, SQL versus NoSQL databases, and self-hosted versus cloud deployment. It presents trade-off analysis in plain language: option A is simpler but less flexible; option B handles scale better but adds complexity; given your constraints of X and Y, I recommend Z because it balances your specific needs.

Step four is choosing patterns and technologies. The agent selects architectural patterns and justifies each choice. For communication style, it decides between synchronous approaches like REST or gRPC versus asynchronous approaches like message queues or events. For data management, it chooses between relational, document, graph, or hybrid approaches. For deployment strategy, it picks monolith, modular monolith, microservices, or serverless. For caching, it selects Redis, CDN, or in-memory solutions. For state management, it decides between stateless services, sticky sessions, or distributed state.

Technology choices come with rationale. It might say use PostgreSQL for ACID compliance and complex queries, or use React for component reusability and strong ecosystem, or use Kafka for event streaming with guaranteed ordering. It always includes alternatives with reasons why they were not chosen. For example, MongoDB could work but lacks transactions - only use it if schema flexibility is absolutely critical.

Step five is creating diagrams described in text. Since the agent cannot actually draw images, it describes diagrams clearly using text representations. A system component diagram might look like this in text: Client points to API Gateway which points to Auth Service and User Service; Auth Service also points to Order Service which points to Database; Order Service also points to Payment Service which points to External Payment API. The agent describes relationships, data flows, and communication protocols in words accompanying this text diagram. A data flow diagram describes the sequence: user submits order, API receives request, validates input, calls Order Service, Order Service creates order in database with status pending, publishes order.created event, Payment Service consumes event, calls Payment API, updates order status. A database schema for relational databases is represented as a list of tables with columns.

Step six is implementation roadmap. The agent breaks the project into phases with clear deliverables, effort estimates in person-weeks, and dependencies between phases. Phase one might be MVP taking four weeks: set up infrastructure, implement authentication service, implement basic CRUD for users, build a simple frontend. Phase two might be core features taking six weeks: order processing workflow, payment integration, admin dashboard. Phase three might be scale and polish taking three weeks: performance optimization, monitoring and alerting setup, documentation completion. Each phase has concrete deliverables that can be checked off.

Step seven is risk assessment. The agent identifies what could go wrong and how to mitigate it. Technical risks include third-party API availability - the mitigation is implementing circuit breakers and fallbacks; database performance - add read replicas, optimize queries, add indexes; security breach - implement defense in depth with regular audits. Operational risks include team knowledge silos - mitigate by documenting architecture and conducting knowledge sharing; deployment failures - use blue-green deployment and feature flags; scaling bottlenecks - do load testing before launch and configure auto-scaling. Timeline risks include unclear requirements - timebox discovery and iterate; integration challenges - do spikes early and mock external services; team availability - cross-train team members and document interfaces.

The agent produces ten output artifacts when asked to design something. First, executive summary for stakeholders. Second, system context showing what is in scope and external integrations. Third, architecture decisions with pattern choices and rationale. Fourth, component diagram description. Fifth, data model with schema and relationships. Sixth, API specifications with endpoints, request and response examples, and authentication details. Seventh, technology stack listing languages, frameworks, databases, and infrastructure. Eighth, implementation roadmap with phases and estimates. Ninth, risk assessment covering what could fail and mitigations. Tenth, alternatives considered showing what was not chosen and why.

The agent has special considerations for scalability, reliability, security, and operability. For scalability, it asks where bottlenecks might be, how the system scales horizontally by adding instances, what needs stateful versus stateless design, what caching strategy to use, and whether database sharding will be needed. For reliability, it examines single points of failure and how to eliminate them, considers failure domains and what happens if an availability zone fails, plans data replication and backup strategy, defines recovery time and recovery point objectives, and designs monitoring with health checks and alerts. For security, it designs authentication and authorization mechanisms, plans network segmentation with VPCs and security groups, chooses secrets management solutions, implements audit logging, and designs input validation strategy with rate limiting. For operability, it ensures deployment process includes rollback capability, provides monitoring visibility with logs and metrics and traces, sets up alerting on key indicators, creates documentation for the operations team, and develops runbooks for common issues.

The agent lives by several principles. No architecture astronautics - do not over-engineer; simple solutions win. Always justify trade-offs because every decision has trade-offs. Consider operational burden - what looks good on paper might be hell to operate day-to-day. Match technology choices to team skills - brilliant technology that your team cannot use is bad technology. Future-proof but do not overbuild - design for expected load, not theoretical maximums. Design security in from the start rather than bolting it on later. Be cost-aware and consider cloud provider pricing implications. The agent provides specific, actionable recommendations with clear rationale. It acknowledges uncertainty and trade-offs rather than claiming a single perfect solution exists.

You use this agent by switching to it - press Tab until you see architect - and then ask your questions. For example: architect, design a microservice architecture for an e-commerce platform with 100,000 users or architect, plan the refactoring of this monolithic application into separate services.

### Research Specialist

This agent is a deep investigator and information synthesizer. Save as .opencode/agents/researcher.md:

This agent is a research specialist. It excels at gathering information from diverse sources, connecting dots that others might miss, and synthesizing clear, structured findings from complex information. Its temperature is set higher at 0.6 because research benefits from some exploration and synthesis ability.

The agent follows a five-phase research methodology. Phase one is clarify and scope. It understands exactly what the research question needs to answer. It breaks the main question into smaller sub-questions that can be tackled individually. It determines the scope - what is in scope and what is out of scope to prevent going down irrelevant rabbit holes. It identifies the audience for this research - are the findings for developers, managers, or executives? And it clarifies deliverables - what format should the results be in: a report, a summary, a comparison table?

Phase two is identify information sources. The agent has access to several types of sources. The codebase of the current project - it can search files, read code, understand patterns and conventions. The web via webfetch - it can retrieve content from official documentation, RFCs, well-known engineering blogs, Stack Overflow. Internal documentation including project READMEs, architecture documents, and ADRs or architecture decision records. For codebase research, it asks what files and components are relevant, what existing patterns or conventions it can observe, and what the key configuration files reveal. For web research, it prefers authoritative sources like official documentation, RFCs from standards bodies, and major technology company engineering blogs. It avoids personal opinion pieces unless they show consensus across many sources. It checks information freshness because technology changes fast and 2019 advice may be completely obsolete in 2025.

Phase three is systematic exploration. The agent executes its research plan methodically. For codebase research, it uses explore or grep to find relevant files, reads source files and configuration and tests, traces data flows and dependencies, identifies patterns and conventions and anti-patterns, extracts business logic and configuration values and constants, and tries to understand the why behind architectural decisions if that intent is evident in comments or documentation. For web research, it uses webfetch on specific URLs rather than fetching entire sites, extracts only the information that is relevant to the research questions, cross-verifies information across multiple sources when possible, notes the source and date for each piece of information, and remains skeptical of outdated content.

As it researches, the agent takes structured notes organized by category or by the original sub-questions. It tracks which sources support which findings. It flags uncertainties and contradictions between sources.

Phase four is synthesize. This is where the agent does not just dump raw findings but connects them into meaningful insights. It identifies consensus - what do multiple reliable sources agree on? It identifies contradictions - where do sources disagree and which source is more authoritative or more recent? It identifies gaps - what remains uncertain or requires deeper investigation? It extracts insights - what do the facts imply, what recommendations follow logically from the findings? And it creates a narrative - structure the findings logically, not just as a random list of facts.

Phase five is present findings. The agent structures its output hierarchically. It provides an executive summary for busy readers who need the bottom line in two or three paragraphs. It provides detailed findings organized by theme or by the original sub-questions. It uses headings, lists, and tables for clarity. It includes code examples from the codebase if relevant. It links to source materials when possible. It distinguishes between facts that were observed directly, inferences that are reasonable conclusions from facts, opinions that are someone's judgment, and speculation where uncertainty exists. For each major finding, it states the finding clearly, provides supporting evidence like code snippets or documentation quotes or citations, indicates confidence level as high, medium, or low with explanation, and explains implications or recommendations.

The executive summary should answer the fundamental question: what do I need to know and what should I do about it? in two or three paragraphs.

The agent has two research examples that illustrate its approach. Example one is codebase research: the question is "How does this codebase currently handle authentication?" The process involves searching for auth-related files with @explore find auth login session, reading authentication middleware and login handlers and session management code, checking for password hashing libraries like bcrypt or argon2, looking for JWT usage and session stores, examining configuration files for authentication settings, and testing authentication-related tests to understand expected behavior. The output is an authentication architecture summary describing the current approach.

Example two is web research: the question is "What are best practices for API rate limiting in 2024?" The process involves identifying authoritative sources like IETF drafts, OWASP guidelines, engineering blogs from Google or Cloudflare or AWS, using webfetch on three to five key sources, extracting recommendations about algorithms like token bucket and leaky bucket and fixed window, understanding distributed rate limiting challenges, and learning about client identification methods. The agent cross-verifies across sources to find consensus, notes contradictions where sources disagree, and pays attention to recency of information. The output is a summary of current best practices.

Example three is hybrid research combining both approaches: the question might be "What authentication methods does our codebase currently use, and are there better alternatives?" The agent first researches the codebase's current state as in example one, then researches modern best practices as in example two, then compares current approach to recommended practices, identifies gaps, and suggests specific improvements with rationale.

The agent's output format includes an executive summary, a background section explaining scope and sources and methodology, detailed findings organized by theme, a questions answered section restating original questions and providing clear answers, an open questions and uncertainties section, recommendations for actionable next steps prioritized, and a sources list with URLs and access dates. The agent follows principles of being thorough but concise, preferring authoritative sources, cross-verifying information, noting dates because technology changes fast, extracting insights rather than dumping raw data, being transparent about confidence levels, and providing actionable insights rather than just information. It structures its output for the intended audience - executives need the summary while implementers need details.

You use this agent with @general followed by your research question. For example: @general research best practices for handling file uploads in web applications or @general what authentication methods does this codebase currently use?

### Quality Gatekeeper

This agent runs linting, tests, formatting, and security scans to ensure code health before commits or releases. Save as .opencode/agents/quality-check.md:

This agent runs quality assurance commands to ensure code health before commits or releases. It checks that code meets style standards, passes tests, is properly formatted, and has no known security vulnerabilities in dependencies.

The agent follows a quality check sequence of six steps. Step one is linting - it runs static analysis to catch style issues, potential bugs, and code smells. Common linting tools include ESLint or biome or stylelint for JavaScript and TypeScript; flake8 or pylint or ruff for Python; golangci-lint for Go; Checkstyle or PMD for Java; clippy for Rust. The agent runs the project's configured linter and passes if there are no errors. Warnings are noted but do not cause failure.

Step two is type checking if the project uses types. For TypeScript projects, it runs npm run type-check or tsc --noEmit. For Python with type hints, it runs mypy. For Java, the compiler performs type checks. It ensures all types are correct and there are no type errors.

Step three is testing. The agent runs the full test suite. It checks that unit tests pass, integration tests pass, and coverage meets the project's threshold which is typically 70 or 80 percent. All tests must pass before proceeding further.

Step four is code formatting. It checks whether code is properly formatted using the project's formatter. Commands might be npm run format:check or equivalent. For Python it uses black --check, for JavaScript Prettier --check, for Go gofmt -l to list unformatted files, for Rust cargo fmt --check. If unformatted files exist, the agent notes them and suggests running the formatting command.

Step five is security scanning if the project has it configured. Tools include Snyk, Dependabot, npm audit for JavaScript, pip-audit for Python, or OWASP Dependency Check for Java. The agent checks for known vulnerabilities in dependencies.

Step six is additional project-specific checks that some projects have. These might include spell checking in documentation, schema validation for data structures, API contract tests to ensure interfaces remain consistent, or performance benchmarks that must not regress. The agent follows whatever configuration the project has defined.

The agent knows language-specific commands for common ecosystems. For JavaScript and TypeScript, the typical commands are npm run lint or yarn lint for linting, npm run type-check for TypeScript type checking, npm test for tests, npm run format:check for formatting, and npm audit for security scanning. For Python, it uses flake8 . or ruff check . for linting, mypy . for type checking if configured, pytest for tests, black --check . for formatting, and pip-audit for security. For Go, the commands are go vet ./... for static analysis, go test ./... for tests, go fmt -l to list unformatted files, and golangci-lint run if configured. For Java, it uses ./mvnw clean test for tests and Maven plugins for checkstyle and spotbugs. For Rust, the commands are cargo clippy --all-targets --all-features for linting, cargo test for tests, and cargo fmt --check for formatting. For .NET and C#, it uses dotnet format --verify-no-changes for formatting, dotnet test for tests, and dotnet analyzers for static analysis.

The agent's output format is structured and clear. It shows the project that was detected from package.json or requirements.txt or other manifest files. It lists the checks that were run. It gives an overall status of pass, fail, or warnings. Then for each check type - linting, testing, type checking, formatting, security scanning - it shows results, details like specific errors if any, and a recommendation about what to do next. Finally it provides a summary with total issues broken down by severity, blocking issues that prevent commit or push, and non-blocking issues that are warnings. The recommendation is clear: either fix the specific issues and run again, or all checks passed and you are good to commit.

The agent's behavior is important to understand. It only runs checks - it does not automatically fix things unless you explicitly ask it to fix specific problems. It uses safe commands and never destructive operations. It prefers dry-run or check modes when available using flags like --check or --verify-no-changes. If a command is not found, it suggests the correct command for that ecosystem. It runs checks in logical order: lint first, then type check, then test, then format, then security. If an earlier check fails, it may stop rather than continuing because there is no point formatting code that has failing tests. It is precise in its reporting, showing exact error messages rather than just saying tests failed.

The configuration for this agent includes permission settings that allow safe commands automatically - things like git status, test runners, and linters are safe. Dangerous commands like rm or dd are blocked entirely, which is appropriate because this agent is a quality checker, not a file destroyer. The configuration allows common test and lint commands automatically because they are known to be safe.

You use this agent with @quality-check followed by your request. For example: @quality-check run all the quality checks for this project or @quality-check should I commit these changes?

---

## Part 9: Advanced Topics (For Later)

Once you are comfortable with the basics and have created a few custom agents, these advanced concepts will help you fine-tune your agent setup and have more control over how OpenCode behaves.

### Configuration Precedence: Which Setting Wins?

When you define the same agent in multiple places, OpenCode loads and merges configurations from all sources. Later sources override earlier ones. Understanding this order is crucial when you have global settings that you want to customize for a specific project.

The priority order from lowest to highest is as follows. Built-in defaults come first - OpenCode's built-in agents have default configurations. Next is the global config file at ~/.config/opencode/opencode.json. Next are global agent files in ~/.config/opencode/agents/*.md that contain individual agent definitions. Then custom config if you set an OPENCODE_CONFIG environment variable pointing to a specific file. Then the project config file at opencode.json in your project root directory. Finally, project agent files in .opencode/agents/*.md have the highest priority.

Here is a practical example to make this concrete. Suppose you have a global plan agent with temperature set to 0.1. This means in all your projects, the plan agent is very consistent and has low creativity. Now in a specific project, you create .opencode/agents/plan.md with temperature set to 0.3. In that specific project, the plan agent will use temperature 0.3 because the project-level configuration overrides the global setting. In all other projects that do not have a project-specific override, the plan agent still uses temperature 0.1 from the global config. This is powerful because it lets you have general preferences globally but fine-tune for particular projects that need different behavior.

To see exactly what configuration OpenCode is actually using after all the merging, you can run opencode config show. This displays the fully merged configuration so you know precisely what settings are in effect. This is invaluable when you are debugging why an agent is not behaving as you expected - you can see if your intended settings were overridden by something with higher priority.

### Configuration Merging Example

Let us walk through a complete merging example to solidify understanding. Suppose your global configuration at ~/.config/opencode/opencode.json contains an agent definition for plan with model set to anthropic/claude-haiku-4-5 and temperature 0.1. Now suppose in a specific project you have a project agent file at .opencode/agents/plan.md that sets mode to primary, temperature to 0.2, and contains a custom prompt about thoughtful planning focused on your project's domain. The resulting plan agent in that project will combine these settings. The model comes from global because the project file did not override it. The temperature comes from the project file at 0.2, overriding the global 0.1. The mode comes from the project file as primary. And the prompt comes entirely from the project file because the project agent file's content after the YAML frontmatter becomes the prompt. The agent effectively uses the most specific definition for each setting, with project-level taking precedence over global.

Do not overcomplicate your configuration setup. Start with one location - either JSON or Markdown agents - and choose either project-specific or global depending on your needs. Use opencode config show when you need to debug why settings are not what you expect. This command will reveal exactly which files contributed to the final configuration and which ones took precedence.

### Provider and Model Setup

Before you can use AI models from providers like Anthropic or OpenAI, you need to connect your API credentials to OpenCode. You do this by running the /connect command in OpenCode. Follow the prompts to add your API key for the provider you want to use. Once you have connected a provider, you can use models from that provider in your agent configurations by specifying them in the model field.

#### What Is an API Key and Why Is It Needed?

An API key is a unique string of characters that serves as both an identifier and a password for accessing an AI provider's services over the internet. Think of it like a membership card or a key to a clubhouse. The API key tells the provider who is making the request, and it proves that you have permission to use their AI models. Without an API key, you cannot access Anthropic's Claude models, OpenAI's GPT models, or any other cloud-based AI service.

API keys are necessary for several fundamental reasons. First, they enable authentication-the provider needs to know that you're you. When you send a request with your API key, the provider looks up your account, checks that the key is valid, verifies you have an active subscription or billing arrangement, and then processes your request. This prevents unauthorized people from using the service at your expense or accessing your account.

Second, API keys enable billing and usage tracking. Every request you make is associated with your API key, and the provider tracks how many tokens (roughly, words) you process, which models you use, and how much each request costs. At the end of the billing period, you're charged based on your usage. Without API keys, providers would have no way to know who to bill or how much to bill them.

Third, API keys provide security and control. If your API key is compromised (someone steals it), you can revoke it and generate a new one, immediately cutting off unauthorized access. You can also create multiple API keys with different permissions if you want separate keys for different projects or environments. You can monitor usage through your provider's dashboard to see if something looks unusual (like a sudden spike that might indicate abuse).

Fourth, API keys enforce rate limits. Providers typically limit how many requests you can make per minute or per day to prevent abuse and ensure fair usage across all customers. Your API key is how they count and limit your requests. If you exceed your limits, requests will be rejected until the rate limit window resets.

How API keys work in practice is straightforward but important to understand. When you run OpenCode and an agent needs to use an AI model, OpenCode constructs a request containing your prompt, the model you specified, and any other parameters. It then sends this request to the provider's API endpoint (a specific web address), including your API key in the request headers, typically in an Authorization header like `Authorization: Bearer sk-ant-...` for Anthropic or `Authorization: Bearer sk-...` for OpenAI. The provider receives the request, validates the API key, checks your account status and balance, processes the request through their model, and sends back a response that includes the model's output. OpenCode receives this response and presents it to you. The entire process happens transparently-you don't see the API key or the request details unless you look at configuration.

Getting an API key is a separate step from installing OpenCode. OpenCode itself doesn't provide AI models; it's an orchestration layer that connects to various providers. You must have your own accounts with the providers you want to use and obtain API keys from their websites. This typically involves: signing up for an account at the provider's website (like console.anthropic.com or platform.openai.com), providing payment information (most providers require a credit card even for free trial tiers), navigating to the API keys section of the dashboard, and generating a new key. The key will be a long string starting with "sk-" (for secret key) followed by many alphanumeric characters. You copy this key and paste it into OpenCode when prompted by the /connect command.

Security considerations for API keys are critically important because anyone who has your API key can use your AI quota and run up charges on your account. Treat your API keys like passwords or credit card numbers. Never commit them to version control-if a key appears in Git history, it's compromised and you must revoke it immediately. Never share your API keys with others-each person should use their own keys. Never hardcode API keys in your application code-use environment variables or OpenCode's configuration system which stores them securely. Be aware of which providers you've shared your keys with-if you give a key to a third-party tool, that tool can use your quota and you're trusting them not to misuse it.

In OpenCode specifically, API keys are stored securely when you use the /connect command. They're saved in your operating system's credential storage or in encrypted configuration files, depending on your setup. The exact storage mechanism varies by platform, but OpenCode takes care to keep them safe. You should still protect your computer and user account with a strong password, especially if others have physical or remote access.

A practical example of how API keys appear in configuration: when you run `/connect`, OpenCode will ask which provider you want to connect (Anthropic, OpenAI, etc.). You paste your API key for that provider. OpenCode validates it by making a small test request. Once validated, OpenCode can now use any model from that provider. In your agent configuration, you specify `"model": "anthropic/claude-sonnet-4-5"`. When that agent runs, OpenCode looks up your stored Anthropic API key, uses it to access Claude Sonnet, and you get the results. You never need to put the API key directly in your agent configuration files-that's handled centrally by the /connect system.

Common mistakes with API keys include: copying the wrong key (sometimes dash the wrong characters, be careful); using a test or expired key that no longer works; having insufficient balance or hitting spending limits that prevent the provider from fulfilling requests; using a key from the wrong provider (e.g., an OpenAI key won't work for Anthropic models); and exposing keys accidentally through screenshots, chat messages, or by pasting them into shared documents. Always double-check that you're using the right key for the right provider.

If an agent fails with an authentication error, the first thing to check is whether you've run /connect for that provider and whether the API key is still valid (not expired or revoked). You can run /connect again to reconnect or add keys for additional providers. The /connect command also lets you view which providers are currently connected and manage your stored credentials.

#### What Is a "Provider"? (In Depth)

As introduced earlier, a provider is the company or service that creates and hosts AI models. But let's explore what that means more thoroughly and why providers are distinct from models and agents.

The AI landscape has several major providers, each with their own philosophies, model families, strengths, and pricing. Anthropic focuses on safety, constitutional AI principles, and models that are particularly good at following complex instructions and reasoning about ethical considerations. Their model lineup includes Claude Haiku (fast, cheap), Claude Sonnet (balanced), and Claude Opus (their most capable but slow and expensive). OpenAI offers GPT models (GPT-4, GPT-4.5, GPT-5 in various variants like standard, mini, nano, codex) that are widely adopted and have strong all-around capabilities. OpenCode provides its own hosted models, including variants optimized specifically for coding tasks. Google provides Gemini models. Mistral AI and other companies also offer models. Each provider builds and trains their own neural networks, manages their own infrastructure, sets their own pricing, and controls their own API specifications.

When you choose a model for your agent, you're implicitly choosing a provider. The provider name is part of the model identifier: `anthropic/claude-sonnet-4-5` specifies Anthropic as the provider. If you haven't connected to Anthropic via /connect, that model won't work even though it's specified correctly. The provider determines where the request goes, who bills you, and what capabilities are available.

Providers differ in important ways beyond just model names. Pricing models vary-some charge per token, some have monthly subscriptions that include a certain amount of usage, some use a hybrid approach. Speed varies-some models are optimized for fast responses (Haiku, GPT-4o Mini), others for thorough reasoning (Opus, GPT-5 Codex) and take longer. Capability differences-some models excel at code generation, others at creative writing, others at mathematical reasoning, others at multi-modal tasks that include images. Context window sizes differ-the maximum amount of text the model can consider in one request varies by model. API features differ-some providers offer streaming responses (where you see output gradually appear), some offer tool use capabilities built-in, some have special parameters.

Provider choice also affects your agent's behavior in subtle ways. Models from different providers respond differently to the same prompt, even if the prompt is well-crafted. Temperature settings don't have identical meaning across providers-what's "medium creativity" on Claude might feel different on GPT. Output formatting tendencies differ-some models naturally use Markdown, others plain text; some are verbose, others concise. Model-specific quirks exist-some models have certain tokens they overuse, some have biases in their responses, some have limitations around certain tasks.

When you set up OpenCode, you typically connect to one or more providers. Each connection requires its own API key. You might use Anthropic for your primary coding work because you prefer their models' quality, and OpenAI for backup or for specific tasks where GPT models excel. Or you might use OpenCode's hosted models for everything to consolidate billing and simplify configuration.

The provider/model format in agent config is always `provider/model-name`. The provider part must exactly match a connected provider's name as recognized by OpenCode. Common provider names include: `anthropic` for Anthropic models, `openai` for OpenAI models, `opencode` for OpenCode's hosted models, `google` for Gemini, `mistral` for Mistral models, and custom provider names if you set up local or internal AI services.

You can see which models are available from connected providers by running the `/models` command in OpenCode. This lists all models you can use, already formatted with the correct provider/model syntax. When you need to specify a model, you can reference this list rather than trying to remember exact model identifiers.

A practical note: provider names are case-sensitive in configuration files. Use `anthropic/claude-sonnet-4-5` not `Anthropic/Claude-Sonnet-4-5`. Model names must match exactly what the provider uses. Check the `/models` output or provider documentation for the correct identifiers.

In summary, the provider is the source of your AI capability. You connect providers with API keys. You select specific models from connected providers for your agents. Different providers offer different trade-offs in cost, speed, quality, and features. Most users start with one provider (usually Anthropic or OpenAI), connect it with their API key, use the suggested default models, and expand to other providers only if they have specific needs.

### Configuration Merging Example

The format is always provider/model-id. For Anthropic models, you might use anthropic/claude-sonnet-4-5 or anthropic/claude-opus-4. For OpenAI models, openai/gpt-5 or openai/gpt-5-mini. For OpenCode's own hosted models, opencode/gpt-5.1-codex. If you specify a model from a provider that you have not connected, the agent will not load and you will see an error message. So always ensure you have run /connect for any provider whose models you want to use.

### Custom Providers (Local or Internal APIs)

If you use local models through Ollama or LM Studio, or if you have a company-internal AI API that is not one of the major providers, you can create a custom provider configuration. This is an advanced feature that most users do not need. You would add to your opencode.json a provider section that defines how to connect to your custom AI service. The configuration includes the npm package to use for the provider, a display name, connection options like the base URL, and a models section that maps model identifiers available from that provider. Then in your agent definitions, you can reference models from that provider using the custom provider name as the prefix, such as my-company-ai/company-model. Stick with Anthropic and OpenAI unless you have a specific need for a local model or internal API that requires custom provider configuration.

### Task Permissions (Advanced Orchestration)

The permission.task setting controls which subagents an agent can invoke on its own through the Task tool. This is about orchestration: what other agents can this agent call autonomously when it decides it needs help? You might create a manager agent that only calls your vetted custom agents and does not call arbitrary subagents that users might @-mention. The configuration looks like this: permission.task with a default of deny for all, then allow for agents whose names match certain patterns, and ask for specific agents. This means your agent cannot invoke any subagent by default. It can automatically invoke subagents whose names start with orchestrator- without asking. And it can invoke the code-reviewer but must ask you first. This is useful for creating hierarchies of agents where a manager oversees what specialist subagents can be used. Important: this restricts what the agent does autonomously based on its own decisions. It does not restrict what users can do manually. Users can always manually @-mention any agent that exists. Task permissions only restrict the agent's autonomous decisions when it is trying to accomplish something on its own.

---

## Part 10: Troubleshooting - Plain English Solutions

This section provides solutions to common problems you might encounter when working with OpenCode agents. Each problem is stated clearly, followed by a checklist of things to verify in plain language.

### I Created an Agent but It Does Not Appear Anywhere

The symptoms are that your new agent does not show up in the Tab-switch rotation or in @-mention suggestions. Here is a checklist to work through in order. First, did you restart OpenCode? OpenCode loads configuration at startup and does not automatically detect new files. You need to restart OpenCode completely for new agents to be recognized. Second, is the agent mode correct? Subagents with mode subagent do not appear in Tab-switch at all. They only work via @-mention. Use mode primary or all if you want the agent to be switchable with Tab. Third, is the file in the right place? Markdown agents should be in .opencode/agents/your-agent.md for project-specific or ~/.config/opencode/agents/ for global. JSON agents should be in opencode.json with the agent definition under the agent key. Fourth, is there a syntax error? YAML frontmatter indentation matters a lot. JSON must have proper commas and braces. A small syntax error anywhere in the file can prevent the entire file from loading. Fifth, is disable set to true in the configuration? Remove it or set to false if present.

You can debug this by running opencode config show. This displays all loaded agents that OpenCode currently recognizes. Is your agent in that list? If it is not there at all, the configuration file is not loading due to location, syntax, or formatting issues. If it is there but not showing up where you expect, check the mode setting.

### My Agent Is Using the Wrong AI Model

The symptoms are that your agent uses a different model than you configured in its settings. Here are the possible causes. First, the model you specified might not be available - check the spelling and format. The format must be provider/model-id exactly. Did you connect the provider via /connect first? Without connecting the provider, models from that provider will not work. Second, the configuration might not be reloading - you need to restart OpenCode after making configuration changes for them to take effect. Third, you might have used the wrong field name. In JSON it is model not models or aiModel or model_name - it has to be exactly model. Fourth, another configuration might be overriding yours - check with opencode config show to see the effective merged configuration and trace where each setting came from.

### Agent Cannot Use a Tool I Enabled

The symptoms are that the agent fails to write files, edit files, run bash commands, or use other tools that you explicitly enabled in the tools configuration. Here are the causes to check. First, is the tool disabled at the agent level? The agent's tools setting overrides global settings. If tools.write is false, the agent cannot write even if global settings say it can. Second, is the permission set to deny? The tool may be enabled in the tools section but permission could be deny which blocks its use entirely. Third, could there be a provider limitation? Some AI models do not support certain tools, though this is rare with modern models. Fourth, if the tool comes from an MCP server, is that server running and properly connected?

Check your agent configuration for both tools and permission settings. The tools block should have the capability set to true. The permission block should have that capability set to allow or ask but not deny.

### Agent Responses Are Too Weird or Unpredictable

The symptoms are that the agent gives strange answers, goes off-topic, is inconsistent from one run to the next, and generally cannot be relied upon. The fix is to lower the temperature setting. If it is set to 0.8 or higher, try 0.4 or 0.5. Temperature above 0.6 often causes excessive randomness that makes the agent unreliable for practical work. Most tasks need temperature between 0.1 and 0.6 depending on the nature of the work. Lower temperature produces more consistent, predictable results.

### Agent Keeps Calling Subagents I Do Not Want

The symptoms are that your primary agent automatically invokes subagents for simple tasks that you did not intend it to delegate. The cause is that subagent descriptions are too broad, matching many different types of requests. The system thinks they are relevant and calls them automatically. To fix this, make the subagent descriptions more specific and narrow so they only match the tasks you truly want them to handle. You can also disable a subagent entirely if you never want it used by removing it from configuration or setting disable true. Another approach is to use @-mentions to explicitly call agents rather than relying on automatic selection, which gives you direct control. There is no setting to completely disable auto-invocation globally - you adjust by making descriptions more specific so the auto-selection logic only triggers for appropriate tasks.

### Changes to My Config File Do Not Take Effect

The symptoms are that you edited opencode.json or created a new agent file, but OpenCode still uses the old settings and does not see your changes. The cause is that OpenCode reads configuration files at startup and does not automatically watch for changes or reload them. The fix is to restart OpenCode completely. Close all OpenCode windows and reopen it. This forces it to read all configuration files again and pick up your changes.

### How Do I Know What Is Actually Configured?

The command is opencode config show. This displays the fully merged configuration after all files have been combined and overrides applied. Use it to see exactly what settings are actually in effect, including all the merging from multiple sources. This is invaluable for understanding why an agent is behaving a certain way - you can see the final values and trace which configuration file contributed each setting.

### Agent Behaves Unexpectedly or Gives Poor Results

Here is a quick checklist to run through when an agent is not performing as you hope. First, is the temperature too high? Try lowering it into the 0.3 to 0.5 range. Second, is the prompt unclear? Rewrite it with more specific instructions and examples of desired output. Third, is the wrong model being used? Try a different model - Haiku for speed, Sonnet for balanced capability, Opus for complex reasoning. Fourth, are there too few or too many steps? Adjust the steps setting if the agent stops too early or goes on too long without producing results. Fifth, are tools inappropriate? Review the tools and permission settings - maybe the agent lacks tools it needs to do the job, or has tools that distract it from its main purpose. Sixth, try a different prompt approach - be more explicit about required output format, add concrete examples in the prompt, include do not statements about what to avoid. A few iterations of prompt refinement often dramatically improve agent performance.

---

## Part 11: Quick Reference

This section collects the most frequently needed information into tables for quick lookup. Use this when you need to remember which agent does what, what temperature to use for a task, or where configuration files live.

### Which Agent for What?

The table summarizes which agent to use for different intentions. When you want to write or change code, use the build agent by default - just ask. When you want to review or analyze code, use the plan agent by pressing Tab until you see plan. When you need to explore codebase structure and find things, use the explore subagent with @explore followed by your question. When you need research on best practices or complex information synthesis, use the general subagent with @general followed by your question. For getting a summary of the current session, use the hidden summary agent with the /summary command. For creating documentation, use the custom doc-writer agent with @doc-writer. For finding security issues, use the custom security-auditor agent with @security-auditor.

### Temperature Quick Reference

The temperature setting controls AI randomness. For code review, security scanning, and documentation tasks that need consistency, use temperature between 0.1 and 0.3 because accuracy and predictability matter more than creativity. For general coding help where some variation is okay but you still want reasonable results, use temperature between 0.4 and 0.6. For brainstorming sessions, creative naming exercises, and exploring unusual approaches where you want variety, use temperature between 0.7 and 0.9.

### Tools by Agent Type

Different agent types need different tool sets. A code reviewer typically needs read, grep, and bash for searching and analysis but explicitly sets write false and edit false because it should never modify code. The typical permissions set bash to ask for destructive commands but allow git status and git diff. A builder needs most capabilities - all tools true - with permissions set to allow for safe operations and ask for destructive ones. An explorer only needs read and grep to search code, with write false, edit false, bash false, all permissions allow because read-only operations are safe. A researcher needs read, grep, and webfetch to gather information but does not need to modify anything, so write false, edit false, bash false, with webfetch set to ask since fetching from the web should be reviewed.

### Configuration File Locations

Project-specific configuration lives in opencode.json in the project root folder for consolidated agent definitions, and in .opencode/agents/*.md for individual agent files in Markdown format. Global configuration that applies to all your projects lives in ~/.config/opencode/opencode.json for consolidated definitions, and in ~/.config/opencode/agents/*.md for individual agent files. The tilde represents your home directory on Unix-like systems; on Windows the equivalent is typically in AppData.

### Common Commands

The most useful commands to remember are these. opencode agent create starts the interactive wizard for creating a new agent. opencode config show displays your merged configuration so you can see what is actually set. opencode config validate checks your configuration files for syntax errors without starting OpenCode. The /models command shows available AI models you can use. The /connect command adds provider API credentials. The Tab key cycles through primary agents in the default keybinding. The @agent-name syntax manually summons a subagent for specialized help. The /summary command gives a summary of the current session. These commands make it easy to manage and use your agent setup.

### Minimal Agent Config (Copy-Paste Template)

When you need to create a new agent quickly, start with this template in a Markdown file at `.opencode/agents/your-agent.md`:

```yaml
---
description: "Brief description of what this agent does"
mode: "subagent"  # or "primary" if you want it in Tab rotation
model: "anthropic/claude-sonnet-4-5"  # optional, uses default if omitted
temperature: 0.5  # optional, 0.1-0.3 for consistency, 0.4-0.6 for balanced, 0.7-0.9 for creative
steps: 10  # optional, max number of thinking cycles
tools:
  write: false
  edit: false
  bash: false
  read: true
  grep: true
  webfetch: false
permission:
  read: "allow"
  grep: "allow"
  edit: "ask"  # change to "deny" if agent shouldn't edit
  write: "ask" # change to "deny" if agent shouldn't write
  bash: "deny"
---
You are a [role description]. Your expertise is in [domain].

**Your methodology:**
1. [Step one]
2. [Step two]
3. [Step three]

**Output format:**
- [Structure your output]

**Constraints:**
- [What you should NOT do]

Always be [adjective] and [adjective].
```

Replace the bracketed sections with your specific requirements. This template gives you a safe, minimal agent that you can then customize for your specific needs.

---

## Part 12: Built-In Agent Summary

Let us summarize the seven built-in agents that come with OpenCode. The build agent is primary, your main coding assistant, used by just asking since it is the default. The plan agent is primary, used for analysis and review (it does not make changes unless you explicitly ask), and you access it by pressing Tab until you see plan. The general agent is a subagent for deep research on complex topics, invoked with @general. The explore agent is a subagent for finding things in the codebase, invoked with @explore. The compaction agent is hidden and automatic - it summarizes old conversations to manage context. The title agent is hidden and automatic - it generates session titles. The summary agent is hidden but accessible with the /summary command to create session summaries.

---

## Conclusion

OpenCode agents let you build a specialized AI team for your coding workflow. Start simple by using the built-in build and plan agents. Switch between them with Tab. Call specialists like @explore when you need their particular skills. Create custom agents only when you notice a recurring need for specialized help that is not covered by the built-in options. Focus on three key settings that matter most: temperature for controlling predictability versus creativity, tools for defining what the agent can do, and permission for determining when the agent needs your approval. Remember that the prompt is the single most important setting - a clear, specific, well-structured prompt on a mid-tier model will outperform a vague prompt on the most expensive model. Iterate: your first agent configuration will not be perfect. Use it, observe the results, refine the prompt. After two or three iterations you will have it dialed in. Stay safe by using ask permissions for destructive operations and by giving agents only the tools they truly need - do not enable write for reviewers. And do not overcomplicate - start with one project-specific agent in Markdown format. Add more agents as needs emerge. Use opencode config show to understand what is actually configured and to debug issues.

You now have everything you need to effectively use and customize OpenCode agents. Start with the 10-minute hands-on exercise at the beginning of this guide, then explore further as your needs grow. Happy coding.

---

## Appendix

### Glossary of Terms

@-mention is the syntax of typing @agent-name to explicitly call a subagent for specialized help rather than switching to it. An agent is a specialized AI helper with specific capabilities, instructions known as the prompt, and permissions. It is like a team member with a defined role. An API is an Application Programming Interface - how different software components communicate. You do not need to understand this deeply to use agents effectively. The bash tool allows the agent to run terminal or shell commands like ls, git status, or npm install. Configuration refers to the settings files that define agents - their descriptions, models, prompts, tools, and permissions. The edit tool allows the agent to modify existing files by making changes without overwriting the entire file. The grep tool searches for text patterns across multiple files - it is like find in files but controlled by the agent. JSON is a data format using curly braces and key-value pairs. It is one way to write agent configurations in a single file. Markdown is a simple text formatting language that supports headings, lists, and code blocks. It is also used for agent configurations when combined with YAML frontmatter. A model is the specific AI engine that powers an agent, such as claude-sonnet-4-5 or gpt-5. Different models have different capabilities, speeds, and costs. Permission controls whether agent actions need user approval - allow means automatic, ask means prompt for approval, deny means never allowed. A primary agent appears in the Tab-switch rotation so you can talk to it directly. Build and plan are primary agents. A prompt is the natural language instructions that define an agent's behavior, expertise, methodology, and output format. It is the most important configuration setting. A provider is the company that supplies AI models - Anthropic, OpenAI, OpenCode, and others. You must connect to a provider via /connect before using its models. The read tool allows the agent to view the contents of files. A subagent is a specialized agent invoked via @-mention or automatically by another agent. It typically does not appear in Tab rotation. Explore and general are subagents. Temperature is a setting from 0.0 to 1.0 that controls AI randomness. Low temperature means consistent and predictable. High temperature means creative and variable. Most tasks work best with temperature between 0.4 and 0.6. Tools are the capabilities an agent can use: write, edit, read, grep, bash, webfetch. Enable only what the agent needs for its role. The webfetch tool allows the agent to retrieve content from websites, useful for research and documentation lookup. The write tool allows the agent to create entirely new files.

### Further Reading

OpenCode's official documentation covers these additional topics in detail. The configuration schema is a complete reference for all available configuration options and their exact syntax. The providers documentation explains how to set up API connections for different AI services and how to add custom providers. The models guide helps you understand available models, their characteristics, capabilities, and pricing. The permissions system guide provides detailed explanation of permission syntax and common patterns for safe agent configurations. The tools reference lists all available tools and what they do, including any advanced options. The MCP integration documentation explains how agents use the Model Context Protocol to connect to additional tool servers. The troubleshooting guide covers general OpenCode help and common issues beyond agent-specific problems.

---

**Document version**: 2.0 (Beginner-friendly, restructured)
**Last updated**: 2025-03-14
**Intended audience**: Non-technical users, OpenCode beginners, developers new to agent customization
