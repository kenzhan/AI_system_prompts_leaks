# Orchestration — Multi-Step Tasks, Pipelines, and Subagents

When a task spans multiple files, tools, or decision points, the structure of your prompt determines whether the agent executes efficiently or spirals into unnecessary exploration.

---

## Decomposition First

Before writing a complex prompt, break the task into stages:

1. What information does the agent need first?
2. What depends on that output?
3. What can run in parallel?
4. What is the stopping condition?

Write the prompt in that order.

---

## Pattern 1: Linear Pipeline

Each step produces output the next step consumes. State steps in sequence with explicit handoffs.

```
Stage 1 — gather: "Run: git diff HEAD~1 --stat"
Stage 2 — decide: "For each .md file in the diff, determine if it's a new system prompt or an edit to an existing one."
Stage 3 — act:    "Add a row to README.md's Recently Updated table for each new prompt. Skip edits to existing ones."
```

**In one prompt:**
```
Run: git diff HEAD~1 --stat
For each .md file that appears as 'new file', add a row to README.md's Recently Updated table:
- Column 1: bold product name derived from the filename
- Column 2: today's date (July 21, 2026)
- Column 3: markdown link to the file
Skip any files that are modifications of existing files.
```

Why this works: the agent runs one terminal command, gets the list, then makes a targeted edit. No exploration, no guessing.

---

## Pattern 2: Parallel Fan-Out

Independent tasks that can execute simultaneously. Group them explicitly:

```
✅  "Simultaneously:
     1. Read Anthropic/claude-opus-5.md — extract the first sentence that names the model
     2. Grep README.md for '| Claude Opus' — find the last row in the Anthropic table
     3. Grep README.md for '## Recently Updated' — find the insertion row for the top table"
```

Three tool calls at once → single latency hit instead of three sequential ones.

**Rule:** Use "Simultaneously:" or "In parallel:" to signal the agent can fan out.

---

## Pattern 3: Conditional Branch

When the next action depends on what the agent finds, state both branches upfront:

```
✅  "Check if Perplexity/deep-research.md exists.
     - If yes: read it and verify the first line contains 'Perplexity'
     - If no:  create it with this content: [content]"
```

Without explicit branches, the agent may stop to ask which path to take.

---

## Pattern 4: Collect → Transform → Write

Common for bulk operations: gather a set of items, apply a transformation, write results.

```
✅  "Collect: file_search for all *.md files under xAI/
     Transform: for each file, produce a table row: | filename (no extension) | [link](path) |
     Write: insert all rows into the xAI vendor section of README.md, replacing the existing rows"
```

Separating the three stages in the prompt prevents the agent from writing partial results after each file.

---

## Pattern 5: Checkpoint + Continue

For long multi-file tasks, tell the agent to report after a milestone before continuing:

```
✅  "Step 1: find all vendor folders that have no entry in README.md's vendor section.
     Report the list. Wait for confirmation before step 2."
```

This is useful when:
- The intermediate result needs your approval
- The task is destructive or hard to reverse
- You want to verify correctness before bulk writes

---

## Pattern 6: Subagent for Context Isolation

Use a subagent (explore_subagent or runSubagent) when:
- You need to research a large portion of the codebase without polluting the main context
- The research result is a single structured output (a list, a table, a decision)
- You don't need the subagent's tool call history

```
✅  "Use a subagent to scan all vendor folders and return a table:
     | Vendor | File count | Newest file (by filename date pattern) |
     Use that table to update README.md's vendor section counts."
```

The subagent runs in isolation, returns its result, and the main agent acts on it.

**Don't use subagents for:**
- Simple single-file reads
- Tasks where you need step-by-step visibility
- Tasks that require interactive back-and-forth

---

## Pattern 7: Output Contract

When one stage produces output for another, define the format of the handoff explicitly:

```
✅  "Stage 1: read all files under Misc/ and produce a JSON array:
     [{"file": "path", "vendor": "inferred vendor", "model": "inferred model name"}]
     
     Stage 2: use that array to add missing rows to the Misc section of README.md."
```

An explicit output contract prevents stage 2 from having to re-interpret ambiguous prose from stage 1.

---

## Anti-Patterns in Orchestration

### Over-specifying the How
```
❌  "First open the file with the read tool, then use grep to find the table,
     then count the rows, then use the replace tool to..."
```
The agent knows which tools to use. Specify the **what**, not the **how**:
```
✅  "Count the rows in the Recently Updated table in README.md and report the count."
```

### Under-specifying the Stopping Condition
```
❌  "Update all the vendor tables in README.md"
```
The agent doesn't know when "done" is. Add a measurable end state:
```
✅  "Update vendor tables in README.md so that every .md file under each vendor folder 
     has a corresponding row. Stop when all files are represented."
```

### Sequential When Parallel Is Possible
```
❌  (three separate prompts for three independent file reads)
✅  "Read all three simultaneously: Anthropic/README.md, OpenAI/README.md, Google/README.md"
```

### No Error Branch
For tasks that might fail (file not found, git command error), specify recovery:
```
✅  "Run: git push. If it fails with 'rejected', run: git pull --rebase then git push again."
```

---

## Orchestration Complexity Scale

| Level | Pattern | Example |
|---|---|---|
| 1 | Single tool | "Read README.md lines 1–30" |
| 2 | Two sequential tools | "Grep for X, then edit the matched line" |
| 3 | Parallel fan-out | "Read 3 files simultaneously" |
| 4 | Collect → Transform → Write | "List all files, derive table rows, insert into README" |
| 5 | Conditional + pipeline | "Check if X exists; if yes update, if no create; then commit" |
| 6 | Subagent + main agent | "Subagent scans repo, returns structured data; main agent writes results" |
