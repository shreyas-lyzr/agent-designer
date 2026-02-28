---
name: elicitation-design
description: >
  Design effective question-asking mechanisms for AI agents. Covers structured
  elicitation tools, multi-option UIs, plan-mode integration, and reducing
  friction in human-agent communication.
license: MIT
allowed-tools: Read Edit Grep Glob
metadata:
  author: lyzr
  version: "1.0.0"
  category: agent-architecture
---

# Elicitation Design — How Agents Ask Questions

## When to Use
When a developer wants their agent to ask better questions, gather user preferences
efficiently, or reduce the friction of human-agent communication.

## The Problem
Agents can ask questions in plain text, but answering them feels slow and ambiguous.
Free-form questions lead to:
- Users typing long answers when a quick selection would suffice
- Ambiguous responses that the agent misinterprets
- Back-and-forth that wastes turns

**Goal:** Lower friction and increase bandwidth between user and agent.

## The Claude Code Journey

### Attempt 1: Piggyback on Existing Tools
Added a `questions` parameter to ExitPlanTool. Result: confused the model. It couldn't
decide whether to present a plan OR ask questions, and conflicting answers broke the plan.

**Lesson:** Don't mix responsibilities in a single tool.

### Attempt 2: Custom Output Format
Asked Claude to output questions in a special markdown format (bullet points with
bracketed options). Result: worked sometimes, but Claude would add extra text, skip
options, or use a different format.

**Lesson:** Structured tool output beats format instructions for reliability.

### Attempt 3: Dedicated AskUserQuestion Tool (Winner)
Created a standalone tool that:
- Takes structured questions with typed options
- Shows a modal UI that blocks until the user answers
- Returns structured responses back to the agent

**Lesson:** The model seemed to *like* calling this tool — it used it well and its
outputs were high quality.

## Designing an Elicitation Tool

### Schema Pattern
```yaml
name: ask-user-question
description: >
  Ask the user one or more questions with predefined options. Use this when
  you need clarification before proceeding. Each question should have 2-4
  concrete options. The user can always provide a custom answer.
input_schema:
  type: object
  properties:
    questions:
      type: array
      maxItems: 4
      items:
        type: object
        properties:
          question:
            type: string
            description: "Clear, specific question ending with ?"
          options:
            type: array
            minItems: 2
            maxItems: 4
            items:
              type: object
              properties:
                label:
                  type: string
                  description: "Short option text (1-5 words)"
                description:
                  type: string
                  description: "Why this option, tradeoffs"
          multiSelect:
            type: boolean
            default: false
        required: [question, options]
  required: [questions]
```

### Key Design Decisions

**1. Limit the number of questions**
Don't let the agent ask 10 questions at once. 1-4 keeps it focused and prevents
user fatigue. If more are needed, ask in rounds.

**2. Always include an "Other" escape hatch**
Users should never feel trapped by predefined options. An implicit or explicit
"Other (type your own)" option prevents frustration.

**3. Recommend a default**
When one option is clearly better, let the agent indicate it. This speeds up
the common case while preserving user choice.

**4. Block the agent loop**
When the question tool fires, the agent should STOP and WAIT. Don't let it
continue working with assumptions — the whole point is to get real input.

**5. Context-aware timing**
Prompt the agent to ask questions:
- During plan mode (before writing code)
- When requirements are ambiguous
- When multiple valid approaches exist
- NOT for trivial decisions the agent can make itself

## Integration with Plan Mode

The most natural place for elicitation is during planning:

```
User: "Add authentication to my app"
  |
  v
Agent enters plan mode
  |
  v
Agent calls AskUserQuestion:
  "Which auth method should we use?"
  - OAuth 2.0 (Recommended) — best for web apps with social login
  - JWT tokens — good for API-first architectures
  - Session cookies — simplest for server-rendered apps
  |
  v
User picks "OAuth 2.0"
  |
  v
Agent writes plan with OAuth 2.0 specifics
  |
  v
Agent calls ExitPlanMode for approval
```

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|-------------|-------------|-----|
| Asking in plain text | Slow, ambiguous answers | Use structured tool |
| Too many questions at once | User fatigue, lower quality | Max 4, ask in rounds |
| Questions with no options | Model gets vague answers back | Always provide options |
| Mixing questions with actions | Confuses the model | Separate tools |
| Asking about obvious choices | Wastes user time | Only ask when it matters |
