# AGENTS.md — AI Agent Instructions

This repository is a curated collection of verbatim leaked system prompts from AI chatbots (ChatGPT, Claude, Gemini, Grok, etc.).

## Project Structure

```
<VendorName>/         # One folder per vendor/company
  <model-name>.md     # Raw system prompt file
README.md             # Root index — tables listing every prompt by vendor
.github/CONTRIBUTING.md
```

Key vendor folders: `Anthropic/`, `OpenAI/`, `Google/`, `xAI/`, `Perplexity/`, `Microsoft/`, `Misc/` (catch-all).

See [Anthropic/README.md](Anthropic/README.md) for an example of sub-folder conventions (model variants, integrations, tools).

## File Naming Convention

- Lowercase, hyphen-separated: `gemini-3.5-flash.md`, `claude-opus-4.8.md`
- Match the model or product name as closely as possible
- Variants use suffixes: `-no-tools.md`, `-api.md`, `-thinking.md`, `-free-account.md`

## Adding a New Prompt

1. Choose the correct vendor folder. Use `Misc/` for products without a dedicated folder.
2. Create a single `.md` file with the raw prompt text — **no summarizing, paraphrasing, or editing**.
3. Update [README.md](README.md):
   - Add a row to the **"Recently Updated"** table at the top (columns: What, Date, Link).
   - Add a row to the appropriate vendor table further down in the file.

## Content Rules

- Paste the prompt **verbatim**. The full, unedited text is the entire value of this repo.
- Do **not** add commentary, headers, or formatting that wasn't in the original prompt.
- Sub-folders are used when a vendor has many variants (e.g., `Anthropic/Claude Code/`, `OpenAI/Codex/`, `OpenAI/ChatGPT/`).

## README Update Pattern

"Recently Updated" row format:
```markdown
| **Product Name** | Month DD, YYYY | [Short description](Path/to/file.md) |
```

Vendor table row format (see existing tables for the correct columns per vendor).
