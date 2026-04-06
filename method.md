# Guided Agentic Development (GAD)

**Version:** 1.0  
**Status:** Draft  
**Author:** Human-AI Collaborative Methodology

---

## Abstract

Guided Agentic Development (GAD) is a structured methodology for building complex software through human-AI collaboration. It addresses the failure modes of unstructured AI-assisted development ("vibe coding") by introducing a disciplined cycle of planning, task decomposition, agent delegation, state persistence, and human verification.

GAD borrows from Agile's iterative delivery and Spec-Driven Development's emphasis on upfront definition, but adapts these ideas for a world where the developer's primary collaborator is an AI agent that can spawn sub-agents, create tools, and recover session state from persistent memory.

---

## When to Use GAD

GAD is designed for software projects that meet one or more of these criteria:

- **Too complex for zero-shot prompting** -- the system has interdependent components, external APIs, or nuanced requirements that no single prompt can capture.
- **High failure cost** -- mistakes compound and are expensive to unwind (e.g., data pipelines, authentication systems, infrastructure).
- **Multi-session work** -- the project spans hours or days, requiring agents to resume from where they left off.
- **Exploratory with hard constraints** -- the solution space is uncertain, but the acceptance criteria are strict.

If the work can be completed reliably in a single, short AI session, GAD is unnecessary overhead.

---

## Core Principles

| # | Principle | Description |
|---|-----------|-------------|
| 1 | **Human Authority** | The human is the final authority on task completion, direction changes, and failure declarations. The AI proposes; the human disposes. |
| 2 | **Incremental Verification** | No task is "done" until a human verifies it against defined acceptance criteria. Work is committed only after verification passes. |
| 3 | **Recoverable State** | Agents are ephemeral. State is not. Every critical milestone is persisted to a memory file so a fresh agent can resume without loss. |
| 4 | **Minimal Viable Task** | Tasks are the smallest unit of work that can be independently verified. Large tasks are split until each has a clear stopping point. |
| 5 | **Tools Over Repetition** | When an agent performs the same complex operation more than once, it should be captured as a reusable skill or MCP tool. |
| 6 | **Isolated Workstreams** | Each task operates on its own git branch, keeping in-progress work isolated from the stable main branch. |
| 7 | **Explicit Failure** | A task that cannot break through after sustained investigation (2+ hours) is declared a failure, documented, and the team moves on or pivots. |

---

## Roles

### Human (Director)

- Authors the plan
- Defines and sequences tasks
- Verifies task completion
- Declares success or failure
- Intervenes during stalls
- Provides domain knowledge the AI cannot infer

### Master Agent (Orchestrator)

- Interprets the plan and current task
- Delegates sub-tasks to specialized agents
- Manages memory persistence at milestones
- Proposes skill and tool creation when patterns emerge
- Reports progress and blockers to the human

### Sub-Agents (Specialists)

- Execute narrowly scoped work delegated by the master agent
- Operate in read-only or read-write mode as appropriate
- Return results to the master agent for integration

---

## Project Structure

A GAD project lives inside a single directory. The following is the canonical layout:

```
project-root/
├── method.md                  # This file -- methodology reference
├── plan.md                    # Human-authored project plan
├── retrospective.md           # Cumulative findings and lessons
├── skills/                    # Shared skill definitions
│   ├── skill-name/
│   │   └── SKILL.md
│   └── ...
├── tools/                     # Shared tool configurations
│   └── ...
├── references/                # Cloned reference repos (read-only source inspection)
│   └── ...
├── src/                       # Source code (structure varies by project)
│   └── ...
└── tasks/                     # Task metadata (stays on main branch)
    ├── 01-task-slug/
    │   ├── task.md            # Task definition
    │   ├── memory.md          # Agent state persistence
    │   └── skills/            # Task-specific skills (optional)
    ├── 02-task-slug/
    │   ├── task.md
    │   └── memory.md
    └── ...
```

---

## Artifacts

### plan.md

The plan is **human-written** and serves as the project's source of truth. It describes the final piece of software, the high-level approach, and the ordered sequence of tasks.

**Required sections:**

```markdown
# Project Plan: [Project Name]

## Vision
What the final software does, who it serves, and why it matters.

## Architecture
High-level technical decisions, key technologies, and system boundaries.

## Task Sequence
Ordered list of tasks with brief descriptions. Each entry points to
its task directory under `tasks/`.

1. `tasks/01-scaffold` -- Set up project structure, dependencies, CI
2. `tasks/02-data-model` -- Define and implement core data model
3. `tasks/03-api-layer` -- Build REST API endpoints
...

## Tools
Required tooling for the project and methodology:
- git (version control, branching)
- gh (GitHub CLI for PRs, issues)
- [project-specific tools]

## Skills
Shared skills available to all tasks:
- `skills/[skill-name]` -- [purpose]
```

### task.md

Each task directory contains a `task.md` that fully defines the unit of work. A task is the GAD equivalent of a sprint backlog item, scoped small enough for a single focused session.

**Required sections:**

```markdown
# Task: [Task Name]

## Objective
One-sentence description of what this task accomplishes.

## Context
What the agent needs to know before starting. References to prior tasks,
relevant documentation, or architectural decisions.

## Acceptance Criteria
Concrete, verifiable conditions that define "done":
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Verification
How the human will verify completion:
- Manual testing steps
- Automated test commands
- Visual inspection points

## Skills & Tools
Skills and tools this task requires:
- `skills/[skill-name]` -- [why]
- [MCP server or CLI tool] -- [why]

## Constraints
Boundaries, limitations, or things explicitly out of scope.
```

### memory.md

The memory file is the mechanism for **agent state persistence**. When an agent reaches a critical milestone or stopping point, it writes a snapshot of everything a fresh agent would need to resume the work without re-discovery.

**Structure:**

```markdown
# Memory: [Task Name]

## Last Updated
[ISO 8601 timestamp]

## Status
[in-progress | blocked | awaiting-verification | complete | failed]

## Current Position
Where the agent left off -- which acceptance criterion it was working on,
what step of the process it had reached.

## Key Decisions
Decisions made during the task that a new agent must respect:
- Decision 1: [what] because [why]
- Decision 2: [what] because [why]

## Working Context
Files modified, commands run, environment state, or any context that
would take significant time to re-derive:
- [file path] -- [what was done]
- [command] -- [what it produced]

## Blockers
Any unresolved issues or questions for the human.

## Next Steps
What the agent should do immediately upon resuming.
```

**When to write memory:**

| Trigger | Action |
|---------|--------|
| Acceptance criterion completed | Update status and current position |
| Significant discovery or decision | Record under Key Decisions |
| Before requesting human verification | Full snapshot of state |
| Agent session ending (timeout, context limit) | Full snapshot of state |
| After human feedback that changes direction | Record feedback, update next steps |

### retrospective.md

A cumulative, append-only document that captures findings, surprises, and lessons across the entire project. Entries are added after each task completes (success or failure) and during notable side-investigations.

**Structure:**

```markdown
# Retrospective

## Task: [task-slug] -- [date]

### Outcome
[success | failure | partial]

### What Worked
- [observation]

### What Didn't
- [observation]

### Human Interventions
- [prompt or action the human took and why]

### Discoveries
- [unexpected finding relevant to future tasks]

### Recommendations
- [suggestion for improving the process or plan]

---
(next entry appended below)
```

---

## Workflow

The GAD workflow is a cycle that repeats for each task in the plan.

```
┌─────────────────────────────────────────────────────────┐
│                      PLAN PHASE                         │
│  Human writes plan.md, defines task sequence            │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   TASK SETUP PHASE                       │
│  1. Create task branch and task metadata directory      │
│  2. Write task.md with objective and criteria           │
│  3. Identify required skills and tools                  │
│  4. Initialize memory.md                                │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                  EXECUTION PHASE                        │
│  Master agent works through acceptance criteria:        │
│  ┌───────────────────────────────────────────────┐      │
│  │  Work on criterion ──► Milestone? ──► Save    │      │
│  │       ▲                   memory              │      │
│  │       │                     │                 │      │
│  │       └─────────────────────┘                 │      │
│  └───────────────────────────────────────────────┘      │
│  Agent may:                                             │
│  - Spawn sub-agents for scoped sub-tasks                │
│  - Create skills when reusable patterns emerge          │
│  - Request human input when blocked                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│               VERIFICATION PHASE                        │
│  1. Agent reports criteria status                       │
│  2. Human tests against verification steps              │
│  3. Human declares: PASS or FAIL                        │
└──────────┬───────────────────────────┬──────────────────┘
           │                           │
        PASS                         FAIL
           │                           │
           ▼                           ▼
┌──────────────────────┐   ┌──────────────────────────────┐
│   COMMIT PHASE       │   │     FAILURE PROTOCOL         │
│  1. Commit to git    │   │  1. Document in retrospective│
│  2. Merge to main    │   │  2. Decide: retry, pivot,    │
│  3. Update retro     │   │     or skip                  │
│  4. Next task         │   │  3. Update plan if needed    │
└──────────────────────┘   └──────────────────────────────┘
```

### Phase Details

#### 1. Plan Phase (Human)

The human authors `plan.md` from scratch. This is not AI-generated -- it represents the human's intent, domain knowledge, and architectural judgment. The AI may help refine it, but the human owns it.

**Checklist:**
- [ ] Vision is clear and specific
- [ ] Architecture decisions are documented
- [ ] Tasks are ordered by dependency
- [ ] Each task is small enough for a single focused session
- [ ] Tools and skills sections are populated

#### 2. Task Setup Phase (Human + AI)

For each task in the sequence:

```bash
# Create a branch for the task
git checkout -b task/01-task-slug

# Create the task metadata directory
mkdir -p tasks/01-task-slug

# Initialize task files
# tasks/01-task-slug/task.md -- human writes or co-authors with AI
# tasks/01-task-slug/memory.md -- initialized with status: not-started
```

The human writes or reviews the `task.md`. The AI may draft it from the plan, but the human approves the acceptance criteria before work begins.

#### 3. Execution Phase (AI, Human-supervised)

The master agent reads `task.md` and `memory.md`, then works through acceptance criteria one at a time. The agent:

- **Persists state** at every milestone (see memory triggers above)
- **Delegates** narrow sub-tasks to sub-agents when appropriate
- **Creates skills** when it recognizes reusable patterns
- **Escalates** to the human when blocked or uncertain

If the agent's session ends (timeout, context window, crash), a new agent reads `memory.md` and resumes from the last recorded position.

#### 4. Verification Phase (Human)

The human tests the work against the acceptance criteria in `task.md`. This is a gate -- no work proceeds to the next task without human sign-off. 

**IMPORTANT**: Agent will ask to proceed to the next task with "Proceed to the next task?".  If the human says yes, then you will continue to the next task, otherwise, you will continue to work on the existing task.  Do not continue to the next task without this permission.

 

**Possible outcomes:**

| Outcome | Action |
|---------|--------|
| **Pass** | Proceed to commit phase |
| **Partial pass** | Agent addresses remaining criteria, re-verify |
| **Fail** | Enter failure protocol |

#### 5. Commit Phase (AI + Human)

```bash
# Commit all work on the task branch
git add -A
git commit -m "task/01-task-slug: [summary of what was accomplished]"

# Merge into main (or create PR for team projects)
git checkout main
git merge task/01-task-slug

# Delete the task branch
git branch -d task/01-task-slug
```

Update `retrospective.md` with the task outcome.

#### 6. Failure Protocol

A task enters failure when:

- **Timeout** -- 2+ hours of continuous investigation without breakthrough
- **Human declaration** -- the human tests and declares it a failure
- **Unresolvable blocker** -- a dependency or external constraint prevents completion

**Failure handling:**

1. Document the failure in `retrospective.md` with full context
2. Save final state to `memory.md` (for potential future retry)
3. Human decides:
   - **Retry** with a different approach (update task.md, restart execution)
   - **Pivot** by modifying the plan to work around the blocker
   - **Skip** if the task is non-critical

---

## Skills System

Skills are packages of knowledge that extend an agent's capabilities. They follow the Cursor SKILL.md format and can be shared across tasks or scoped to a single task.

### Skill Types

| Type | Location | Purpose |
|------|----------|---------|
| **Project skill** | `skills/[name]/SKILL.md` | Shared across all tasks |
| **Task skill** | `tasks/[slug]/skills/[name]/SKILL.md` | Scoped to a single task |

### What Goes in a Skill

Skills package knowledge that an agent would not have by default:

- **Technical reference** -- API details, library usage patterns, domain-specific conventions
- **Source acquisition** -- Instructions to clone, fetch, or cache external source code from repositories
- **Remote content retrieval** -- Techniques to fetch documentation, HTML, or markdown from external sources
- **Tool creation** -- Instructions to build an MCP server that provides a reusable capability to any agent

### When to Create a Skill

Create a skill when:

1. An agent performs the same complex lookup or operation more than once
2. A task requires specialized domain knowledge the base model lacks
3. External content (docs, source code, API specs) needs to be reliably accessed
4. A reusable tool (MCP server) would benefit multiple tasks

### Skill Lifecycle

```
Identify need ──► Draft SKILL.md ──► Test with agent ──► Refine ──► Commit
```

If a skill is created during a task, the agent records it in `memory.md` and references it in the task's `retrospective.md` entry.

---

## Tools

GAD requires a minimal, standard toolset.

### Required

| Tool | Purpose |
|------|---------|
| `git` | Version control, branching |
| `gh` | GitHub CLI for pull requests, issues, and repository operations |

### Setup Verification

Before starting a GAD project, verify the toolchain:

```bash
git --version          # Any modern version
gh --version           # GitHub CLI
gh auth status         # Confirm authenticated
```

### Git Branch Cheat Sheet

```bash
# Create a branch for a task
git checkout -b task/01-task-slug

# List branches
git branch

# Switch back to main
git checkout main

# Merge a completed task
git merge task/01-task-slug

# Delete the task branch after merge
git branch -d task/01-task-slug
```

---

## Memory Persistence Deep Dive

Memory persistence is what separates GAD from ad-hoc AI-assisted development. Without it, every new agent session starts from zero -- re-reading files, re-discovering decisions, and re-making mistakes.

### What to Capture

| Category | Examples |
|----------|----------|
| **Position** | "Working on criterion 3 of 5", "Debugging the auth middleware" |
| **Decisions** | "Chose JWT over sessions because...", "Using PostgreSQL array type for tags" |
| **Context** | Modified files, test results, error messages, environment variables |
| **Dead ends** | Approaches that were tried and failed, with reasons |
| **Blockers** | Unresolved questions, missing access, unclear requirements |
| **Next steps** | Exactly what to do next, in priority order |

### What NOT to Capture

- Entire file contents (reference by path instead)
- Verbose logs (summarize the relevant finding)
- Generic knowledge the base model already has

### Recovery Protocol

When a fresh agent starts on a task with an existing `memory.md`:

1. Read `task.md` for objective and criteria
2. Read `memory.md` for current state
3. Verify the recorded state matches the actual file system
4. Resume from the documented next steps

---

## Quick-Start Checklist

Start a new GAD project with this sequence:

```
1. [ ] Create project directory
2. [ ] Initialize git repository
3. [ ] Write plan.md (human)
4. [ ] Create skills/ and tasks/ directories
5. [ ] Commit initial project structure to main
6. [ ] Create first task branch (git checkout -b task/01-...)
7. [ ] Write first task.md (human + AI)
8. [ ] Initialize first memory.md
9. [ ] Begin execution phase
```

---

## Glossary

| Term | Definition |
|------|------------|
| **Plan** | The human-authored document describing the target software and task sequence |
| **Task** | The smallest independently verifiable unit of work, executed on its own git branch |
| **Memory** | A persistent state file that allows agents to resume work across sessions |
| **Skill** | A markdown package of specialized knowledge or instructions for agents |
| **Retrospective** | A cumulative log of outcomes, discoveries, and lessons across tasks |
| **Master Agent** | The primary AI agent that interprets tasks and orchestrates work |
| **Sub-Agent** | A specialized AI agent spawned by the master agent for narrow sub-tasks |
| **Verification** | Human testing of completed work against defined acceptance criteria |
| **Failure Protocol** | The structured process for handling tasks that cannot be completed |
| **Task Branch** | A git branch (e.g., `task/01-scaffold`) isolating a task's work from main |

---

## Appendix A: Example Plan

```markdown
# Project Plan: MapLibre LMV Viewer

## Vision
A web-based viewer that renders Autodesk LMV models on a MapLibre GL
basemap, enabling geospatial context for BIM models.

## Architecture
- Frontend: Vanilla JS + MapLibre GL JS + LMV Viewer
- Build: Vite
- Backend: Node.js proxy for Autodesk APS token exchange
- Deployment: Vercel (frontend) + Railway (backend)

## Task Sequence
1. `tasks/01-scaffold` -- Initialize monorepo, Vite config, MapLibre hello-world
2. `tasks/02-aps-auth` -- APS OAuth2 token exchange proxy
3. `tasks/03-lmv-overlay` -- Load LMV viewer as a MapLibre custom layer
4. `tasks/04-geolocation` -- Position model at real-world coordinates
5. `tasks/05-interaction` -- Click-to-select, property panel, camera sync

## Tools
- git, gh (methodology)
- node, npm (runtime)
- vite (build)

## Skills
- `skills/aps-auth` -- Autodesk Platform Services authentication patterns
- `skills/maplibre-custom-layers` -- MapLibre GL JS custom layer API reference
```

## Appendix B: Example Task

```markdown
# Task: 02-aps-auth

## Objective
Build a Node.js proxy that exchanges APS client credentials for a
2-legged access token and serves it to the frontend.

## Context
Task 01 established the project scaffold with Vite and MapLibre.
The APS viewer requires an access token to load models. Tokens must
not be generated client-side (client secret exposure).

## Acceptance Criteria
- [ ] POST /api/token returns a valid APS access token
- [ ] Token response includes expires_in for client-side caching
- [ ] Client secret is loaded from environment variables, never hardcoded
- [ ] Error responses return structured JSON with status codes
- [ ] Integration test covers happy path and invalid credentials

## Verification
1. Start the proxy: `node server.js`
2. `curl -X POST http://localhost:3000/api/token`
3. Confirm response contains `access_token` and `expires_in`
4. Confirm `.env` contains credentials and is in `.gitignore`

## Skills & Tools
- `skills/aps-auth` -- APS 2-legged OAuth2 flow reference
- express -- HTTP server
- node-fetch -- APS API calls

## Constraints
- No frontend changes in this task
- Token endpoint only; model loading is task 03
```

## Appendix C: Example Memory

```markdown
# Memory: 02-aps-auth

## Last Updated
2026-03-24T18:45:00Z

## Status
in-progress

## Current Position
Criteria 1-3 complete. Working on criterion 4 (error handling).
The happy path works. Currently implementing structured error
responses for expired/invalid credentials.

## Key Decisions
- Using express over fastify for simplicity and team familiarity
- Token cached in-memory with TTL based on expires_in minus 60s buffer
- Error responses follow { error: string, status: number } shape

## Working Context
- `server.js` -- main proxy, 45 lines, token route implemented
- `middleware/errorHandler.js` -- in progress
- `.env` -- APS_CLIENT_ID and APS_CLIENT_SECRET configured
- Tests passing: 2/3 (error handling test not yet written)

## Blockers
None.

## Next Steps
1. Finish error handler middleware in middleware/errorHandler.js
2. Write integration test for invalid credentials scenario
3. Request human verification
```
