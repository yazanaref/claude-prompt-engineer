# prompt-engineer — a Claude Skill for writing and debugging prompts

A drop-in Claude Skill that turns any Claude conversation into a focused prompt-engineering session. Ask Claude to write a new prompt, fix a broken one, or explain why an LLM is misbehaving, and this skill activates a structured workflow distilled from Anthropic's official prompt-engineering curriculum.

Built by [Yazan Aref](https://linkedin.com/in/yazanaref) from [Anthropic's *Prompt Engineering Interactive Tutorial*](https://github.com/anthropics/prompt-eng-interactive-tutorial), packaged as an installable `.skill` file for Claude.

---

## What this is

A Claude Skill (the packaged-instruction format for claude.ai, Claude Code, and the API) that activates whenever you ask for help with a prompt. Once installed, Claude will:

- **Write prompts from scratch** when you describe a task — picking the right weight of structure for the job, applying the techniques that earn their place, and delivering a clean copy-pasteable result.
- **Diagnose and rewrite broken prompts** when you paste one in — matching symptoms (preamble you didn't want, inconsistent JSON, hallucinated facts, position bias, ignored rules) to their underlying causes and applying the right fix instead of piling on more words.
- **Adapt to the target** — API/system prompts (full toolbox including prefill and the `system` field) or chat prompts pasted into Claude.ai (no prefill, role framing folded into the message).

It is *not* a passive reference. It changes how Claude approaches the request — running a triage first (writing vs. improving, simple vs. complex, API vs. chat), then reaching for techniques in a disciplined order.

## Why it exists

Anthropic's [prompt-engineering tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) is one of the best free resources in the field — nine chapters of clear lessons with executable exercises, covering everything from basic message structure to building complex prompts for legal and financial use cases. The problem is that a tutorial is a *learning* artifact: you read it once, internalize what you can, and the rest stays in a folder of `.ipynb` files you may or may not open again.

This skill turns that knowledge into something Claude consults on every relevant turn. You don't have to remember the ten-element framework for complex prompts, the difference between when prefill helps and when it isn't available, or which symptom maps to which cause — the skill encodes all of that and applies it for you. The principles are the same; the access pattern is what changes.

## Why it's good

Three things make this skill more useful than just "tell Claude to be a prompt engineer":

1. **Distilled, not duplicated.** The original tutorial is excellent but dated in spots — it uses Claude 3 Haiku, predates extended thinking, and over-applies "always think step by step." This skill pulls the durable principles forward (clarity, examples, XML delimitation, structured reasoning where it actually helps, role/tone, format anchoring, hallucination guards, the complex-prompt framework) and drops the version-specific scaffolding. Modern Claude is smart enough that judgment matters more than ritual; the skill is written to reflect that.

2. **Symptom → cause → fix.** The improve-an-existing-prompt path uses a diagnostic table that maps observable failures ("output format varies," "ignores some instructions," "makes up facts") to their likely causes and the specific technique that closes the gap. This is the part most prompt guides leave out — they teach you techniques but not which technique fixes which problem.

3. **Tight by design.** The skill explicitly budgets its commentary — the prompt is the deliverable, the explanation is the receipt. You get the rewritten prompt with a few sentences of prose around it, not a four-bullet "what changed and why" appendix on every reply.

---

## Repo contents

```
claude-prompt-engineer/
├── prompt-engineer.skill                     ← install this
├── CHANGELOG.md
│
├── prompt-engineer/                          ← skill source (readable on GitHub)
│   ├── SKILL.md
│   └── references/
│       ├── technique-library.md
│       └── complex-prompt-framework.md
│
├── evals/
│   └── evals.json
│
├── examples/
│   ├── tallybook-support-system-prompt.md
│   └── interview-summarizer-template.md
│
└── docs/development/
    ├── iteration-1-review.md
    └── iteration-2-review.md
```

### [`prompt-engineer.skill`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/prompt-engineer.skill)
The installable skill file. Download and upload it to Claude Settings → Customize → Skills. This is a packaged zip of the `prompt-engineer/` source folder.

### [`CHANGELOG.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/CHANGELOG.md)
Development history — what changed between iterations and why, plus notes for contributors on how to test and re-package changes.

---

### [`prompt-engineer/`](https://github.com/yazanaref/claude-prompt-engineer/tree/main/prompt-engineer) — Skill source

The unpacked skill source. GitHub renders these as readable markdown, so you can browse the full skill logic without downloading anything.

#### [`prompt-engineer/SKILL.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/prompt-engineer/SKILL.md)
The entry point Claude loads when the skill triggers. Contains the core mental model, the write vs. improve triage, the symptom→cause→fix diagnostic table, the technique 80/20 ordered by frequency, and the output/commentary conventions.

#### [`prompt-engineer/references/technique-library.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/prompt-engineer/references/technique-library.md)
Nine core techniques with worked examples — clarity, few-shot prompting, XML data delimitation, structured reasoning, role and tone prompting, output formatting and prefilling, hallucination guards, prompt chaining, and tool use. Claude pulls this in when it needs precise phrasing or wants to understand *why* a technique works before applying it.

#### [`prompt-engineer/references/complex-prompt-framework.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/prompt-engineer/references/complex-prompt-framework.md)
The 10-element structure for heavyweight prompts (chatbots with personas, document analysis, multi-step or rule-dense tasks). Explains the recommended element order, which ordering decisions are load-bearing, and includes a fully worked career-coach chatbot example.

---

### [`evals/`](https://github.com/yazanaref/claude-prompt-engineer/tree/main/evals) — Test cases

#### [`evals/evals.json`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/evals/evals.json)
Five realistic test prompts used to develop and validate the skill. Covers the full range: improving a broken API JSON extractor, writing a chatbot system prompt from scratch, debugging a reasoning failure in the Claude.ai chat app, building a reusable transcript-summarizer template, and tightening an over-engineered prompt. If you make changes to the skill, run your version against these before packaging.

---

### [`examples/`](https://github.com/yazanaref/claude-prompt-engineer/tree/main/examples) — Sample outputs

Real prompts the skill produced during development, showing what to expect.

#### [`examples/tallybook-support-system-prompt.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/examples/tallybook-support-system-prompt.md)
A complete support-chatbot system prompt generated from a plain-language description. Shows how the skill applies the complex-prompt framework: role context, tone, rules with explicit outs, and two examples (one happy-path how-to, one refund escalation) that anchor the hardest behavior.

#### [`examples/interview-summarizer-template.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/examples/interview-summarizer-template.md)
A reusable API template for summarizing customer interview transcripts into takeaways, feature requests, and sentiment. Shows evidence-first design to block invented feature requests, fixed XML output tags for machine parsing, and `{TRANSCRIPT}` placeholder convention.

---

### [`docs/development/`](https://github.com/yazanaref/claude-prompt-engineer/tree/main/docs/development) — Build history

#### [`docs/development/iteration-1-review.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/docs/development/iteration-1-review.md)
Full outputs from the first test run across all five eval prompts. The prompts were correct but commentary was too heavy — each response ended with a labeled "what changed and why" bullet section.

#### [`docs/development/iteration-2-review.md`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/docs/development/iteration-2-review.md)
Outputs after adding the commentary budget to the skill. Same prompts, ~62% less surrounding text. Shows the before/after contrast clearly.

---

## Installation

1. Download [`prompt-engineer.skill`](https://github.com/yazanaref/claude-prompt-engineer/blob/main/prompt-engineer.skill) from this repo.
2. In [Claude.ai](https://claude.ai), open **Settings → Customize → Skills**, then upload the `.skill` file.  
   For Claude Code: place the unpacked [`prompt-engineer/`](https://github.com/yazanaref/claude-prompt-engineer/tree/main/prompt-engineer) folder under `~/.claude/skills/`.
3. Done — the skill is available to every conversation and activates automatically when you ask for prompt help.

## Usage

You can invoke it explicitly with `/prompt-engineer` or just ask naturally. Examples that trigger it:

- *"`/prompt-engineer` Help me write a system prompt for a customer-support chatbot."*
- *"My JSON extractor keeps wrapping the output in markdown backticks. Fix it."*
- *"Why does Claude keep adding 'Here is the…' before every answer? How do I stop that?"*
- *"Turn this into a reusable template I can drop a transcript into."*
- *"This prompt is huge and probably overkill. Tighten it without breaking it."*

Short prompts come back inline in chat. Longer ones (system prompts, reusable templates) are delivered as files you can download and save.

## Credits

The entire intellectual substance of this skill is drawn from Anthropic's [**Prompt Engineering Interactive Tutorial**](https://github.com/anthropics/prompt-eng-interactive-tutorial) — the lessons, the worked examples, the ten-element complex-prompt framework, and the diagnostic patterns all originate there. If you want to *learn* prompt engineering rather than just have Claude apply it, work through the tutorial yourself; it's the best free resource on the topic.

Built using Anthropic's [`skill-creator`](https://github.com/anthropics/skills) workflow — draft → test → iterate → package.

## License & attribution

The original tutorial is published by Anthropic. This skill is an unofficial distillation and is not affiliated with or endorsed by Anthropic. All credit for the underlying material belongs to the tutorial's authors; any errors in the distillation are mine.

## Author

**Yazan Aref**  
LinkedIn: [linkedin.com/in/yazanaref](https://linkedin.com/in/yazanaref)  
Email: [YazanAref079@gmail.com](mailto:YazanAref079@gmail.com)

If you find this useful, or have ideas for what to add or change, get in touch — feedback is welcome.
