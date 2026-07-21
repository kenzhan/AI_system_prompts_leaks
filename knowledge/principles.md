# Core Principles for Effective Prompts

## 1. Be Explicit, Not Implicit

State exactly what you want. Models fill ambiguity with their defaults, which may not match your intent.

**Weak:** "Answer helpfully."  
**Strong:** "Answer in plain English. Use numbered steps for procedures. Keep responses under 200 words unless the question requires more."

---

## 2. Positive Instructions Over Prohibitions

Tell the model what to do, not only what to avoid. Prohibition-only prompts leave behavior undefined.

**Weak:** "Don't be verbose. Don't use jargon."  
**Strong:** "Use concise, plain language. Target a general audience with no technical background."

---

## 3. Persona Before Task

Define the model's role or identity before describing its tasks. This anchors tone, expertise level, and decision-making throughout.

```
You are a senior software engineer specializing in Python and distributed systems.
Your job is to review code for correctness, security, and maintainability.
```

---

## 4. Specify Output Format Explicitly

If you want structured output (JSON, Markdown table, bullet list), say so. Models default to prose otherwise.

```
Return a JSON object with keys: "summary" (string), "severity" (low|medium|high), "fix" (string).
```

---

## 5. Provide Examples (Few-Shot)

One or two concrete input → output examples dramatically improve accuracy on non-obvious tasks.

```
Example:
Input: "GPT-4 system prompt from May 2025"
Output file: OpenAI/gpt-4-2025-05.md
```

---

## 6. Separate Concerns with Structure

Use XML tags, headers, or labeled sections to separate the persona, context, task, constraints, and format. This reduces confusion when the prompt is long.

```xml
<persona>...</persona>
<task>...</task>
<constraints>...</constraints>
<output_format>...</output_format>
```

---

## 7. Chain of Thought for Complex Reasoning

For multi-step or analytical tasks, ask the model to reason before concluding.

```
Think step by step before giving your final answer.
```

Or assign the reasoning to a specific XML block so it doesn't pollute the final output:

```
Put your reasoning in <thinking> tags. Only the content after </thinking> will be shown to the user.
```

---

## 8. Ground Truth Over General Knowledge

When accuracy matters, give the model the source material and tell it to answer from that, not from memory.

```
Use only the text below to answer. Do not use outside knowledge.
<source>...</source>
```

---

## 9. Temperature and Tone Signals

Describe the desired tone in behavioral terms, not abstract adjectives.

**Vague:** "Be professional."  
**Precise:** "Respond as you would in a formal written report. No contractions. Cite sources inline."

---

## 10. End with the Task

Place the actual task or question at the end of the prompt, after all context and instructions. Models give more weight to the end of the input.
