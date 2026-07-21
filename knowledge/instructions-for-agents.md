# Writing Agent Instructions (AGENTS.md / copilot-instructions.md)

Instructions for AI coding agents differ from chat system prompts. Agents read instructions once per session and use them to make decisions autonomously over many tool calls. Clarity and conciseness matter more than completeness.

---

## What to Include

Include only what an agent **cannot discover on its own** in a few tool calls:

| Include | Omit |
|---------|------|
| Non-obvious folder layout | Things visible from a `ls` / `list_dir` |
| Build / test commands | Language and framework (infer from files) |
| Conventions that differ from community defaults | Standard conventions the model already knows |
| Known pitfalls in this specific codebase | Generic best practices |
| Which files are authoritative for a topic | File contents (link instead) |

---

## Structure

```markdown
# AGENTS.md

## Project Overview
One paragraph. What the project is, what problem it solves.

## Directory Layout
Short table or bullet list of non-obvious folders.

## Build & Test
Exact commands. No prose around them.

## Conventions
Bullet points only. One fact per bullet. Link to docs for detail.

## Common Pitfalls
Short, actionable. "When doing X, always Y because Z."
```

---

## Link, Don't Embed

If documentation already exists in the repo, link to it instead of copying it:

```markdown
## API Design
See [docs/api-conventions.md](docs/api-conventions.md) for REST endpoint conventions.
```

Embedding duplicates content that then goes stale. Links stay correct as long as the target file exists.

---

## Specificity Over Completeness

An agent that has 5 precise rules follows them reliably. An agent given 50 rules follows them inconsistently because they overload the context window and dilute each other.

---

## Use Imperative Verbs

Every instruction should start with a verb. This makes instructions scannable and unambiguous.

**Passive/ambiguous:** "Files are named with hyphens."  
**Imperative/clear:** "Name all files with lowercase hyphens: `my-file.md`."

---

## Scope `applyTo` Carefully

For `.instructions.md` files (VS Code / Copilot), `applyTo: "**"` means the file is injected into every single request. Use specific globs to limit context injection:

```yaml
applyTo: "**/*.test.ts"   # Only when editing test files
applyTo: "docs/**"        # Only when editing documentation
```

---

## Testing Your Instructions

After writing or updating instructions:

1. Open a new chat session (instructions are loaded at session start).
2. Ask the agent to perform a task covered by the instructions.
3. Check whether the agent followed the conventions without being reminded.
4. If it didn't, the instruction is either missing, too vague, or buried too deep — revise and retest.

---

## Example: This Repository

The [AGENTS.md](../AGENTS.md) in this repo follows these principles:
- States the project purpose in one sentence
- Lists the folder structure with a short explanation of each vendor folder
- Gives the exact file naming convention with examples
- Describes the README update pattern with a format template
- Contains no prose that doesn't guide a specific decision
