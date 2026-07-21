# ****

# Tool Routing — Directing the Agent to the Right Tool

The agent has many tools: file reads, searches, terminal, browser, code edits. Ambiguous prompts cause it to pick the slowest, most expensive path. Explicit routing saves tokens and turns.

---

## The Core Principle

**You know what you want. Say it directly.**  
The agent doesn't need to decide whether to search or read — you already know which one is correct.

```
❌  "Find where the README tables are updated"
     → Model does semantic search, then grep, then reads 3 files

✅  "Read README.md lines 20–60"
     → One tool call, done
```

---

## Tool Selection Reference

| You need to… | Say… | Tool used |
|---|---|---|
| Read a known file | `"Read <path>"` | `read_file` |
| Find files by name pattern | `"Find all *.md files in Anthropic/"` | `file_search` |
| Find exact text in files | `"Grep for 'Recently Updated' in README.md"` | `grep_search` |
| Find code/concept by meaning | `"Search for where vendor tables are defined"` | `semantic_search` |
| Run a shell command | `"Run: git log --oneline -5"` | `run_in_terminal` |
| Edit a known file | `"In <path> line N, change X to Y"` | `replace_string_in_file` |
| Create a new file | `"Create <path> with content: …"` | `create_file` |

---

## Precision Routing: Search Tools

The three search tools are not interchangeable. Pick the right one:

### `grep_search` — Use when you know the exact text
```
✅  "Grep for '## Recently Updated' in README.md"
✅  "Search for 'claude-opus-4.8' across all .md files"
```
Fast, deterministic, no AI reasoning involved.

### `file_search` — Use when you know the file name pattern
```
✅  "Find all files named claude-*.md under Anthropic/"
✅  "List all README.md files in the repo"
```
Pattern match against paths only — doesn't read file contents.

### `semantic_search` — Use only when you don't know the exact text or path
```
✅  "Find where the repo explains the file naming convention"
✅  "Search for any mention of sub-folder organization rules"
```
Slowest and most expensive. Reserve for genuine unknowns.

**Rule:** If you know any part of the exact text, use `grep_search` first.

---

## Routing File Edits

### Single targeted change
```
✅  "In README.md, replace the first row of the Recently Updated table with: 
     | **Claude Opus 5** | July 21, 2026 | [link](Anthropic/claude-opus-5.md) |"
```

### Multiple independent changes in different files
```
✅  "Make both changes simultaneously:
     1. In README.md: add row to Recently Updated table
     2. In Anthropic/README.md: add row to the model table"
```
The agent can execute these in a single `multi_replace_string_in_file` call.

### Multiple changes in the same file
Give all edits at once rather than one at a time:
```
✅  "In README.md make these two changes:
     1. Add row to Recently Updated table (top)
     2. Add row to Google vendor table (line ~150)"
```

---

## Routing Terminal Commands

Be explicit about the command — don't describe it:

```
❌  "Check if there are any uncommitted files"
✅  "Run: git status"

❌  "Push the changes"
✅  "Run: git add -A ; git commit -m 'Add Claude Opus 5' ; git push"
```

If the command is potentially destructive or irreversible, state it anyway — the agent will ask for confirmation before running it.

---

## Avoiding Unnecessary Tool Calls

Every tool call costs latency and tokens. Common ways to eliminate them:

### 1. Provide the content directly
```
❌  "Read the CONTRIBUTING.md file and then apply those rules to my PR description"
✅  "Apply these contributing rules to my PR description: [paste the 3 rules]"
```
If you already have the content, don't make the agent fetch it.

### 2. Give context the agent would otherwise search for
```
❌  "Add a new entry for the Google/ folder in the table"
✅  "Add to the vendor table in README.md:
     | Google | [Gemini models](Google/) |"
```
Providing the final value directly eliminates a read-then-generate cycle.

### 3. Specify the exact location for edits
```
❌  "Add the new prompt to the Anthropic table"
✅  "In README.md, insert after the '| Claude Sonnet 5 |' row:"
```
Exact location = no search required to find the insertion point.

---

## Tool Chaining: When Sequence Matters

Some tasks require one tool's output before another can run. Make the dependency explicit:

```
✅  "Run: git diff HEAD~1 --name-only
     Then for each changed file, add a row to README.md's Recently Updated table."
```

The agent reads the terminal output, then performs the edit. The dependency is clear.

When there is **no** dependency, signal that explicitly to allow parallelism:

```
✅  "Simultaneously: read Anthropic/claude-opus-5.md AND grep README.md for 
     '## Anthropic' to find the insertion point."
```

---

## System Prompt Leverage in Tool Routing

The agent's built-in system prompt already knows about all available tools. You don't need to describe tools — just name the action:

```
❌  "Use the file reading tool to open and read the contents of…"
✅  "Read knowledge/cookbook/README.md"

❌  "Could you use a terminal command to check the git log?"
✅  "Run: git log --oneline -10"
```
