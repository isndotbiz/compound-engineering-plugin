# Orchestrator Integration — Cheap Worker Alternative

When an orchestrator MCP server is available (`mcp__orchestrator__*` tools), use it as a **cost-effective alternative** to spawning Claude subagents for tasks that don't require tool access.

## Available Orchestrator Tools

| Tool | Purpose | Replaces |
|------|---------|----------|
| `mcp__orchestrator__orchestrator_dispatch` | Single worker dispatch | 1 Claude Agent subagent |
| `mcp__orchestrator__orchestrator_swarm` | Fan-out to ALL workers | Multiple Claude Agent subagents |
| `mcp__orchestrator__orchestrator_health` | Check worker availability | N/A |
| `mcp__orchestrator__orchestrator_spend` | Track spend | N/A |

## When to Use Orchestrator Instead of Claude Subagents

### Use Orchestrator For:
- **Research** — `dispatch` with `task_type: "research"` instead of `best-practices-researcher` agent
- **Code review** — `swarm` instead of 3+ review agents (`security-sentinel`, `performance-oracle`, etc.)
- **Brainstorming** — `swarm` instead of spawning multiple `general-purpose` agents
- **Summarization** — `dispatch` with `task_type: "generation"` instead of a dedicated agent
- **Validation** — `dispatch` with `task_type: "validation"` instead of spawning a validator

### Still Use Claude Subagents For:
- Tasks requiring **tool access** (Edit, Write, Bash, Git)
- Tasks requiring **conversation context** that can't be serialized
- Tasks requiring **iterative refinement** with tool feedback loops
- **Design agents** (figma-design-sync, design-iterator) that need screenshots

## Hybrid Pattern

Combine orchestrator for research/review with Claude subagents for implementation:

```
1. TaskCreate: "Research best practices for X" (Task A)
2. TaskCreate: "Implement X" (Task B, blockedBy: [A])

3. orchestrator_dispatch(prompt: "Research X", task_type: "research")
4. TaskUpdate A: completed

5. Agent(subagent_type: "general-purpose", prompt: "Implement X using: <research results>")
6. TaskUpdate B: completed
```

## Parallel Review Pattern (Replaces 3+ Review Agents)

Instead of:
```
Agent(subagent_type: "security-sentinel", ...)
Agent(subagent_type: "performance-oracle", ...)
Agent(subagent_type: "code-simplicity-reviewer", ...)
```

Use:
```
orchestrator_swarm(prompt: "Review this code for security, performance, and simplicity: <code>")
```

All 14 workers review in parallel. Cost: ~$0 vs ~$5-10 for 3 Claude agents.

## Task Type Mapping

| Compound Agent | Orchestrator Equivalent |
|---------------|------------------------|
| `best-practices-researcher` | `dispatch(task_type: "research")` |
| `framework-docs-researcher` | `dispatch(task_type: "research")` |
| `security-sentinel` | `swarm` or `dispatch(task_type: "review")` |
| `performance-oracle` | `swarm` or `dispatch(task_type: "review")` |
| `code-simplicity-reviewer` | `dispatch(task_type: "review")` |
| `architecture-strategist` | `dispatch(task_type: "review")` |
| `pattern-recognition-specialist` | `dispatch(task_type: "review")` |

## Detection

Check if orchestrator is available before deciding dispatch strategy:

1. Try `mcp__orchestrator__orchestrator_health` — if it returns data, orchestrator is available
2. If available, prefer orchestrator for research/review/brainstorm tasks
3. If unavailable, fall back to standard Claude subagent patterns
