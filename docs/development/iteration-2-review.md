# prompt-engineer skill — Iteration 2 Test Results

Skill change since iteration 1: added a **Commentary budget** to Output conventions (a few sentences of prose, no labeled "why" sections, no bullet-per-change, diagnosis only when asked, max one forward-looking tip, don't duplicate file contents in chat) and rewrote the improve-step to "name the change that carries the fix" instead of "explain the changes."

The prompts produced are identical to iteration 1 — only the commentary around them changed. Compare against iteration-1-review.md.

---

## Test 1 — Improve an inconsistent JSON extractor (API)

### Output produced

All three symptoms come from the same gap: the prompt never defines an output contract, so the model improvises — chat formatting, preamble, and clarifying questions whenever a field is missing. Here's the fix:

```
Extract the person's name, email, and company from the text below.

<text>
{text}
</text>

Return a single raw JSON object with exactly these keys: "name", "email", "company".
If a field is not present in the text, use null for that value. Always return the
JSON object — never ask clarifying questions, never add commentary, and do not wrap
the output in markdown code fences.
```

Since you're on the API, also prefill the assistant turn with `{` — add `{"role": "assistant", "content": "{"}` as the last message and prepend the brace when parsing. The model continues the object from there, which makes preamble and code fences structurally impossible. If you ever see format drift again, one example of a correct input→output pair under the rules is the strongest anchor.

---

## Test 2 — Write a support chatbot system prompt from scratch

### Output produced

Built on the standard chatbot structure (role → tone → rules → examples) and saved as **tallybook-support-system-prompt.md** — drop it into your API call's `system` parameter. One thing to fill in: `{ESCALATION_ACTION}` appears twice; replace it with your real refund/cancel handoff (tool call, link, or ticket flow). The two examples aren't decorative — one anchors the how-to tone, the other demonstrates the refund escalation, because the model imitates examples more reliably than it follows prose rules. When you have your FAQ handy, paste it into the prompt inside `<product_docs>` tags with "only describe features that appear in the docs" — grounding beats instructions for keeping invented features out.

*(File: tallybook-support-system-prompt.md — unchanged from iteration 1.)*

---

## Test 3 — Debug a reasoning failure (chat app)

### Output produced

It fails because it commits to an answer before doing any case analysis — "we don't know if Anne is married" pattern-matches straight to "can't be determined." The puzzle's trick is that Anne's status doesn't matter: if she's married, she's looking at unmarried George; if she's unmarried, married Jack is looking at her. Both branches land on yes. So the fix is forcing that enumeration to happen before the answer:

```
Solve this logic puzzle. Before giving a final answer, work through every possible
case for the unknown information — write out what follows in each case — and only
then conclude.

Jack is looking at Anne. Anne is looking at George. Jack is married, George is not,
and we don't know if Anne is married. Is a married person looking at an unmarried
person?
```

The named move ("work through every case") matters more than a generic "think carefully" — it tells the model exactly which step it's been skipping.

---

## Test 4 — Reusable template with variables (API)

### Output produced

Saved as **interview-summarizer-template.md** — one variable, `{TRANSCRIPT}`; your code parses the fixed `<summary>` block and discards the `<evidence>` block, which is the model's working space. That evidence-first pass is the load-bearing choice: feature requests have to be backed by quotes pulled *before* the summary, which is what blocks this task's classic failure — invented feature requests. If you want to suppress preamble entirely, prefill the assistant turn with `<evidence>`.

*(File: interview-summarizer-template.md — unchanged from iteration 1.)*

---

## Test 5 — Tighten a bloated prompt

### Output produced

Your instinct is right — this is a one-step task wearing a costume. Trimmed:

```
Rewrite the sentence below to be more concise while preserving its meaning.
Reply with only the rewritten sentence.

<sentence>
{sentence}
</sentence>
```

The persona, the step-by-step ritual, and the emphasis all go — none of them add capability on a one-line rewrite; they just bury the instruction. Two small additions the original was missing: the `<sentence>` tags, so input containing instruction-like wording can't bleed into the task, and "reply with only the rewritten sentence," since the original was one run away from preamble.
