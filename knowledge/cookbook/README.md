# Prompt Cookbook — Index

Guides for writing prompts that are token-efficient, direct, and reliably routed to the right tools.

## Quick-Reference Card

### Language Rules (apply to every prompt)

| ❌ Remove | ✅ Replace with |
|-----------|----------------|
| "Can you please…" | Imperative verb: "List…", "Fix…", "Create…" |
| "I was wondering if…" | Direct statement of what you need |
| "Would you be able to…" | Just ask |
| "Feel free to…" | Delete entirely |
| "Thank you in advance" | Delete entirely |
| "Please make sure to…" | State the constraint directly |
| "I think maybe…" | State your intent clearly |

### Token Budget Rules

- **One sentence = one intent.** Don't bundle unrelated requests.
- **State the output format first**, not last.
- **Say what to skip.** "No explanation." / "No summary." / "Code only."
- **Constrain scope.** "Only modify `src/auth.ts`" beats a vague description.
- **Name the file.** Don't describe what a file does — give its path.

---

## Files in this folder

| File | What it covers |
|------|----------------|
| [token-efficiency.md](token-efficiency.md) | Writing concise, imperative prompts; avoiding AI over-thinking |
| [tool-routing.md](tool-routing.md) | Directing the agent to the right tool at the right time |
| [orchestration.md](orchestration.md) | Multi-step pipelines, parallel execution, subagent patterns |
| [examples.md](examples.md) | Progressive examples — Level 1 (simple) → Level 5 (complex orchestration) |
