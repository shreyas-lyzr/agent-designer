# Agent Action Space Design Principles
## Distilled from "Lessons from Building Claude Code: Seeing like an Agent"

### Principle 1: See Like an Agent
Put yourself in the model's position. What tools would YOU want if you had the model's
abilities and constraints? Experiment, read outputs, iterate.

### Principle 2: Match Tools to Abilities
The "math problem" analogy — paper (text output), calculator (specialized tools),
computer (bash/code). More powerful tools require more skill to use.

### Principle 3: One Tool, One Job
The ExitPlanTool lesson — mixing plan approval and questions confused the model.
Split tools by responsibility.

### Principle 4: Structured Output > Format Instructions
Don't ask models to output custom markdown formats. Use tool schemas to guarantee
structure.

### Principle 5: The Model Must Like Calling It
Even well-designed tools fail if the model doesn't naturally understand when and how
to use them. Test with real prompts and read the outputs.

### Principle 6: Progressive Disclosure Over Prompt Stuffing
Don't put everything in the system prompt. Let agents discover context through skill
files, references, search, and subagents.

### Principle 7: Let Agents Build Their Own Context
Replace RAG with search tools. Smarter models are better at knowing what they need
and finding it themselves.

### Principle 8: Revisit Tools as Models Improve
What helped a weaker model (TodoWrite + reminders) can constrain a stronger one.
Audit your tools regularly.

### Principle 9: Fewer Tools, Better Tools
~20 tools is the Claude Code benchmark. Every new tool adds cognitive load. Consider
progressive disclosure before adding a tool.

### Principle 10: It's an Art, Not a Science
There are no rigid rules. It depends on the model, the goal, and the environment.
Experiment often, read your outputs, try new things.
