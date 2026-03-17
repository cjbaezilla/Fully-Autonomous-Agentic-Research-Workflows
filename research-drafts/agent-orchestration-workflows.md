# Agent Orchestration Workflows for OpenCode Extensions

## Overview

Optimal agent orchestration requires a modular, event-driven architecture with clear separation of concerns. The system should support both sequential pipelining and parallel execution patterns.

## Core Agent Types

### Research Agents
- `researcher`: Gathers information from multiple sources, validates facts
- `synthesizer`: Combines research into coherent summaries
- `fact-checker`: Verifies claims against trusted sources

### Development Agents
- `architect`: Designs system structure and component relationships
- `implementer`: Writes code followingpatterns and conventions
- `reviewer`: Performs code review, suggests improvements
- `tester`: Creates and executes test cases
- `documenter`: Generates API docs and usage guides

### Writing Agents
- `outliner`: Creates structured article outlines
- `draft-writer`: Generates content sections
- `editor`: Polishes language, improves clarity
- `seo-optimizer`: Enhances content for search
- `format-applyer`: Applies style guidelines

### Meta-Agents
- `orchestrator`: Manages workflow execution, task allocation
- `monitor`: Tracks agent health, performance metrics
- `learning-coordinator`: Analyzes past runs, improves future behavior

## File System Structure for Autopilot

```
research-drafts/
├── heartbeat.md                    # System status & health
├── knowledge-base/
│   ├── patterns/                  # Code & workflow patterns
│   ├── failures/                  # Learned mistakes
│   ├── successes/                 # Effective approaches
│   └── context/                   # Domain-specific knowledge
├── workflows/
│   ├── research-workflow.yaml    # Research pipeline definition
│   ├── dev-workflow.yaml         # Development pipeline
│   └── article-workflow.yaml     # Writing pipeline
├── agents/
│   ├── researcher/
│   │   └── config.yaml
│   ├── developer/
│   │   └── config.yaml
│   └── writer/
│       └── config.yaml
├── session/
│   ├── active-sessions.json
│   └── session-{id}/
│       ├── state.json
│       ├── logs/
│       └── artifacts/
└── metrics/
    ├── performance.json
    ├── errors.json
    └── learning-curve.json
```

## Heartbeat Mechanism

The `heartbeat.md` file serves as the system's pulse:

```markdown
# System Heartbeat

## Current Status
- State: running/paused/error/idle
- Active Sessions: 3
- Queue Depth: 12
- Last Update: 2026-03-17T10:30:00Z

## Agent Health
| Agent | Status | Last Active | Success Rate |
|-------|--------|-------------|--------------|
| researcher | healthy | 2m ago | 94% |
| developer | degraded | 15m ago | 78% |
| writer | healthy | 5m ago | 96% |

## Resource Usage
- Memory: 2.3GB / 8GB
- CPU: 34%
- Disk IO: 12MB/s

## Recent Events
- 10:28: Completed article draft for "X"
- 10:25: Error in test-runner: timeout
- 10:20: Checkpoint created

## Alerts
⚠️ developer agent success rate dropped below 80%
```

## Recursive Learning Loop

### 1. Execution Tracking
Every agent run logs:
- Input parameters
- Output results
- Duration & resource usage
- Success/failure indicators
- User feedback (if available)

### 2. Pattern Extraction
Nightly batch job analyzes logs to:
- Identify recurring failure modes
- Extract successful strategies
- Update agent configurations
- Refine workflow definitions

### 3. Knowledge Persistence
Learnings stored as:
- **YAML patterns**: "When X happens, do Y"
- **Embedding vectors**: Similarity-based retrieval
- **Decision trees**: Conditional logic flows
- **Confidence scores**: Weight future decisions

### 4. Autopilot Mode
System can:
- Auto-select optimal agents for tasks
- Adjust timeout/retry parameters
- Route around known failure modes
- Suggest workflow optimizations

## Workflow Definition Format

```yaml
workflow:
  name: "research-to-article"
  description: "End-to-end content creation"
  version: "1.2.0"
  
  agents:
    - id: researcher
      role: primary
      inputs: [topic]
      outputs: [research-notes]
      max_retries: 3
      fallback: web-search-agent
    
    - id: synthesizer
      role: secondary
      inputs: [research-notes]
      outputs: [outline]
      depends_on: [researcher]
    
    - id: outliner
      role: tertiary
      inputs: [outline]
      outputs: [structure]
      depends_on: [synthesizer]
    
    - id: draft-writer
      role: primary
      inputs: [structure]
      outputs: [draft]
      parallel: true  # Can run with editor
    
    - id: editor
      role: quality
      inputs: [draft]
      outputs: [polished]
      parallel: true

  coordination:
    strategy: "pipeline-with-parallel"
    checkpoint_after: [researcher, outliner]
    rollback_on_failure: true
    timeout: 1800  # seconds

  learning:
    track_metrics: [duration, quality_score, retries]
    feedback_loop: "session-completion"
    auto_optimize: true
```

## Event-Driven Communication

Agents communicate via JSON event bus:

```json
{
  "event_id": "evt_123",
  "type": "task.completed",
  "timestamp": "2026-03-17T10:30:00Z",
  "source": "agent.researcher",
  "payload": {
    "task_id": "task_456",
    "result": { ... },
    "metrics": { "duration": 45.2, "tokens": 1234 }
  }
}
```

## Implementation Priorities

1. **Phase 1**: Core orchestration with manual workflows
2. **Phase 2**: Automatic error recovery & retries
3. **Phase 3**: Basic learning (success/failure tracking)
4. **Phase 4**: Predictive agent selection
5. **Phase 5**: Full autopilot with self-modification

## OpenCode Integration Points

- **Skills**: Wrap agents as callable skills
- **Tools**: Expose orchestration commands (`/orchestrate`, `/workflow status`)
- **Context**: Store session state in workspace
- **Commands**: `/agent health`, `/workflow optimize`, `/learning insights`

## Key Metrics to Track

- Agent success rate by domain/task type
- Workflow completion time trends
- Resource efficiency (tokens/time)
- User satisfaction scores
- Autonomous correction rate
- Pattern discovery count

## Anti-Patterns to Avoid

❌ Hard-coded agent chains (no flexibility)
❌ Monolithic agents (do too much)
❌ Silent failures (no heartbeat updates)
❌ State loss between sessions
❌ Learning without validation (overfitting)