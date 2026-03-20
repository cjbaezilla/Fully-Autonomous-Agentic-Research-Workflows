# Beads + Dolt Integration for Agentic Workflows

## Executive Summary

**Beads** is a distributed graph issue tracker built specifically for AI agents, powered by **Dolt** - a SQL database with Git-style version control. Together, they form a powerful system for implementing long-horizon, persistent agent workflows with full auditability, branching, merging, and collaborative capabilities.

### Core Technologies

- **Dolt**: Git-for-data SQL database (MySQL-compatible) with cell-level versioning, branching, and merging
- **Beads**: Task/graph management CLI built on Dolt, providing hash-based IDs, dependencies, and agent-optimized JSON output

---

## Architecture Analysis

### Dolt: The Storage Backbone

Dolt provides the foundational storage layer with:
- **Version Control for Data**: Every change is a commit with full history
- **Branching & Merging**: Isolate experimental work, test changes, merge results
- **Cell-Level Lineage**: Track exactly how each cell changed across commits via `dolt_history_<table>` and `dolt_diff_<table>`
- **SQL Interface**: Standard MySQL protocol with version-control extensions (`dolt_add`, `dolt_commit`, `dolt_merge`, etc.)
- **CLI Parity**: Git-like commands (`dolt status`, `dolt diff`, `dolt log`, `dolt checkout`, `dolt push/pull`)
- **Conflict Resolution**: Built-in merge conflicts on table rows, just like Git file conflicts
- **Storage Format**: Single `.dolt` directory per database, self-contained (or separate directory structure)

### Beads: The Task Graph Layer

Beads builds on Dolt to provide:
- **Task Tracking**: Create, update, close tasks with metadata (title, description, priority, assignee, status)
- **Dependency Graph**: Relate tasks via `blocks`, `depends_on`, `relates_to`, `duplicates`, `supersedes`, `replies_to`
- **Hierarchical IDs**: Auto-generated hash IDs (`bd-<8char>`) prevent collision; supports epics (`bd-xyz.1`) and subtasks (`bd-xyz.1.1`)
- **Agent Optimization**: JSON output, `--json` flag, machine-readable responses
- **Auto-Ready Detection**: `bd ready` shows tasks with no open blockers (no `blocks` relationships to open tasks)
- **Compaction/Memory Decay**: Summarize old closed tasks to reduce context window usage
- **Messaging System**: `--thread` for threaded discussions, ephemeral tasks, mail delegation
- **Stealth Mode**: Run without committing to project git (`--stealth`) or as contributor (`--contributor`)
- **Multi-Repo Federation**: Sync planning across multiple repositories

---

## Key Integration Patterns for Long-Horizon Workflows

### 1. Persistent State Across Sessions

**Problem**: Agents lose context between invocations or after crashes.

**Solution**: Use Beads tasks to persist agent state, progress, and intermediate results. Dolt ensures full history and rollback.

**Implementation**:
```bash
# Initialize Beads in project (once)
cd my-project
bd init

# Agent creates task for the overall goal
bd create "Implement OAuth2 authentication" -p 0 --description "Complete implementation with tests"

# Result: Task bd-a1b2c3d4 created with full audit trail in Dolt
```

Agent stores all state in Beads:
- Current task ID in working memory
- Progress updates via `bd update <id> --message "Partial result..."`
- Attach artifacts via description updates or external storage with links

**Benefits**:
- Recovery: Any agent can pick up where another left off by reading tasks
- Audit: Complete history of what was attempted and why
- Collaboration: Multiple agents can claim and work on different tasks

---

### 2. Hierarchical Task Decomposition

**Problem**: Long-horizon tasks need to be broken down into manageable, trackable subtasks.

**Solution**: Use Beads' hierarchical IDs and dependency graph.

**Implementation**:
```bash
# Create epic
EPIC=$(bd create "Build ML pipeline" -p 0 --json | jq -r '.id')
# Results in: bd-abc12345

# Create subtasks with hierarchical IDs
bd create "Data preprocessing" -p 1 --parent $EPIC --description "Clean and validate dataset"
# Result: bd-abc12345.1

bd create "Feature engineering" -p 1 --parent $EPIC.1
# Result: bd-abc12345.1.1

bd create "Model training" -p 0 --parent $EPIC
# Result: bd-abc12345.2

# Link dependencies
bd dep add bd-abc12345.2 bd-abc12345.1.1  # training depends on feature engineering
```

Agent workflow:
1. Query `bd ready` to find tasks with no open blockers
2. Claim a task: `bd update <id> --claim`
3. Work on task, update progress
4. Close task: `bd close <id> "Completion message"`
5. Dependencies auto-resolve for downstream tasks

---

### 3. Multi-Agent Collaboration with Conflict Prevention

**Problem**: Multiple agents working on same project cause conflicts and duplicate work.

**Solution**: Beads uses hash-based IDs (`bd-<8random>`) guaranteeing no collisions across branches/repos. Combine with Dolt branching to isolate work.

**Implementation**:

```bash
# Each agent creates their own branch
dolt checkout -b agent-1-work
# Or use Beads contributor mode
bd init --contributor  # Saves planning to separate repo

# Agent claims tasks atomically
bd update bd-abc12345 --claim  # Sets assignee, status=in_progress in single transaction

# Work happens on feature branch
dolt status  # Shows task changes
dolt commit -m "Agent-1: Completed data validation"

# When ready, merge back to main
dolt checkout main
dolt merge agent-1-work --no-ff
# Dolt handles row-level conflicts if two agents modified same task
```

**Benefits**:
- No ID collisions even across distributed teams/agents
- Branch isolation prevents stepping on toes
- Merge conflicts resolvable at task level
- Full provenance tracking per agent

---

### 4. Long-Running Workflows with Memory Compaction

**Problem**: After many tasks, context window fills up with stale closed tasks.

**Solution**: Beads provides compaction (semantic memory decay) to summarize old closed tasks.

**Implementation**:
```bash
# Periodically run compaction (manual or automated)
bd compact --older-than 30d --keep-count 100

# Or use daemon mode with auto-compaction
bd daemon start --auto-compact=30d
```

Compaction creates summary tasks that replace individual old tasks in queries, preserving essential information while reducing noise.

**Agent Considerations**:
- Before querying all tasks, check if compaction is needed
- Use `bd ready --no-compact` to see raw tasks
- Summaries include links to original tasks if details needed

---

### 5. Experimental Branching and Rollback

**Problem**: Agents need to try approaches that might fail, without losing good work.

**Solution**: Use Dolt's branching to isolate experiments; rollback with `dolt reset --hard` if needed.

**Implementation**:
```bash
# Save current state
dolt branch experiment/new-algorithm

# Try something risky
dolt checkout experiment/new-algorithm
bd create "Test radical approach" -p 2 --description "Try transformer instead of LSTM"
# ... agent works ...
dolt commit -m "Experiment: Results with transformer"

# Compare results
dolt diff main experiment/new-algorithm --data-only

# If experiment fails
dolt checkout main
dolt branch -D experiment/new-algorithm  # Delete branch

# If experiment succeeds
dolt checkout main
dolt merge experiment/new-algorithm
dolt commit -m "Replace LSTM with transformer based on experiment bd-xyz789"
```

**Agent Pattern**:
- Create "experiment" tasks with explicit acceptance criteria
- Work on separate branch
- Use `dolt diff` to generate comparison reports
- Merge only if criteria met, otherwise rollback

---

### 6. Complete Audit Trail and Reproducibility

**Problem**: Need to understand why decisions were made and reproduce results.

**Solution**: Dolt's cell-level history + Beads' task audit trail.

**Implementation**:
```bash
# See full history of a task's changes
dolt sql -q "SELECT * FROM dolt_log WHERE message LIKE '%bd-abc%' ORDER BY date DESC"

# See who changed what in a task
dolt sql -q "SELECT * FROM dolt_diff_tasks WHERE from_id='bd-abc12345' OR to_id='bd-abc12345'"

# Time travel: See state of all tasks at a specific commit
dolt sql -q "SELECT * FROM tasks AS OF 'bd-abc12345.2.commit_hash'"

# Reconstruct agent's thought process
bd show bd-abc12345 --show-history  # Shows all updates with timestamps and authors
```

**Reproducibility**:
- Any commit hash can be checked out to reproduce exact state
- Branch names can be tagged for release snapshots
- All changes are signed with user.email/name (configuration required)

---

## Use Cases and Implementation Patterns

### Use Case 1: Autonomous Software Development Agent

**Scenario**: Agent builds features from specification to deployment without human intervention.

**Implementation**:
```bash
# Project setup (one-time)
bd init
cat <<EOF > .agent/workflows/dev.yml
agent:
  task_tracker: beads
  state_persistence: dolt
  workflow:
    - analyze_requirements
    - design_architecture
    - implement_code
    - write_tests
    - run_ci
    - deploy
EOF

# Agent workflow
# 1. Create master task
MASTER=$(bd create "Implement user profile page" -p 0 --json | jq -r '.id')

# 2. Decompose
bd create "Design API contract" -p 1 --parent $MASTER
bd create "Implement React components" -p 1 --parent $MASTER
bd create "Write unit tests" -p 2 --parent $MASTER
bd create "E2E tests" -p 2 --parent $MASTER

# 3. Dependencies
bd dep add bd-xyz123 bd-abc456  # React impl depends on API design

# 4. Agent executes tasks in order (ready tasks first)
while TASK=$(bd ready --json | jq -r '.[0].id'); do
  [ -z "$TASK" ] && break
  bd update $TASK --claim --message "Agent starting work"
  # ... perform actual work (generate code, run tests, etc.) ...
  bd update $TASK --message "Implementation complete" --status in_review
  # ... optionally request human review ...
  bd close $TASK "Deployed to staging"
done
```

**CLI Commands for Agent**:
```bash
# Agent uses these commands programmatically
bd init                                    # Initialize project
bd create "Title" -p <0-3> [--parent ID] [--description TEXT]
bd update <id> --claim                    # Atomic claim (assignee + in_progress)
bd update <id> --message "update" [--status status]
bd dep add <child> <parent>               # Establish dependency (child blocks on parent)
bd ready --json                           # List tasks ready to work on (no blockers)
bd show <id> --json                       # Get task details
bd close <id> "reason"                   # Mark completed
bd compact --older-than 7d --keep-count 50  # Memory management
```

---

### Use Case 2: Multi-Agent Research Project

**Scenario**: Multiple specialized agents collaborate on research paper (literature review, experiments, writing).

**Implementation**:
```bash
# Master orchestrator creates task graph
bd init
PAPER=$(bd create "Research: Neural Architecture Search" -p 0 --json | jq -r '.id')

# Create agent-specific branches
dolt checkout -b literature-agent
LIT=$(bd create "Literature review" -p 1 --parent $PAPER --json | jq -r '.id')
dolt commit -m "Assign literature review to agent-1"

dolt checkout -b experiment-agent
EXP=$(bd create "Run experiments" -p 1 --parent $PAPER --json | jq -r '.id')
dolt commit -m "Assign experiments to agent-2"

dolt checkout main

# Agents work independently on their branches
# Agent-1:
dolt checkout literature-agent
bd update $LIT --claim --assignee agent-1
# ... research papers, summarize ...
bd close $LIT "Completed review of 50 papers"
dolt commit -m "Agent-1: Literature review done"

# Agent-2:
dolt checkout experiment-agent
bd update $EXP --claim --assignee agent-2
# ... run experiments, record results in Dolt tables ...
bd update $EXP --message "Baseline accuracy: 87%" --status in_progress
# ... more iterations ...
bd close $EXP "Best model: 92.4% accuracy"
dolt commit -m "Agent-2: Experiments complete"

# Merge results
dolt checkout main
dolt merge literature-agent experiment-agent
dolt commit -m "Merge agent results"

# Generate paper (writing agent)
WRITE=$(bd create "Write paper" -p 1 --parent $PAPER --json | jq -r '.id')
# ... writing agent reads literature and experiment results from Dolt ...
bd close $WRITE "Paper draft ready for review"
```

**Benefits**:
- Parallel work streams on separate branches
- Dolt handles automatic merges when no conflicts
- Row-level conflicts if two agents edit same task (rare)
- Full provenance of which agent did what

---

### Use Case 3: CI/CD Pipeline with Stateful Workflows

**Scenario**: Complex deployment pipeline requiring state across stages (e.g., blue-green, canary, feature flags).

**Implementation**:
```bash
# Pipeline state stored in Dolt
bd init
bd create "Deploy v2.0" -p 0
DEPLOY=$(bd show last --json | jq -r '.id')

# Each CI stage updates state
bd update $DEPLOY --message "Build started" --status building
# CI runner:
dolt sql -q "INSERT INTO build_artifacts (commit, image, status) VALUES ('$COMMIT', '$IMAGE', 'pending')"
dolt commit -m "CI: Build triggered"

bd update $DEPLOY --message "Tests passed" --status testing
# ... tests run ...

bd update $DEPLOY --message "Deployed to staging" --status deployed_staging
dolt sql -q "CALL dolt_add('deployments'); CALL dolt_commit('-m', 'Staging deployment')"

# Approval gate (human or automated policy check)
APPROVAL=$(bd create "Approve production deployment" -p 0 --parent $DEPLOY --json | jq -r '.id')
# ... wait for approval task to be closed ...

bd update $DEPLOY --message "Production deployment approved" --status deploying
# ... deploy ...

bd close $DEPLOY "Production deployment complete"
```

**Pipeline State Queries**:
```sql
-- SQL: Get current deployment status for all services
SELECT * FROM tasks WHERE title LIKE 'Deploy%' AND status != 'closed';

-- Get exact state at last successful deployment
SELECT * FROM tasks AS OF (SELECT hash FROM dolt_log WHERE message LIKE '%Production deployment complete%' LIMIT 1);
```

---

### Use Case 4: Fault-Tolerant Long-Running Agents

**Scenario**: Agent runs for hours/days generating code, needs to survive crashes and restart.

**Implementation**:
```bash
# Agent initializes and creates session task
SESSION=$(bd create "Session: 2025-03-20-code-gen" -p 1 --description "Generate API clients" --json | jq -r '.id')

# Periodic checkpoints
trap 'save_checkpoint $SESSION' INT TERM EXIT

save_checkpoint() {
  bd update $SESSION --message "Checkpoint: processed $COUNT files, last: $LAST_FILE"
  dolt commit -m "Checkpoint at $(date)"
  # Optionally push to remote
  dolt push origin $SESSION_BRANCH
}

# Work loop
for file in $(find src -name "*.go"); do
  # ... generate code ...
  bd update $SESSION --message "Completed $file"
  dolt add -A
  dolt commit -m "Generated $file"
done

bd close $SESSION "All files generated successfully"
```

**Recovery**:
```bash
# On restart, find last session task
LAST_SESSION=$(bd ready --json | jq -r '.[] | select(.title | startswith("Session:")) | .id' | head -1)
if [ -n "$LAST_SESSION" ]; then
  # Find last commit with that session
  LAST_COMMIT=$(dolt log --grep "$LAST_SESSION" --oneline | head -1 | cut -d' ' -f1)
  dolt checkout $LAST_COMMIT
  # Resume work
fi
```

---

### Use Case 5: Federated Planning Across Repositories

**Scenario**: Organization maintains separate repos for microservices, but needs coordinated planning.

**Implementation**:
```bash
# Central planning repo (or DoltHub remote)
# Each service repo:
cd service-a
bd init --contributor  # Points to shared planning database

# Developers create tasks that appear in central view
bd create "Add OAuth support" -p 1 --description "Needs changes in API and DB"
# Task appears in central dashboard that aggregates all contributor repos

# Central maintainer reviews and creates epics spanning services
bd init  # As maintainer
EPIC=$(bd create "OAuth Rollout" -p 0 --json | jq -r '.id')
bd dep add bd-xyz123 bd-abc456  # Links task from service-a to service-b

# Beads federation syncs across repos automatically via remote
bd sync --all  # Pulls changes from planning remote
```

**Federation Setup**:
```bash
# In each contributor repo
bd config --global remote.planning.url https://dolthub.com/org/planning
bd init --contributor
bd push planning main  # Sends tasks to central planning DB

# Central planning repo
dolt remote add planning https://dolthub.com/org/planning
dolt fetch planning
dolt merge planning/main  # Aggregates all contributor tasks
```

---

## CLI Reference & Usage Patterns

### Beads Commands (Agent-Focused)

**Task Management**:
```bash
bd create "Title" -p <priority 0-3> [--description "text"] [--parent <parent-id>]
  Creates new task with auto-generated ID. Returns JSON with {id, title, status, etc.}

bd update <id> --claim [--assignee <name>]
  Atomically set assignee and status=in_progress. Prevents double-claiming.

bd update <id> --message "Update text" [--status <status>]
  Add comment and optionally change status (open, in_progress, in_review, blocked, closed).

bd close <id> "Completion reason"
  Set status=closed. Optionally include completion details.

bd show <id> [--show-history] [--json]
  View task details, dependencies, audit trail.

bd dep add <child-id> <parent-id>
  Create dependency: child blocks on parent. Types: blocks, depends_on, relates_to, etc.
```

**Querying**:
```bash
bd ready [--json] [--no-compact]
  List tasks with no open blockers (ready to work on).

bd show <id> --show-reverse-deps
  Show what tasks are blocked by this task.

bd search "query" [--status open|closed|all] [--json]
  Full-text search across tasks.
```

**Maintenance**:
```bash
bd compact --older-than <duration> [--keep-count <N>] [--dry-run]
  Summarize old closed tasks to reduce context size. Creates summary task per epic.

bd daemon start [--auto-compact=<duration>] [--port <port>]
  Run background daemon for MCP server, auto-compaction, etc.

bd sync [--all] [--prune]
  Sync with configured remote (for contributor mode).
```

**Setup**:
```bash
bd init [--stealth] [--contributor] [--quiet]
  Initialize Beads in current project. Options:
    --stealth: Don't install git hooks, store .beads locally
    --contributor: Route planning to separate repo (for open-source contributors)
    --quiet: Minimal output

bd config [--global] <key> <value>
  Configure: remote.*, user.*, beads.role (maintainer|contributor), etc.
```

### Dolt Commands (Dolt Operations)

**Git-style**:
```bash
dolt init                    # Create new database directory
dolt status                  # Show modified tables
dolt add <table>             # Stage table changes
dolt commit -m "message"     # Commit staged changes
dolt log [--oneline]         # View commit history
dolt diff [table]            # Show changes in working set
dolt diff <branch1> <branch2> [<table>]  # Compare branches
dolt checkout <branch> [-b]  # Switch/create branch
dolt merge <branch>          # Merge branch into current
dolt branch [-d <name>]      # List/create/delete branches
dolt reset --hard            # Discard all uncommitted changes
dolt cherry-pick <commit>    # Apply commit from another branch
dolt revert <commit>         # Create new commit that undoes changes
```

**SQL Interface**:
```bash
# Start server (run in background)
dolt sql-server &

# Run queries
dolt sql -q "SELECT * FROM tasks WHERE status='open';"
# Or interactive:
dolt sql
mysql> SELECT * FROM dolt_log LIMIT 10;

# Version control in SQL
mysql> CALL dolt_add('tasks', 'dependencies');
mysql> CALL dolt_commit('-m', 'Updated task graph');
mysql> CALL dolt_merge('feature-branch');
mysql> SELECT * FROM tasks AS OF 'main~2';  -- Time travel

# System tables
dolt_status       -- Shows working table changes
dolt_log          -- Commit history
dolt_branches     -- All branches
dolt_diff_<table> -- Row-level diffs for specific table
dolt_history_<table> -- Full history of specific table
```

**Data Management**:
```bash
dolt table ls                         # List all tables
dolt table export <table> <file.csv>  # Export table
dolt table import <table> <file.csv>  # Import CSV
dolt schema import <file.sql>         # Import schema
dolt clone <remote-url> [<dir>]       # Clone remote database
dolt push <remote> <branch>           # Push commits
dolt pull <remote> <branch>           # Fetch and merge
dolt remote add <name> <url>          # Configure remote
```

---

## Best Practices for Agent Implementation

### 1. Idempotent Operations

Agents should handle re-runs gracefully:
```bash
# Instead of blindly creating, check if exists
if ! bd show $TASK_ID >/dev/null 2>&1; then
  bd create "..." -p 1 --parent $PARENT
fi
```

### 2. Transactional Updates

Wrap critical sequences in Dolt transactions via `dolt sql`:
```bash
dolt sql -q "
START TRANSACTION;
CALL dolt_add('tasks');
CALL dolt_commit('-m', 'Bulk update');
COMMIT;
"
```

### 3. Branch Hygiene

- Create feature branches for each major task: `dolt checkout -b feature/xyz`
- Delete branches after merge: `dolt branch -d feature/xyz`
- Use naming conventions: `agent/<agent-name>`, `experiment/<desc>`, `hotfix/<issue>`

### 4. Remote Backups

```bash
# Push to DoltHub or self-hosted remote
dolt remote add origin dolthub.com/username/project
dolt push -u origin main
# Agents can push work branches too
dolt push origin feature-branch
```

### 5. Monitoring & Alerting

```bash
# Check for stuck tasks
bd show --status blocked --json | jq -r '.[] | "\(.id) blocked by: \(.blocked_by)"'

# Find long-running tasks
dolt sql -q "
SELECT id, title, updated_at
FROM tasks
WHERE status = 'in_progress'
  AND updated_at < NOW() - INTERVAL 1 DAY;
"
```

---

## Advanced Patterns

### MCP Integration

Beads provides MCP (Model Context Protocol) server for AI tools:

```bash
# Start MCP server
bd daemon start --mcp

# Tools exposed:
# - beads_create_task
# - beads_update_task
# - beads_get_task
# - beads_list_ready
# - beads_search_tasks
# - beads_get_dependencies
```

Agents (Claude Desktop, Cursor, etc.) can discover and use these tools natively.

### Compaction Strategies

**Time-based**: `bd compact --older-than 30d`
**Count-based**: `bd compact --keep-count 100 --per-epic`
**Manual**: Review summaries before deletion

Recommendation: Compaction runs in daemon (`--auto-compact=7d`) but keep summaries with links to originals.

### Federation Topologies

**Hub-and-Spoke**: Central planning DoltDB; contributors push via `bd sync`
**Mesh**: Peer-to-peer via Dolt remotes; each repo has full task graph
**Hybrid**: Local + remote; agents work locally, sync periodically

---

## Comparison Table

| Feature | Beads | Dolt | Notes |
|---------|-------|------|-------|
| Storage layer | Uses Dolt | Self-contained DB | Beads requires Dolt |
| Primary use | Task graph | Versioned data | Complementary |
| Conflict handling | Hash IDs prevent collision | Row-level merge conflicts | Different strategies |
| Query language | CLI/JSON | SQL | Beads adds abstraction |
| Branches | Inherited from Dolt | Native | Dolt does the branching |
| Audit trail | Task updates in Dolt | Commit history + cell lineage | Dolt provides foundation |
| Scalability | Thousands of tasks | Millions of rows | Dolt handles scale |
| Learning curve | Low (simple commands) | Medium (SQL + Git concepts) | Start with Beads CLI |

---

## Getting Started Checklist

1. **Install Dolt**: `curl -fsSL https://raw.githubusercontent.com/dolthub/dolt/main/install.sh | sudo bash`
2. **Install Beads**: `go install github.com/steveyegge/beads/cmd/bd@latest`
3. **Configure**: `dolt config --global user.email "agent@org"; dolt config --global user.name "Agent"`
4. **Initialize Project**: `cd project && bd init`
5. **Create First Task**: `bd create "First task" -p 0`
6. **Explore**: `bd ready`, `bd show <id>`, `dolt sql -q "SELECT * FROM tasks"`
7. **Branch and Experiment**: `dolt checkout -b experiment/test-approach`
8. **Merge and Deploy**: `dolt checkout main && dolt merge experiment/test-approach`

---

## Conclusion

Beads + Dolt provide a complete solution for persistent, collaborative, long-horizon agent workflows. Dolt's versioned SQL storage gives you Git-like operations on structured data, while Beads adds the task graph, dependencies, and agent-friendly interface on top.

The combination enables:
- **Reliability**: Crash recovery, rollback, branching
- **Collaboration**: Multi-agent, conflict prevention, merging
- **Auditability**: Full lineage, cell history, provenance
- **Scalability**: Handles thousands of tasks, millions of data rows
- **Usability**: Simple CLI, JSON output, MCP integration

For implementing sophisticated agent systems that need to remember, collaborate, and iteratively improve over long time horizons, Beads + Dolt is currently the most robust open-source solution available.
