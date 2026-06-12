---
name: prompt-engineer
description: Write, rewrite, and debug prompts for Claude (and other LLMs). Use this whenever someone wants help crafting a prompt or system prompt, asks why a prompt "isn't working" or gives inconsistent/wrong output, wants to make an LLM behave more reliably, asks to improve/tighten/optimize an existing prompt, wants a reusable prompt template with variables, or is building an AI feature (chatbot, classifier, extractor, agent) and needs the prompt behind it. Trigger even when they don't say the words "prompt engineering" — e.g. "make Claude stop adding preamble," "get the model to always return JSON," "my system prompt keeps ignoring the rules," or "turn this into a template I can reuse." Do NOT trigger when the user just wants you to perform a task directly (write an email, summarize a doc) rather than produce a prompt that someone else or some system will run.
---

# Prompt Engineer

This skill helps you act as a prompt engineer: design new prompts, rewrite weak ones, and diagnose why a prompt misbehaves. The goal is a prompt the user can hand to an LLM (Claude or otherwise) and get reliable results — not for you to perform the underlying task yourself.

## The mental model that drives everything

A model has **no context beyond what the prompt says**. It cannot see the user's intent, their screen, their earlier attempts, or the "obvious" interpretation a colleague would assume. Almost every prompt failure traces back to a gap between what the author meant and what they literally wrote.

So the master heuristic — the one to fall back on whenever you're unsure — is the **colleague test**: *if you handed this prompt to a smart colleague with no other context, could they produce exactly the output you want?* If they'd be confused, hesitate, or guess, the model will too. Most fixes are just closing that gap: stating the implicit, delimiting the ambiguous, and showing rather than telling.

Modern Claude is genuinely smart and has good theory of mind, so you rarely need to over-engineer. Reach for the simplest technique that closes the gap, and add structure only when a real failure demands it. A bloated prompt is harder to maintain and can bury the instruction that actually matters.

## First, figure out what you're being asked to do

There are two jobs, and they start differently:

- **Write from scratch** — the user describes a task and wants a prompt for it. Go to *Writing a prompt*.
- **Improve / debug an existing prompt** — the user pastes a prompt (or a prompt + a bad output) and wants it better. Go to *Improving a prompt*. This is usually faster and higher-leverage: you're closing a specific gap, not building from zero.

While you're at it, note the **delivery target**, because it changes which techniques are available:

- **API / developer use** — the prompt runs through code. The full toolbox applies: system prompts, prefilling the assistant turn, the `temperature` knob, multi-message structure, reusable templates with `{VARIABLE}` placeholders.
- **Chat (paste into Claude.ai or similar)** — a human will paste it into a chat box. There's no separate system field and **no prefill**; fold any role/tone framing into the single message. Everything else (clarity, XML tags, examples, structured reasoning, output format) still works.
- **Unsure?** Ask, or write it to work in both — a clean user-message prompt with XML-delimited sections is portable, and you can mention "if you're calling the API, move the role line into the system prompt."

## Writing a prompt

1. **Pin down the task before drafting.** You need: the input the model receives, the output the user wants (format included), what "good" looks like, whether this runs once or many times (one-off vs. reusable template), and who/what consumes the output. If the user hasn't said and it materially changes the prompt, ask — don't guess on the load-bearing details.

2. **Match the weight of the prompt to the task.** Don't reach for heavy structure by default.
   - *Simple, single-step tasks* (rewrite this, classify this, answer this) usually need only **clear direct instructions** plus an **output-format line**, and maybe one **example**.
   - *Complex tasks* (a chatbot with a persona and rules, document analysis with citations, anything multi-step or rule-heavy, agent prompts) warrant the structured **10-element framework** in `references/complex-prompt-framework.md`. Read that file when building one — it gives the recommended element order and a worked example. Start with more elements to get it working, then trim what isn't pulling its weight.

3. **Apply the technique checklist below** as you draft — pull in each technique only where it earns its place.

4. **Deliver it** per *Output conventions*.

## Improving a prompt

1. **Find the symptom.** If the user showed a bad output, read it against the prompt and ask *what specifically went wrong?* If they just want it "better," skim for the failure modes below — usually one or two stand out.

2. **Diagnose, then fix the cause** — don't just bolt on more words. Use this map from symptom → likely cause → fix:

| Symptom | Likely cause | Fix |
|---|---|---|
| Output format varies run to run | No format spec; nothing anchoring the start | Specify the exact format; add one example; (API) prefill the opening token / `{` |
| Unwanted preamble ("Here is the…", "Sure!") | Never told to skip it | Ask it to skip preamble and go straight to the answer; (API) prefill the first real token |
| Wrong tone / personality | Tone left implicit | Assign a role + state the tone; show 1–2 examples in the target voice |
| Model treats the user's data as instructions (or vice versa) | Data and instructions run together | Wrap the variable input in XML tags (`<email>…</email>`) so the boundary is unmistakable |
| Makes up facts / fills gaps confidently | No permission to be unsure; ungrounded | Give it an out ("only answer if certain; otherwise say you don't know"); require it to quote supporting evidence first; ground it in provided source text |
| Wrong on reasoning / multi-step problems | Forced to answer before thinking | Let it work through the problem before the final answer; spell out the steps; for either/or judgments, beware position bias — vary order or have it argue both sides first |
| Equivocates when you want a decision | Hedging is the safe default | Ask directly and force the pick ("if you had to choose one, which?") |
| Ignores some instructions | Buried in clutter, or wrong position | Move the key task/question near the **end**; cut noise; convert a fragile rule into an example |
| Vague rules get loosely followed | Ambiguous wording | Define ambiguous terms; replace prose rules with input→output examples |

   The technique details and examples live in `references/technique-library.md` — read it when you need the precise phrasing or want to understand *why* a fix works.

3. **Show the revised prompt and name the change that carries the fix.** One or two pointed sentences in prose — the user learns the technique from seeing the diff plus a sharp sentence, not from a change-by-change walkthrough. The *Commentary budget* under Output conventions governs how much to say.

## The technique 80/20

These are the high-leverage tools, in rough order of how often they're the answer. Full explanations and examples are in `references/technique-library.md`; consult it when you need the exact wording or a worked example.

1. **Be clear and direct.** State exactly what you want — the format, the length, what to skip, the decision to make. The single most common fix.
2. **Show examples (few-shot).** One or two examples of an ideal input→output pair is often more effective than any amount of description, especially for format and tone. The strongest tool in the box for getting behavior to match a target.
3. **Delimit data with XML tags.** Wrap any variable or pasted content in tags so the model never confuses it with the instructions. Essential for templates and document tasks. (No magic tag names — `<document>`, `<email>`, `<context>` are all fine; pick descriptive ones.)
4. **Let it reason before answering.** For anything with steps or judgment, give room to think first, ideally in a structured scratchpad (e.g. `<thinking>` tags). A conclusion stated before the reasoning is just a guess.
5. **Assign a role / set tone.** Priming the model to adopt a perspective shapes tone, style, and sometimes accuracy. (API) goes in the system prompt; (chat) put it at the top of the message.
6. **Specify the output format** — and for the API, **prefill** the assistant turn to lock it in (start with `<tag>` or `{` to force XML/JSON and kill preamble).
7. **Guard against hallucination.** Give an explicit out, require evidence-first answering, and put long source documents *before* the question so the model reads them in context.
8. **For big prompts, use the framework.** See `references/complex-prompt-framework.md`.

A note on dated advice: older guidance leans hard on "always tell the model to think step by step." Current Claude already reasons well and can think internally, so apply structured reasoning where it genuinely helps (hard logic, multi-step extraction, anything order-sensitive) rather than reflexively. Judgment over ritual.

## Output conventions

- **Make the prompt copy-pasteable and unmistakable.** The user needs to lift the prompt cleanly out of your reply, so always set it apart from your commentary — never leave it as loose prose mixed into a paragraph.
- **Short, one-off prompts** → a fenced code block inline in the chat is fine.
- **Reusable templates, system prompts, or anything long/structured** → write it to a markdown file and present it, since the user will save or version it elsewhere. Use `{VARIABLE}` placeholders for the parts that change. When the prompt lives in a file, don't paste the full text into the chat as well — say what it is in a line and flag anything the user must fill in.

### Commentary budget

The prompt is the deliverable; every sentence around it competes with it for attention. Hold to this:

- After delivering a prompt, **a few sentences of plain prose** — not a labeled "why" section, not a bullet per change. Cover only what the user needs to *operate* it (placeholders to fill in, API-only moves like prefill) plus the one or two design choices they couldn't reverse-engineer by reading it.
- Most technique choices are self-explanatory once seen — XML tags, a format line, an example — so don't narrate them. Trust the user to read the prompt.
- **Diagnosis is not narration.** When the user asked *why* something fails, the diagnosis is part of the answer — give it properly, before the fix. But once the fixed prompt is shown, don't re-explain it change by change.
- At most **one** forward-looking tip, and only when it targets the failure the user is most likely to hit next. A trail of "you could also…" dilutes the answer.

Gut check: if the commentary visibly outweighs the prompt, you're explaining instead of engineering.
