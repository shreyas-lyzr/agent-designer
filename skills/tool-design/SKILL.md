---
name: tool-design
description: >
  Design the right action space for an AI agent. Helps choose between bash-only,
  specialized tools, and hybrid approaches. Uses the "math problem" framework —
  match tools to the model's abilities.
license: MIT
allowed-tools: Read Edit Grep Glob
metadata:
  author: lyzr
  version: "1.0.0"
  category: agent-architecture
---

# Tool Design — Shaping the Action Space

## When to Use
When a developer is deciding what tools to give their agent, how to structure tool
schemas, or whether to add/remove/merge tools.

## The Math Problem Framework

Imagine giving someone a hard math problem. The tools they need depend on their skills:

| Tool Level | Analogy | Agent Equivalent |
|-----------|---------|-----------------|
| Paper | Minimum viable | Plain text output only |
| Calculator | Targeted power | A few specialized tools (Read, Edit, Grep) |
| Computer | Maximum power | Bash / code execution (but must know how to use it) |

**Key insight:** You want tools shaped to the model's abilities. A model that's great
at writing code benefits from bash/code execution. A model that struggles with complex
shell commands benefits from dedicated tools with clear schemas.

## Design Principles

### 1. Fewer Tools, Better Tools
Claude Code has ~20 tools and the bar to add a new one is high. Every tool is one more
decision for the model. Before adding a tool, ask:
- Can an existing tool handle this?
- Can progressive disclosure handle this without a new tool?
- Will the model understand when and how to call this?

### 2. One Tool, One Responsibility
The Claude Code team tried adding questions to the ExitPlanTool. It confused the model
because it mixed two concerns: approving a plan and asking questions. They split it into
ExitPlanTool + AskUserQuestion.

**Anti-pattern:**
```yaml
name: plan-and-ask
description: Submit a plan and optionally ask questions
input_schema:
  properties:
    plan: { type: string }
    questions: { type: array }  # Confusing — is this optional? Required?
```

**Better:**
```yaml
# Tool 1: ExitPlanMode
name: exit-plan-mode
description: Submit the completed plan for user approval

# Tool 2: AskUserQuestion (separate tool)
name: ask-user-question
description: Ask the user clarifying questions with multiple-choice options
input_schema:
  properties:
    questions:
      type: array
      items:
        properties:
          question: { type: string }
          options: { type: array }
```

### 3. Test with the Model, Not Your Intuition
The best-designed tool doesn't work if the model doesn't understand how to call it.
Always:
1. Write the tool schema
2. Run the agent and read its outputs
3. Check: Does it call the tool at the right time? With the right parameters?
4. Iterate based on what you observe

### 4. Structured Output > Format Instructions
The Claude Code team tried getting Claude to output questions in a special markdown
format. It worked sometimes but was unreliable — Claude would append extra text or
use a different format. A tool with a schema guarantees structure.

## Designing a Tool Schema

Good tool schemas are:
- **Descriptive** — clear name and description so the model knows when to use it
- **Constrained** — required fields, enums, and types that prevent misuse
- **Documented** — parameter descriptions that explain what good values look like

```yaml
name: search-codebase
description: >
  Search for files or code patterns in the project. Use this instead of
  bash grep when you need to find specific code. Returns file paths and
  matching lines.
input_schema:
  type: object
  properties:
    pattern:
      type: string
      description: "Regex pattern to search for (e.g., 'function\\s+handleAuth')"
    file_glob:
      type: string
      description: "Optional glob to filter files (e.g., '**/*.ts')"
    max_results:
      type: integer
      description: "Max results to return (default 20, max 100)"
      default: 20
  required:
    - pattern
```

## Checklist: Should You Add a Tool?

- [ ] Does the model struggle to do this with existing tools?
- [ ] Is this a common action (not a one-off)?
- [ ] Can the model reliably understand when and how to call it?
- [ ] Does it have a single, clear responsibility?
- [ ] Have you tested it and read the model's outputs?
