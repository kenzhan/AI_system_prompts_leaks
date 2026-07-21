# ****

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

> **System prompt cross-reference:** [`Anthropic/claude-sonnet-5.md`](../../Anthropic/claude-sonnet-5.md)
> Claude's system prompt opens with the line: *"This iteration of Claude is Claude Sonnet 5."* — explicit naming that eliminates all ambiguity about which product the instructions apply to. Apply the same logic to your prompts: always give the exact file path (`Anthropic/claude-sonnet-5.md`) rather than a description of it (*"the Claude Sonnet 5 system prompt"*). Descriptions require a search to resolve; paths do not.

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

> **System prompt cross-reference:** [`Misc/docker-gordon-ai.md`](../../Misc/docker-gordon-ai.md) (Gordon — Docker AI)
> Gordon's prompt requires: *"SPECIFIC, COMPREHENSIVE plan... mentioning concrete files, commands, and techniques. Not vague ('I'll examine') — specific ('I'll 1) read the Dockerfile...')"*. It also maintains a BANNED WORDS list: *"Perfect", "Great", "Excellent", "Sure", "Absolutely"...* — all the filler that adds tokens without information. Providing the exact row text in your prompt enforces the same discipline on the input side: the model never needs to invent content, and there is no room for filler in the output.

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

> **System prompt cross-reference:** [`Misc/docker-gordon-ai.md`](../../Misc/docker-gordon-ai.md) (Gordon — Docker AI)
> Gordon's agent routing reads: *"If you are the best to answer the question according to your description, you can answer it. If another agent is better... call `transfer_task` to transfer the question to that agent."* The system prompt gives the model explicit if/else branches with no ambiguity. Your Level 3 prompt mirrors this exactly: every branch is named, every outcome is defined. Models follow explicit branches reliably — they guess when given vague goals like "set that up for me."

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

> **System prompt cross-reference:** [`Google/gemini-cli.md`](../../Google/gemini-cli.md) (Gemini CLI)
> Gemini CLI's system prompt dedicates an entire section to context efficiency: *"Combine turns whenever possible by utilizing parallel searching and reading... Unnecessary turns are generally more expensive than other types of wasted context."* It lists numbered patterns: search → read → edit — exactly the stage structure of Level 4. When you number your stages in a prompt, you activate the same discipline that production system prompts enforce internally.

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

> **System prompt cross-reference:** [`Microsoft/vscode-copilot-agent.md`](../../Microsoft/vscode-copilot-agent.md) (GitHub Copilot CLI)
> The Copilot CLI system prompt marks this in all-caps: *"CRITICAL: USE PARALLEL TOOL CALLING — when you need to perform multiple independent operations, make ALL tool calls in a SINGLE response."* Writing *"Update README.md simultaneously"* in your prompt directly invokes this built-in capability. Without the explicit signal, the agent may default to sequential edits. The system prompt already knows how to parallelize — your prompt just needs to tell it when to.

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

> **System prompt cross-reference:** [`Microsoft/vscode-copilot-agent.md`](../../Microsoft/vscode-copilot-agent.md) (GitHub Copilot CLI)
> The same Copilot CLI system prompt notes: *"When prompting sub-agents, provide comprehensive context — brevity rules do not apply to sub-agent prompts."* Sub-agents start in a fresh session with no conversation history and no access to the files the user has open. The JSON output contract in Level 5b ensures the main agent's result is fully self-contained — everything the next step needs is in the structured return value, not in session history.

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

## Lessons Extracted from the Repo's System Prompts

The system prompts collected here reveal how production AI products are instructed to behave efficiently. These patterns directly inform how to write better user prompts.

| System Prompt | Pattern used by the model | What to do in your prompt |
|---|---|---|
| [`Microsoft/vscode-copilot-agent.md`](../../Microsoft/vscode-copilot-agent.md) | "USE PARALLEL TOOL CALLING" for independent operations | Write "simultaneously" or "in parallel" when ops have no dependency |
| [`Microsoft/vscode-copilot-agent.md`](../../Microsoft/vscode-copilot-agent.md) | "Sub-agent prompts: brevity rules do not apply" | Give subagents full self-contained context; they have no session history |
| [`Microsoft/vscode-copilot-agent.md`](../../Microsoft/vscode-copilot-agent.md) | Tool preference order: code intelligence > LSP > glob > grep | Name the search type explicitly: "grep for X" beats "find X" |
| [`Google/gemini-cli.md`](../../Google/gemini-cli.md) | Numbered stage discipline: understand → search → read → edit | Number your stages; the model follows explicit sequencing reliably |
| [`Google/gemini-cli.md`](../../Google/gemini-cli.md) | "Unnecessary turns cost more than any other wasted context" | Front-load all context so no clarifying turn is needed |
| [`Misc/docker-gordon-ai.md`](../../Misc/docker-gordon-ai.md) | BANNED WORDS: "Perfect", "Great", "Sure", "Absolutely"… | Remove prompt filler too: no "please", "feel free", "if you could" |
| [`Misc/docker-gordon-ai.md`](../../Misc/docker-gordon-ai.md) | Intermediate messages between tool calls = `""` (empty) | "No explanation between steps. Final result only." |
| [`Misc/docker-gordon-ai.md`](../../Misc/docker-gordon-ai.md) | Explicit routing with named agents and transfer conditions | Name the tool or file explicitly; never describe what you want the model to find |
| [`Anthropic/claude-sonnet-5.md`](../../Anthropic/claude-sonnet-5.md) | Identity block names the exact model at the start | Start your prompt with the role/scope: "You are auditing the Google/ vendor folder for…" |
| [`Anthropic/claude-sonnet-5.md`](../../Anthropic/claude-sonnet-5.md) | Fallback defined: "search docs.claude.com then answer from it" | Always define the fallback: "If not found in X, check Y before answering." |

### Key Takeaway

Production system prompts are optimized for the same goals you have as a user: reduce unnecessary turns, route to the right tool, and produce predictable output. When you write structured, imperative, branch-explicit prompts, you align with the model's system-level operating mode rather than working against it.
