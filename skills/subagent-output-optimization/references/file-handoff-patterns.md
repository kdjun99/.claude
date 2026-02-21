# File Handoff Patterns — Inter-Subagent Communication

## The Problem

Subagents cannot call other subagents (no nesting). The main context orchestrates all subagent calls sequentially. If subagent B needs subagent A's detailed output, there are two ways:

### Bad: Through Main Context

```
Main Context
├── Call Agent A → receives full 500-line analysis → stored in main context
├── Call Agent B with "here's what A found: [500 lines pasted]" → duplicated
└── Main context now has 1000+ lines of agent output
```

### Good: Through File System

```
Main Context
├── Call Agent A → A saves to analysis.md → returns 5-line summary
├── Call Agent B with "read analysis.md for A's output" → B reads file directly
└── Main context only has 10 lines of summaries total
```

## File Handoff Rules

### Rule 1: Command Passes File Paths, Not Content

The orchestrating command should tell the next subagent **where to read**, not **what was found**.

```
# Bad — command reads file and passes content to agent
agent_a_result = call_agent_a()  # returns summary
file_content = read("analysis.md")  # command reads full file
call_agent_b(context=file_content)  # passes full content → bloats prompt

# Good — command passes file path to agent
agent_a_result = call_agent_a()  # returns summary + file path
call_agent_b(read_from="analysis.md")  # agent reads it in own context
```

### Rule 2: Subagents Declare Their Output Path

Every subagent should clearly state where its detailed output is saved:

```markdown
## Output
Save result to the file path specified by the calling command.
```

The calling command specifies the path:
```
Create analysis for component X.
Save to: ~/.claude/workspace/pipelines/my-pipeline/details/component-x_draft.md
```

### Rule 3: Predecessor Chain via File References

When a subagent needs context from multiple predecessors, pass paths as a list:

```
Design component: review-agent

Read these for context:
- Pipeline design: ~/.claude/workspace/pipelines/review/pipeline_final.md
- Build report: ~/.claude/workspace/pipelines/review/build_report.md
- Predecessor details:
  - ~/.claude/workspace/pipelines/review/details/review-patterns.md
  - ~/.claude/workspace/pipelines/review/details/code-analyzer.md

Save draft to: ~/.claude/workspace/pipelines/review/details/review-agent_draft.md
```

The subagent reads each file in its own context window — none of this content passes through main.

---

## Naming Conventions for Handoff Files

| Prefix/Suffix | Meaning | Lifecycle |
|--------------|---------|-----------|
| `_` prefix | Internal intermediate artifact | Consumed by next agent, not directly by human |
| No prefix | External artifact | Input for next pipeline step or human review |
| `_draft` suffix | Awaiting human feedback | Becomes final version after review |

Examples:
```
_process-analysis.md     → Internal, consumed by pipeline-architect
pipeline_draft.md        → External, reviewed by human
component_draft.md       → Awaiting human feedback
component.md             → Final version after feedback applied
```

---

## Workspace as Communication Bus

Think of the workspace directory as a message bus between subagents:

```
~/.claude/workspace/pipelines/{id}/
│
│  Agent A writes here ──►  _analysis.md
│                                │
│  Agent B reads ◄───────────────┘
│  Agent B writes here ──►  _design.md
│                                │
│  Agent C reads ◄───────────────┘
│  Agent C writes here ──►  build_report.md
│
│  Human reads/writes ──►  discussion.md
│
└── details/
    │  Agent D writes ──►  component_draft.md
    │  Human reviews ──►  (feedback via command args)
    │  Agent E reads draft + feedback
    │  Agent E writes ──►  component.md (final)
```

Each agent only sees what it needs. The main context only sees summaries. The file system carries the full detail.

---

## Anti-Patterns

### 1. Echo Anti-Pattern

```
# Main reads what agent already saved — wasteful
result = call_agent_a()          # A saves to file + returns summary
content = read("analysis.md")    # Main reads the full file
display(content)                 # Dumps full content into main context
```

Fix: Trust the summary. Only read the file if the main context needs to make a decision based on specific details.

### 2. Relay Anti-Pattern

```
# Main relays full content between agents
a_output = call_agent_a()       # Returns full analysis
call_agent_b(input=a_output)    # Passes full analysis as input
```

Fix: Agent A saves to file, main tells Agent B the file path.

### 3. Accumulation Anti-Pattern

```
# Running 10 agents, each returning verbose results
for component in components:
    result = call_detail_agent(component)  # Each returns 50+ lines
    all_results.append(result)              # 500+ lines accumulated
```

Fix: Each agent returns 3-5 line summary. Detailed results are in files. Main tracks progress with a status table, not full results.
