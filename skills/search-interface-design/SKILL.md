---
name: search-interface-design
description: >
  Design search tools and context-building strategies for AI agents. Covers the
  evolution from RAG to agent-driven search, grep/glob patterns, and how to let
  agents build their own context.
license: MIT
allowed-tools: Read Edit Grep Glob
metadata:
  author: lyzr
  version: "1.0.0"
  category: agent-architecture
---

# Search Interface Design — Letting Agents Build Their Own Context

## When to Use
When a developer is choosing how their agent finds and gathers information — whether
through RAG, search tools, or a combination.

## The Evolution in Claude Code

### Phase 1: RAG (Vector Database)
Claude Code initially used a RAG vector database to find context.

**Pros:** Fast retrieval, semantic matching.
**Cons:**
- Required indexing and setup
- Fragile across different environments
- Context was given TO Claude, not found BY Claude
- No ability to refine or deepen the search

### Phase 2: Grep-Based Search (Current)
Replaced RAG with Grep and Glob tools that let Claude search the codebase itself.

**Key insight:** As models get smarter, they get increasingly good at building their
own context IF given the right tools.

**Why it works:**
- No indexing or setup required
- Works in any environment with a filesystem
- Agent can iteratively refine searches
- Agent understands what it found and can search deeper
- Agent builds a mental model of the codebase through exploration

### Phase 3: Nested Search (Skills + Search)
Skills reference files, which reference other files. The agent follows the chain,
using search tools when references aren't enough.

**Result:** Over a year, Claude went from not being able to build its own context
to doing nested search across several layers of files to find exactly what it needed.

## Designing Search Tools for Your Agent

### The Core Search Toolkit

**1. File Search (Glob)**
```yaml
name: find-files
description: >
  Find files by name pattern. Use glob syntax. Returns file paths sorted
  by modification time. Use this to discover what files exist before reading.
input_schema:
  properties:
    pattern:
      type: string
      description: "Glob pattern (e.g., 'src/**/*.ts', '**/*test*')"
    path:
      type: string
      description: "Directory to search in (default: project root)"
```

**2. Content Search (Grep)**
```yaml
name: search-content
description: >
  Search file contents using regex. Use this to find specific code patterns,
  function definitions, imports, or error messages.
input_schema:
  properties:
    pattern:
      type: string
      description: "Regex pattern to match"
    file_glob:
      type: string
      description: "Optional filter (e.g., '*.py')"
    context_lines:
      type: integer
      description: "Lines of context around each match (default: 2)"
```

**3. File Reader**
```yaml
name: read-file
description: >
  Read the contents of a file. Use after finding files with search tools.
  Can read specific line ranges for large files.
input_schema:
  properties:
    path:
      type: string
    offset:
      type: integer
      description: "Start line (optional)"
    limit:
      type: integer
      description: "Max lines to read (optional)"
```

### Design Principles

**1. Let the agent drive**
Don't pre-fetch context for the agent. Give it search tools and let it decide what
to look for. Smarter models are better at knowing what they need.

**2. Support iterative refinement**
The agent should be able to:
- Search broadly, then narrow down
- Read a file, then search for related files
- Follow imports and references

**3. Limit result size**
Don't return 500 grep matches. Cap results and let the agent refine:
```yaml
max_results:
  type: integer
  default: 20
  maximum: 100
  description: "Limit results. If too many matches, narrow your pattern."
```

**4. Prefer dedicated tools over bash**
`Grep` tool > `bash -c "grep -r pattern ."` because:
- Structured output the model can parse
- Built-in safety (no accidental `rm -rf`)
- Better UX for the user reviewing tool calls
- Consistent behavior across environments

**5. Include file metadata**
When returning search results, include enough context for the agent to decide
what to read next: file path, line number, surrounding lines.

## RAG vs. Agent Search: When to Use Each

| Factor | RAG | Agent Search |
|--------|-----|-------------|
| Setup cost | High (indexing, embeddings) | Low (grep works anywhere) |
| Semantic matching | Strong | Weak (regex only) |
| Agent understanding | Low (context is given) | High (agent builds its own) |
| Iterative refinement | Hard | Natural |
| Environment portability | Low | High |
| Works with code | Decent | Excellent |
| Works with prose/docs | Excellent | Decent |

**Recommendation:** Start with agent-driven search (grep/glob). Add RAG only if the
agent consistently fails to find what it needs through search — typically for large
prose knowledge bases where semantic matching matters.

## Composing Search with Skills

A powerful pattern: skills that ADD search capabilities.

```markdown
# Skill: database-explorer

## Instructions
When the user asks about database schema or data:

1. Read `knowledge/schema.sql` for the current schema
2. Use the `run-query` tool to inspect actual data:
   - `SELECT * FROM information_schema.tables`
   - `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '...'`
3. Never run destructive queries (DELETE, DROP, TRUNCATE)
```

This skill doesn't add a tool — it teaches the agent how to use existing tools
for a new domain. Progressive disclosure in action.
