---
name: progressive-disclosure
description: >
  Design layered context strategies for AI agents. Avoid prompt stuffing by letting
  agents discover information incrementally through skill files, recursive references,
  subagents, and search tools.
license: MIT
allowed-tools: Read Edit Grep Glob
metadata:
  author: lyzr
  version: "1.0.0"
  category: agent-architecture
---

# Progressive Disclosure — Layered Context Discovery

## When to Use
When a developer needs their agent to access a large body of knowledge without stuffing
it all into the system prompt. Also when deciding between adding a tool vs. adding
context that the agent can discover.

## The Problem
Agents often need domain knowledge, but putting everything in the system prompt causes:
- **Context rot** — irrelevant information dilutes focus on the actual task
- **Interference** — reference docs make the model drift from its primary job
- **Wasted tokens** — most of the context goes unused in any given conversation

## The Progressive Disclosure Pattern

Instead of front-loading context, create layers that the agent discovers on demand:

```
Layer 0: System prompt (identity, core instructions, tool list)
    |
Layer 1: Skill files (loaded when skill is invoked)
    |
Layer 2: References within skills (files the skill points to)
    |
Layer 3: Search results (agent uses grep/glob to find specifics)
    |
Layer 4: Subagent results (delegated deep research)
```

**Each layer is only loaded when needed.** The agent starts lean and adds context
as the conversation requires it.

## Claude Code Examples

### Example 1: The Claude Code Guide
**Problem:** Users asked Claude Code about its own features (MCP, slash commands, etc.)
but Claude didn't know the answers.

**Rejected approach:** Put all docs in the system prompt.
- Would add thousands of tokens of context rot
- Users rarely ask these questions
- Would interfere with Claude Code's main job: writing code

**Progressive disclosure approach:**
1. Give Claude a link to its docs (Layer 1)
2. Claude loads docs when asked about itself (Layer 2)
3. But Claude loaded too much — so they built a Guide subagent (Layer 4)
4. The subagent has specialized instructions for searching docs efficiently

**Result:** Claude can answer self-referential questions without polluting its
main context.

### Example 2: Agent Skills
Skills formalized progressive disclosure:
1. Agent reads `SKILL.md` (Layer 1)
2. Skill file references other files: `See knowledge/api-reference.md` (Layer 2)
3. Agent reads those files and follows more references recursively (Layer 2+)
4. Agent uses search tools if it still needs more context (Layer 3)

**A common use of skills is to add search capabilities** — instructions on how to
query a database, call an API, or navigate a specific codebase structure.

## Designing Your Disclosure Layers

### Layer 0: System Prompt
Keep it focused:
- Agent identity (SOUL.md)
- Hard rules (RULES.md)
- Tool definitions
- 1-2 sentence description of each skill

**Do NOT put here:** reference docs, API schemas, lengthy examples, rarely-used
instructions.

### Layer 1: Skill Files
Each skill's `SKILL.md` contains:
- When to use this skill
- Core instructions (under 5000 tokens)
- References to deeper documents

```markdown
# My Skill

## When to Use
When the user asks about X.

## Instructions
Do A, then B, then C.

## References
- For API details, read `knowledge/api-spec.md`
- For examples, read `knowledge/examples/`
```

### Layer 2: Referenced Documents
Files that skills point to. The agent reads them only when following a reference.

Structure as `knowledge/` with an index:
```yaml
# knowledge/index.yaml
documents:
  - path: core-concepts.md
    always_load: true      # Layer 0 — keep very small
  - path: api-reference.md
    always_load: false     # Layer 2 — loaded on demand
  - path: troubleshooting.md
    always_load: false
```

### Layer 3: Search
Give the agent tools to search when references aren't enough:
- Grep for code patterns
- Glob for file discovery
- Web search for external docs

### Layer 4: Subagents
For deep research that would pollute the main context:
- Spawn a subagent with specialized search instructions
- Subagent returns a concise summary
- Main agent stays focused

## Decision Framework

```
Is this info needed in EVERY conversation?
  YES → Layer 0 (system prompt) — but keep it minimal
  NO  →
    Is it needed when a specific skill is invoked?
      YES → Layer 1 (skill file)
      NO  →
        Can the agent find it by following references?
          YES → Layer 2 (referenced docs)
          NO  →
            Can the agent search for it?
              YES → Layer 3 (search tools)
              NO  → Layer 4 (subagent with specialized instructions)
```

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|-------------|-------------|-----|
| Entire API spec in system prompt | Context rot, interference | Move to Layer 2 reference doc |
| No references in skills | Agent can't go deeper | Add "See also" links |
| Too many search results in context | Dilutes focus | Use subagent to filter |
| Flat knowledge structure | Hard to navigate | Use index.yaml + directories |
| Loading everything "just in case" | Wasted tokens, confused model | Default to always_load: false |
