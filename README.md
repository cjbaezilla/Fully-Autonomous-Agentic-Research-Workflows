# Fully Autonomous Agentic Research Workflows

## Introduction

This repository demonstrates how to configure and deploy specialized AI agents that can operate autonomously to perform complex research and development tasks. It serves as a practical proof-of-concept for the OpenCode SDK, showing how multiple custom agents with distinct personalities and capabilities can work together in a coordinated workflow.

Imagine having a team of expert assistants, each with their own specialty, that you can delegate tasks to. One might be a technical writer who can explain complex ideas in simple terms, another might be a Git expert who manages version control with precision. This project shows you how to set up such a system using OpenCode, allowing these AI agents to work independently on their assigned tasks while maintaining safety and oversight.

The repository contains everything needed to understand and reproduce this agent configuration, including detailed documentation, custom agent prompts, and the main configuration file that brings it all together.

## Understanding the OpenCode Framework

OpenCode is a framework that transforms how developers interact with artificial intelligence throughout the software development lifecycle. Rather than having a single, generic AI assistant that attempts everything, OpenCode allows you to create multiple specialized agents, each optimized for specific types of tasks. Think of it like assembling a professional team where each member has a clearly defined role: you have architects who plan but don't build, builders who implement, researchers who investigate, and reviewers who analyze without modifying.

The framework operates around two fundamental agent types. Primary agents are your main conversational partners—the ones you directly interact with during a coding session. They have full access to tools and can respond to your questions, execute your requests, and orchestrate other agents when needed. Subagents, by contrast, are specialists that get called in for particular tasks. They work behind the scenes, handling specific problems that match their expertise, and then return their results to the primary agent. This division creates an efficient workflow where you talk to one person (the primary agent) who knows when to bring in the right specialist for the job.

Agents in OpenCode are defined through configuration files that specify their capabilities, limitations, and personalities. You can configure which AI model powers each agent, what tools they have access to, what permissions they operate under, and most importantly, the system prompt that defines their behavior and expertise. This configuration can live in JSON files for centralized management or in individual Markdown files for modular organization.

## The Custom Agents in This Repository

This proof-of-concept repository features two custom subagents that showcase the breadth of capabilities possible with OpenCode. Each agent has a distinct personality, skill set, and use case that demonstrates different aspects of autonomous operation.

### Documentor: The Technical Writing Specialist

The Documentor agent is designed to write clear, accessible documentation for any audience. It operates under a specific philosophy: complex topics should be explained in simple terms without assuming any prior technical knowledge. This agent is built to transform technical concepts into understandable explanations using everyday language, analogies, and real-world examples. It can write README files, API documentation, user guides, technical tutorials, and code comments that prioritize clarity over complexity.

What makes Documentor particularly interesting is its writing methodology. It follows a structured approach that includes starting with a warm, welcoming introduction that establishes the topic in approachable terms. It avoids bullet points entirely, instead using extended paragraphs that explore concepts from multiple angles. When technical terms are unavoidable, it introduces them with plain-language definitions first. The agent maintains a grounded, practical tone without excessive enthusiasm, focusing on usefulness rather than excitement.

In practice, Documentor can be invoked manually by typing `@documentor` in a message, or it can be called automatically by a primary agent when documentation work is detected. It has full access to file creation and editing tools, allowing it to produce complete documentation files independently. However, it can also be configured to ask for approval before making changes, providing a safety mechanism for production environments.

### GitMasters: The Version Control Expert

The GitMasters agent specializes in all things Git. It possesses comprehensive knowledge of repository management, branching strategies, merging techniques, conflict resolution, and best practices for collaborative development. This agent can perform any Git operation autonomously, from basic commands like `git status` and `git commit` to advanced workflows involving rebasing, cherry-picking, and complex merges.

GitMasters operates with built-in safety protocols that reflect the potentially destructive nature of Git operations. Before performing actions that could lose data, it checks the current repository state, warns about potential issues, and can be configured to request explicit confirmation. It follows best practices such as keeping working directories clean, writing meaningful commit messages, and avoiding direct commits to main branches. The agent understands when to use safer alternatives like `git revert` instead of destructive resets, and it systematically handles merge conflicts by identifying affected files and guiding through resolution steps.

The agent's prompt includes extensive command templates and examples, covering everything from initialization and cloning to advanced operations like stashing, tagging, and history inspection. It maintains clear communication throughout its operations, reporting what it's about to do, showing relevant command output, and clearly indicating success or failure. When errors occur, it analyzes the problem and suggests solutions.

## Repository Structure

The repository is organized to be both self-documenting and easy to navigate. Each component serves a specific purpose in the overall configuration.

At the root level, you'll find `opencode.json`, the main configuration file that defines the two custom agents and their behavior. This file follows the OpenCode schema and specifies each agent's model, description, tools, permissions, and link to their prompt files. It also contains provider configurations for OpenRouter, which supplies the AI models used by the agents.

The `prompts/` directory contains separate text files for each agent. `documentor.txt` holds the comprehensive instructions that give the Documentor agent its writing philosophy and methodology. `gitmasters.txt` contains the detailed specifications that make GitMasters an expert in version control operations. These prompt files are loaded by the main configuration using the `{file:./prompts/filename.txt}` syntax.

The `research-drafts/` directory contains `opencode_agents.md`, an extensive reference guide that documents the entire OpenCode agent system. This file explains the difference between primary and subagents, covers all built-in agents, details every configuration option, provides use case examples, and serves as a comprehensive learning resource. While this repository is a proof-of-concept, the reference guide is production-quality documentation that could be useful for any OpenCode user.

Standard Git infrastructure files like `.gitignore` and the `.git/` directory handle version control. The repository is itself a Git repository, allowing you to track changes to the configuration and documentation over time.

## Getting Started with This Configuration

To use this agent configuration, you'll need to have OpenCode installed on your system. OpenCode is available through various installation methods depending on your operating system, and the framework requires an API key from an AI provider to function.

The first step is to review the `opencode.json` configuration file. This file references OpenRouter as the provider, with multiple model options configured including StepFun's Step 3.5 Flash, Hunter Alpha, Nemotron, and Trinity. The API keys in the configuration are placeholders and should be replaced with your own valid credentials. You'll need to obtain these keys from OpenRouter and update the provider configuration accordingly.

Once the provider credentials are configured, you can launch OpenCode in this repository directory. When OpenCode starts, it will automatically detect the `opencode.json` file and load the custom agent definitions. You'll then have access to the `documentor` and `gitmasters` agents as subagents that can be invoked using the `@` syntax.

To test the Documentor agent, you could ask a primary agent (like the default Build agent) to write documentation for your project using a command like: "Use @documentor to create a README that explains this project to non-technical users." The Documentor agent will be invoked automatically and will produce a draft according to its writing guidelines.

To test GitMasters, you could ask: "Use @gitmasters to create a new branch, commit all current changes, and push to origin." The agent will execute these Git operations autonomously, following its safety protocols and best practices.

## The Proof-of-Concept Nature

This repository is explicitly designed as a proof-of-concept to demonstrate the feasibility and power of autonomous agentic workflows. It shows that with proper configuration and prompting, AI agents can operate independently on specialized tasks without constant human oversight. The two agents here represent different domains—technical writing and version control—but the pattern extends to many other areas like security auditing, code review, testing, deployment, and research.

The proof-of-concept nature means that while the configurations work and demonstrate the concept, they are not necessarily production-ready without further customization. The prompts are designed to be instructive examples rather than exhaustive specifications. Real-world deployment would involve refining the prompts based on specific organizational needs, adjusting permission levels to match security requirements, and potentially adding additional agents to cover other parts of the development workflow.

What this project proves is that the OpenCode SDK provides a robust foundation for building autonomous AI systems that can handle specialized tasks with minimal human intervention. The agents can be invoked programmatically via the Task tool, they maintain appropriate safety boundaries through permission systems, and they can collaborate by passing work between primary and subagents. This creates a workflow where complex tasks can be decomposed and handled by appropriate specialists, much like how human teams operate.

## Extending and Customizing

The repository is structured to make customization straightforward. To create your own specialized agents, you would add new entries to the `agent` section of `opencode.json` or create separate Markdown files in an agents directory. The key is crafting an effective system prompt that clearly defines the agent's role, methodology, constraints, and expected output format.

You might extend this configuration by adding agents for security auditing, performance optimization, API design, test generation, or any other domain-specific task. Each agent should have a clear description that helps primary agents recognize when to invoke it, appropriate tool and permission settings that balance capability with safety, and a well-crafted prompt that guides its behavior.

The model configuration is also flexible. While this proof-of-concept uses StepFun's Step 3.5 Flash (a free model), you can assign different models to different agents based on their needs. Some tasks benefit from more capable, expensive models, while others can use faster, cheaper ones. You might use a high-end model for architectural planning and a lightweight model for routine file operations.

## Safety and Oversight Considerations

Autonomous operation introduces important safety considerations that this configuration addresses through multiple layers. The permission system ensures that agents can't perform destructive operations without approval. You can set individual tools to deny, ask, or allow access, with "ask" being the prudent choice for potentially harmful operations like file deletion or command execution. Command-specific permissions allow fine-grained control, such as allowing read-only Git commands while requiring approval for force pushes.

The subagent architecture itself provides safety. Subagents operate under the oversight of primary agents, and their work is visible in the conversation. You can navigate into subagent sessions to monitor their progress and intervene if necessary. This transparency prevents the "black box" problem where agents work without any visibility into their actions.

Finally, the use of version control through Git means that all changes made by agents can be reviewed, reverted, or audited later. The GitMasters agent enforces disciplined Git practices, ensuring that work happens on branches, commits are well-documented, and the main branch remains stable. This creates a safety net where mistakes can be identified and corrected before they cause problems.

## Ready to Publish Drafts

The following research drafts are complete and ready for publication:

- [OpenCode Agents Fundamentals: Create, Configure, Deploy Custom Agents and Automated Workflows](research-drafts/opencode_agents.md)
- [A Beginner's Guide to DEX and AMM: Trading Without Middlemen](research-drafts/uniswap_amm_dex.md)

## Future to Publish Drafts

- [Ethereum Cryptography](research-drafts/ethereum_cryptography.md)
- [Karpathy Autoresearch](research-drafts/karpathy_autoresearch.md)

## Conclusion

This repository provides a working example of how to configure specialized AI agents for autonomous operation using the OpenCode SDK. It demonstrates the core concepts of agent architecture, configuration, and collaboration while providing practical agents for technical writing and Git operations. The comprehensive documentation in `opencode_agents.md` offers deeper guidance for those wanting to extend the configuration or build their own custom agents.

As AI-assisted development continues to evolve, the ability to create specialized, autonomous agents will become increasingly valuable. This proof-of-concept shows that such systems are not only possible but practical, providing a template you can adapt for your own development workflows. The combination of clear agent definitions, appropriate safety measures, and thoughtful prompting creates a foundation for truly agentic systems that can handle complex, multi-step tasks with minimal human intervention.