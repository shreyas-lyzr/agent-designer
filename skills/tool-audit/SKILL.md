---
name: tool-audit
description: >
  Audit and refine an agent's tool set as model capabilities evolve. Identify
  tools that are now constraining the model, tools that should be merged or
  split, and opportunities to replace tools with progressive disclosure.
license: MIT
allowed-tools: Read Edit Grep Glob
metadata:
  author: lyzr
  version: "1.0.0"
  category: agent-architecture
---

# Tool Audit — Evolving Your Action Space

## When to Use
When a developer wants to review their agent's existing tools, reduce tool count,
or adapt their action space after upgrading to a newer model.

## Why Tools Need Auditing

From the Claude Code article:
> "As model capabilities increase, the tools that your models once needed might now
> be constraining them. It's important to constantly revisit previous assumptions
> on what tools are needed."

### The TodoWrite → Task Tool Story
1. **Early Claude Code:** Model needed TodoWrite to stay on track. System reminders
   every 5 turns reminded it of the list.
2. **Improved models:** No longer needed reminders. The reminders actually made Claude
   think it had to stick rigidly to the list instead of adapting.
3. **Opus 4.5:** Got much better at using subagents. But how could subagents share a
   todo list?
4. **Solution:** Replaced TodoWrite with Task Tool — focused on inter-agent communication
   instead of self-reminding. Tasks support dependencies, shared updates, and
   modification/deletion.

**Lesson:** A tool designed for a weaker model can actively harm a stronger one.

## The Audit Framework

### Step 1: Inventory Your Tools
List every tool your agent has access to. For each, note:
- How often is it called? (check logs)
- What does the model use it for?
- Are there common misuse patterns?

### Step 2: Check for These Signals

**Signal: Tool is rarely called**
- Maybe the model doesn't understand it → improve description
- Maybe it's genuinely not needed → consider removing
- Maybe something else covers it → merge

**Signal: Tool is called incorrectly**
- Schema might be confusing → simplify parameters
- Description might be unclear → rewrite with examples
- Responsibility might be mixed → split into focused tools

**Signal: Model works around the tool**
- Uses bash instead of a dedicated tool → the dedicated tool might be worse
- Outputs data in tool calls that should go elsewhere → tool design issue
- Asks the user things it should figure out itself → missing search tools

**Signal: Model is constrained by the tool**
- Sticks to rigid patterns even when flexibility would be better (TodoWrite)
- Can't share state with subagents (TodoWrite)
- Tool assumes capabilities the model has outgrown

### Step 3: Apply These Fixes

| Issue | Fix |
|-------|-----|
| Too many tools | Merge related tools, replace with progressive disclosure |
| Tool rarely used | Remove or improve its description |
| Tool often misused | Simplify schema, add better description, or split |
| Tool constrains model | Redesign for current capabilities |
| Missing capability | Add tool only if progressive disclosure can't handle it |

### Step 4: Test and Compare
1. Run the same prompts with old and new tool sets
2. Read the outputs carefully — what changed?
3. Check: Does the model complete tasks faster? With fewer errors? With less confusion?

## Audit Checklist

For each tool in your agent's action space:

```
[ ] Is this tool called at least 5% of the time? If not, why?
[ ] Can the model reliably explain when to use this tool?
[ ] Does the tool have a single, clear responsibility?
[ ] Is the tool's schema as simple as possible?
[ ] Does the tool still match the current model's capabilities?
[ ] Could progressive disclosure replace this tool?
[ ] Does this tool interact well with other tools?
[ ] Has the tool's description been updated for the current model?
```

## When to Remove a Tool

Remove a tool when:
- The model can do the same thing with bash/code execution reliably
- A skill file can teach the model to achieve the same result without a tool
- Two tools have overlapping responsibilities and can be merged
- The tool was designed for a weaker model and now constrains behavior

**When removing, always check for regressions.** Run your test prompts and verify
the model can still accomplish the tasks the removed tool handled.

## The 20-Tool Rule of Thumb

Claude Code has ~20 tools. This is not a hard limit but a useful benchmark:
- Under 10: Might be missing important capabilities
- 10-20: Sweet spot for most agents
- 20-30: Getting heavy — audit for overlap and progressive disclosure opportunities
- 30+: Almost certainly too many — the model is spending too much time choosing

**Every tool you add is a decision the model has to make on every turn.**
Keep the action space focused.
