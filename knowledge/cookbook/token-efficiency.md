# ****

# Token Efficiency — Writing Prompts That Don't Waste Budget

Every token you spend on pleasantries, hedging, or redundant context is a token the model uses to generate preamble instead of output. The goal is **maximum signal, minimum noise**.

---

## Rule 1: Imperative First, Context After

Start with the verb. Put constraints and context after the action.

```
❌  "I was wondering if you could look at the README and maybe update the 
     Recently Updated table to include the new file I just added."

✅  "Update README.md — add a row to the Recently Updated table for the 
     new file Anthropic/claude-opus-5.md, dated July 21, 2026."
```

The imperative (`Update`) tells the model its job immediately. Context follows.

---

## Rule 2: Say What to Skip

Models default to explaining their reasoning, adding summaries, and asking clarifying questions. Preempt this:

```
❌  (says nothing — model adds 3 paragraphs of explanation)

✅  "No explanation. Code only."
✅  "Return the result only. Skip commentary."
✅  "One-line answer."
✅  "Diff only, no prose."
```

Without these signals, the model assumes you want a full response. With them, it stops early.

---

## Rule 3: Constrain Scope Explicitly

Vague scope forces the model to guess — and it guesses wide.

```
❌  "Fix the bug in the authentication module."

✅  "Fix the null-dereference on line 47 of src/auth/login.ts. 
     Change only that function. Don't touch other files."
```

Explicit scope:
- Prevents unintended edits to other files
- Reduces the number of tool calls to gather context
- Gives the model a clear stopping condition

---

## Rule 4: Specify Output Format Upfront

Format specification at the end is often ignored. Put it first or immediately after the verb.

```
❌  "List all vendor folders and then tell me as a markdown table."

✅  "Markdown table: list all vendor folders with a one-line description of each."
```

When the model knows the output shape before it starts thinking, it plans the answer around that shape rather than reformatting at the end.

---

## Rule 5: Give the Path, Not the Description

Never describe a file when you can name it.

```
❌  "Look at the file that contains the Anthropic system prompts listing."
✅  "Read Anthropic/README.md."

❌  "In the file where we track recently added prompts..."
✅  "In README.md, under ## Recently Updated..."
```

Descriptions require the model to search to resolve the reference. Paths are direct.

---

## Rule 6: Pre-Commit to Structure for Multi-Part Tasks

When asking for something with multiple parts, define the structure upfront so the model doesn't invent its own.

```
❌  "Tell me about the differences between Claude and GPT system prompts."

✅  "Compare claude.ai and ChatGPT system prompts across three dimensions:
     1. Identity block structure
     2. Tool declaration style
     3. Safety/refusal framing
     One paragraph per dimension. No intro, no conclusion."
```

---

## Rule 7: Avoid Open-Ended Questions

Open-ended questions trigger exhaustive answers. Constrain the answer space.

```
❌  "What do you think about this file structure?"
✅  "Does this file structure follow the naming convention in AGENTS.md? Yes/No, then one sentence."

❌  "How should I improve this prompt?"
✅  "Identify the single biggest inefficiency in this prompt. One sentence."
```

---

## Rule 8: Don't Re-Explain Shared Context

If the model has already read a file in this session, don't paste its contents again. Reference it:

```
❌  (pastes the full AGENTS.md content)

✅  "Using the conventions in AGENTS.md..."
```

If the model needs to re-read a file it has already seen, it will do so. Repasting burns context for no gain.

---

## Rule 9: Batch Independent Requests

If two things can be done in parallel (no dependency between them), ask for both in one prompt.

```
❌  (prompt 1) "Create knowledge/cookbook/README.md"
    (prompt 2) "Create knowledge/cookbook/token-efficiency.md"

✅  "Create both files simultaneously:
     - knowledge/cookbook/README.md — index and quick reference
     - knowledge/cookbook/token-efficiency.md — token efficiency guide"
```

This avoids a round-trip and the model can execute parallel tool calls.

---

## Rule 10: Leverage System Prompt Knowledge

The agent already knows the conventions in `AGENTS.md` and other instruction files. Don't repeat them — invoke them:

```
❌  "Name the file with lowercase hyphens and put it in the Google/ folder 
     and add a row to README.md in the Recently Updated section and also 
     the vendor table further down..."

✅  "Add the new Google prompt following AGENTS.md conventions."
```

The instruction file does the work of encoding the full procedure. Your prompt just needs to trigger it.

---

## Anti-Patterns That Cause Over-Thinking

| Anti-pattern | What happens | Fix |
|---|---|---|
| "What do you think?" | Model generates extensive opinion | Ask a yes/no or constrained question |
| No output format | Model picks verbose prose by default | Specify: table, list, code block, one-liner |
| Ambiguous pronoun ("fix it") | Model re-reads everything to resolve "it" | Name the file and line explicitly |
| "Try your best to…" | Model hedges and adds caveats | Just state the task |
| Asking for multiple alternatives | Model generates all of them | Ask for the single best option |
| "Feel free to ask if unclear" | Triggers a clarifying question loop | Provide the missing info upfront |
