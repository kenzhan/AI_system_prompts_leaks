# Prompt Examples — Level 1 to Level 5

Progressive examples using this repository as the context. Each level introduces more complexity in tool use, orchestration, and output precision.

---

## Level 1 — Single Action, Known Target

**Task:** Read a specific file.

```
❌  "Can you please take a look at the Claude Sonnet 5 system prompt 
     and tell me what it says about the product name?"

✅  "Read Anthropic/claude-sonnet-5.md. Return only the first sentence 
     that identifies the model name."
```

**Why it's better:**
- Names the file directly (no search)
- Specifies the output constraint ("first sentence that…")
- No pleasantries consuming tokens

**Token delta:** ~40 tokens saved. Zero extra tool calls.

> **Textbook example — Claude Sonnet 5 system prompt (identity block):**
> ```
> This iteration of Claude is Claude Sonnet 5.
> ```
> One declarative sentence. No hedging, no description. The subject is named before anything else is said. Apply the same discipline to your prompts: state the exact target first (`Anthropic/claude-sonnet-5.md`), then the task. A description (*"the Claude Sonnet 5 prompt"*) forces a search step; a path does not.

---

## Level 2 — Search + Targeted Edit

**Task:** Add a new prompt file and update the index.

```
❌  "I just added a new file for Google's Gemini 3.5 Flash AI Studio version. 
     Can you update the readme to reflect this please?"

✅  "In README.md:
     1. Add to Recently Updated table (top): 
        | **Gemini 3.5 Flash AI Studio** | July 21, 2026 | [Google/gemini-3.5-flash-ai-studio.md](Google/gemini-3.5-flash-ai-studio.md) |
     2. Add to the Google vendor table:
        | Gemini 3.5 Flash AI Studio | [Google/gemini-3.5-flash-ai-studio.md](Google/gemini-3.5-flash-ai-studio.md) |
     Make both edits simultaneously."
```

**Why it's better:**
- Provides the exact row content — no generate-from-scratch step
- "Simultaneously" → one `multi_replace_string_in_file` call instead of two sequential edits
- No ambiguity about which table or what format

**Token delta:** ~80 tokens saved. One tool call instead of three (read README → find tables → edit).

> **Textbook example — Gordon (Docker AI) system prompt (specificity + banned filler):**
> ```
> state a SPECIFIC, COMPREHENSIVE plan as a numbered list mentioning concrete files,
> commands, and techniques. Not vague ("I'll examine and optimize") — specific
> ("I'll 1) read the Dockerfile and project structure, 2) apply multi-stage build
> and layer caching, 3) rebuild and verify size reduction").
>
> BANNED WORDS — never write anywhere in any response:
> "Perfect" "Great" "Excellent" "Awesome" "Wonderful" "Fantastic"
> "Sure" "Absolutely" "Amazing" "Good"
> ```
> Gordon's prompt bans filler on the **output** side. Apply the same rule on the **input** side: remove every word that adds no information. Providing the exact table row text in your prompt means the model never has to invent content — the value is already there, the tool just places it.

---

## Level 3 — Conditional Check + Create or Update

**Task:** Ensure a vendor folder and README entry exist for a new vendor.

```
❌  "We should add support for Mistral AI in this repo. 
     Can you set that up for me?"

✅  "Check if Mistral/ folder exists.
     - If yes: list its files and report.
     - If no: create Mistral/mistral-medium-3.5.md (empty file as placeholder).
     Then check README.md for a '## Mistral' vendor section.
     - If missing: add a new vendor section after the '## Microsoft' section with a placeholder table row."
```

**Why it's better:**
- Explicit conditional branches eliminate stop-and-ask behavior
- Checks both the folder and the README entry — two things that must stay in sync
- Defines the insertion point (`after '## Microsoft'`) — no search to locate it

**Result:** Agent runs file_search → conditional create → grep → conditional edit. No back-and-forth.

> **Textbook example — Gordon (Docker AI) system prompt (explicit routing branches):**
> ```
> If you are the best to answer the question according to your description,
> you can answer it.
>
> If another agent is better for answering the question according to its
> description, call `transfer_task` function to transfer the question to
> that agent using the agent's ID. When transferring, do not generate any
> text other than the function call.
> ```
> Every decision point has an explicit branch. No fallback to "figure it out." Notice the outcome for each path is also specified — what to call, what NOT to generate. Your Level 3 prompt follows this structure exactly: every `if yes` and `if no` names the action. Models follow explicit branches reliably; they over-think when given vague goals.

---

## Level 4 — Bulk Collect → Transform → Write

**Task:** Audit and sync the README vendor tables against actual files on disk.

```
❌  "The README might be out of date. Can you check 
     if all the files are listed properly?"

✅  "Audit the Google/ folder:
     1. List all .md files in Google/ (file_search).
     2. Read the Google vendor table in README.md (grep for '## Google').
     3. Compare: identify files in Google/ that have no row in the vendor table.
     4. For each missing file, add a row:
        | <Title-cased model name from filename> | [filename](Google/filename.md) |
        Insert after the last existing row in the Google table.
     Report the count of rows added."
```

**Why it's better:**
- Stages are explicit: collect (1), read current state (2), diff (3), write (4)
- The "report the count" stopping condition tells the agent when to stop
- Specifies row format so no generation decision is needed

**Token delta:** No back-and-forth clarification. Single coherent execution.

> **Textbook example — Gemini CLI system prompt (stage discipline and context cost):**
> ```
> Combine turns whenever possible by utilizing parallel searching and reading
> and by requesting enough context before grep_search to enable you to skip
> using an extra turn reading the file.
>
> Unnecessary turns are generally more expensive than other types of
> wasted context.
>
> Examples:
> - Searching: use grep_search with a conservative result count and a narrow scope.
> - Searching and editing: use grep_search with context/before/after to avoid
>   reading the file before editing.
> - Large files: use grep_search and read_file in parallel with start_line/end_line.
> ```
> Gemini CLI's system prompt structures every operation as: search narrow → read minimum → edit once. The Level 4 prompt mirrors this: each numbered stage minimizes extra turns by pre-specifying what to collect, what to compare, and what format to write. Numbering your stages is not style — it activates the agent's built-in efficiency mode.

---

## Level 5 — Full Pipeline: Research → Decide → Write → Commit

**Task:** Add a batch of new prompts and publish them.

```
❌  "I have some new system prompts to add. 
     Can you help me get them into the repo and push them?"

✅  "New files to add (already created on disk):
     - OpenAI/gpt-5.7-mini.md
     - OpenAI/gpt-5.7-thinking.md
     - Anthropic/claude-haiku-4.6.md

     Step 1 — Verify files exist: run `git status` and confirm all three appear as untracked.
     
     Step 2 — Update README.md simultaneously:
       a. Add three rows to Recently Updated (top table), dated July 21, 2026.
       b. Add gpt-5.7-mini and gpt-5.7-thinking to the OpenAI vendor table.
       c. Add claude-haiku-4.6 to the Anthropic model table.
     
     Step 3 — Stage and commit:
       Run: git add OpenAI/gpt-5.7-mini.md OpenAI/gpt-5.7-thinking.md Anthropic/claude-haiku-4.6.md README.md
       Run: git commit -m "Add GPT-5.7 mini/thinking and Claude Haiku 4.6"
     
     Step 4 — Push: run `git push`
     
     Stop after each step and report success/failure before proceeding."
```

**Why it's better:**
- The agent doesn't need to discover what needs adding — it's told
- Steps have explicit checkpoints ("Stop after each step and report")
- Git commands are written out — no ambiguity
- README changes are batched into one simultaneous edit (step 2)

**What would go wrong without structure:**
- Agent might search for the files instead of checking git status
- Might update README tables one at a time (3 round-trips instead of 1)
- Might commit without verifying the files exist
- Might push immediately after editing without a checkpoint

> **Textbook example — GitHub Copilot CLI system prompt (parallel tool calling):**
> ```
> CRITICAL: Maximize tool efficiency:
> USE PARALLEL TOOL CALLING — when you need to perform multiple independent
> operations, make ALL tool calls in a SINGLE response. For example, if you
> need to read 3 files, make 3 Read tool calls in one response, NOT 3
> sequential responses.
> Chain related bash commands with && instead of separate calls.
> Suppress verbose output (use --quiet, --no-pager, pipe to grep/head).
> ```
> The capability is already built in. Writing *"simultaneously"* in your prompt is the signal that unlocks it. Without that word, the agent defaults to sequential — one edit, wait for response, next edit. One word cuts three round-trips to one.

---

## Level 5b — Subagent-Assisted Research Pipeline

**Task:** Find all vendor folders with no README.md and generate stub files for them.

```
✅  "Use a subagent to:
       - List all top-level vendor folders (Anthropic/, OpenAI/, Google/, etc.)
       - Check each for the presence of a README.md
       - Return a JSON array: [{"folder": "Vendor/", "hasReadme": true|false}]
     
     Then for each folder where hasReadme is false:
       Create <folder>/README.md with this template:
       ---
       # <FolderName> — which file is which product?
       
       | File | Product |
       |------|---------|
       | `*.md` | (fill in) |
       ---
     
     List the files created. No other output."
```

**Why subagent here:**
- The research phase (scan all folders) produces a single structured result
- Isolating it in a subagent keeps the main context clean
- The main agent only needs the JSON array to act — not the full exploration history

> **Textbook example — GitHub Copilot CLI system prompt (sub-agent context rule):**
> ```
> When prompting sub-agents, provide comprehensive context —
> brevity rules do not apply to sub-agent prompts.
> ```
> Nine words that invert the normal rule. Sub-agents start in a completely fresh session: no conversation history, no open files, no prior context. The JSON output contract in Level 5b is the application of this principle — the structured return value must be fully self-contained so the main agent can act on it without knowing anything about the sub-agent's session.

---

## Pattern Summary

| Level | Key technique | Prompt signal |
|---|---|---|
| 1 | Name the target directly | File path + exact output constraint |
| 2 | Provide the final value | Give the exact text to insert |
| 3 | Explicit conditional branches | "If yes: … If no: …" |
| 4 | Collect → diff → write stages | Numbered steps with defined output format |
| 5 | Full pipeline with checkpoints | Numbered steps + "stop and report after each step" |
| 5b | Subagent for research isolation | "Use a subagent to… return JSON… then…" |

---

## Lessons Extracted from Real System Prompts

The system prompts in this collection reveal how production AI products instruct their models to operate efficiently. Each row below is a pattern drawn directly from a real prompt, paired with its user-prompt equivalent.

| Product | Verbatim pattern | Your prompt equivalent |
|---|---|---|
| GitHub Copilot CLI | `"USE PARALLEL TOOL CALLING — make ALL tool calls in a SINGLE response"` | Write "simultaneously" or "in parallel" for independent ops |
| GitHub Copilot CLI | `"brevity rules do not apply to sub-agent prompts"` | Give subagents full self-contained context; they start with no history |
| GitHub Copilot CLI | `"preference order: code intelligence > LSP > glob > grep"` | Name the search type: "grep for X" is more precise than "find X" |
| Gemini CLI | `"Unnecessary turns are generally more expensive than other wasted context"` | Front-load all context; eliminate any prompt that needs a follow-up |
| Gemini CLI | `"Combine turns by requesting enough context to skip an extra read"` | Number stages; each stage should produce enough output for the next |
| Gordon (Docker AI) | `"BANNED WORDS: Perfect Great Excellent Sure Absolutely Amazing Good"` | Remove input filler: no "please", "feel free", "if you could", "thanks" |
| Gordon (Docker AI) | `"ALL intermediate messages between tool calls MUST be \"\""` | Add: "No explanation between steps. Final result only." |
| Gordon (Docker AI) | `"state a SPECIFIC, COMPREHENSIVE plan... not vague... specific"` | Give the exact value to insert, not a description of it |
| Claude Sonnet 5 | `"This iteration of Claude is Claude Sonnet 5."` (identity block) | Open with the subject: "You are reviewing Anthropic/claude-sonnet-5.md" |
| Claude Sonnet 5 | `"Claude searches docs.claude.com and answers from the documentation"` | Define the fallback explicitly: "If not found in X, search Y, then answer." |

### Key Takeaway

Production system prompts are optimized for the same goals you have as a user: reduce unnecessary turns, route to the right tool, and produce predictable output. When you write structured, imperative, branch-explicit prompts, you align with the model's system-level operating mode rather than working against it.
