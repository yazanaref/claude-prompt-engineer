# prompt-engineer — a Claude Skill for writing and debugging prompts

A drop-in Claude Skill that turns any Claude conversation into a focused prompt-engineering session. Ask Claude to write a new prompt, fix a broken one, or explain why an LLM is misbehaving, and this skill activates a structured workflow distilled from Anthropic's official prompt-engineering curriculum.

Built by [Yazan Aref](https://linkedin.com/in/yazanaref) from [Anthropic's *Prompt Engineering Interactive Tutorial*](https://github.com/anthropics/prompt-eng-interactive-tutorial), packaged as an installable `.skill` file for Claude.

---

## What this is

A Claude Skill (the new packaged-instruction format Anthropic released for claude.ai, Claude Code, and the API) that activates whenever you ask for help with a prompt. Once installed, Claude will:

- **Write prompts from scratch** when you describe a task — picking the right weight of structure for the job, applying the techniques that earn their place, and delivering a clean copy-pasteable result.
- **Diagnose and rewrite broken prompts** when you paste one in — matching symptoms (preamble you didn't want, inconsistent JSON, hallucinated facts, position bias, ignored rules) to the underlying causes and applying the right fix instead of piling on more words.
- **Adapt to the target** — API/system prompts (full toolbox including prefill and the `system` field) or chat prompts pasted into Claude.ai (no prefill, role framing folded into the message).

It is *not* a passive reference. It changes how Claude approaches the request — running a triage first (writing vs. improving, simple vs. complex, API vs. chat), then reaching for techniques in a disciplined order.

## Why it exists

Anthropic's [prompt-engineering tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) is one of the best free resources in the field — nine chapters of clear lessons with executable exercises, covering everything from basic message structure to building complex prompts for legal and financial use cases. The problem is that a tutorial is a *learning* artifact: you read it once, you internalize what you can, and the rest stays in a folder of `.ipynb` files you may or may not open again.

This skill turns that knowledge into something Claude consults on every relevant turn. You don't have to remember the ten-element framework for complex prompts, the difference between when prefill helps and when it isn't available, or which symptom maps to which cause — the skill encodes all of that and applies it for you. The principles are the same; the access pattern is what changes.

## Why it's good

Three things make this skill more useful than just "tell Claude to be a prompt engineer":

1. **Distilled, not duplicated.** The original tutorial is excellent but dated in spots — it uses Claude 3 Haiku, predates extended thinking, and over-applies "always think step by step." This skill pulls the durable principles forward (clarity, examples, XML delimitation, structured reasoning where it actually helps, role/tone, format anchoring, hallucination guards, the complex-prompt framework) and drops the version-specific scaffolding. Modern Claude is smart enough that judgment matters more than ritual; the skill is written to reflect that.

2. **Symptom → cause → fix.** The improve-an-existing-prompt path uses a diagnostic table that maps observable failures ("output format varies," "ignores some instructions," "makes up facts") to their likely causes and the specific technique that closes the gap. This is the part most prompt guides leave out — they teach you techniques but not which technique fixes which problem.

3. **Tight by design.** The skill explicitly budgets its commentary — the prompt is the deliverable, the explanation is the receipt. You get the rewritten prompt with a few sentences of prose around it, not a four-bullet "what changed and why" appendix on every reply.

## What's inside

```
prompt-engineer/
├── SKILL.md                                  ← entry point: mental model, triage, diagnostic table, technique 80/20
└── references/
    ├── technique-library.md                  ← 9 core techniques with examples (clarity, few-shot, XML,
    │                                            reasoning, roles, formatting/prefill, anti-hallucination,
    │                                            chaining, tool use)
    └── complex-prompt-framework.md           ← the 10-element structure for heavyweight prompts,
                                                 with a worked example (chatbot persona + rules + examples)
```

The skill uses Anthropic's *progressive disclosure* pattern: only `SKILL.md` is loaded into context when the skill triggers. The reference files are pulled in only when Claude actually needs the deeper material (a complex prompt build, or a technique it wants to apply precisely). That keeps the always-on footprint small.

## Installation

1. Download [`prompt-engineer.skill`](./prompt-engineer.skill) from this repo.
2. In [Claude.ai](https://claude.ai), open **Settings → Customize → Skills**, then upload the `.skill` file. (For Claude Code, place the unpacked folder under `~/.claude/skills/`.)
3. That's it — the skill is now available to every conversation. It activates automatically when you ask for help with a prompt.

## Usage

You invoke it manually (/prompt-engineer) or just ask for prompt help in the way you naturally would. Examples that will trigger it:

- *"`/prompt-engineer~` Help me write a system prompt for a customer-support chatbot for my SaaS product."*
- *"My JSON extractor keeps wrapping the output in markdown backticks. Fix it."*
- *"Why does Claude keep adding 'Here is the…' before every answer? How do I stop that?"*
- *"Turn this into a reusable template I can run a transcript through."*
- *"This prompt is huge and probably overkill. Tighten it without breaking it."*

The skill produces short prompts inline in chat and longer ones (reusable templates, system prompts) as files you can download and save.

## Credits

The entire intellectual substance of this skill is drawn from Anthropic's [**Prompt Engineering Interactive Tutorial**](https://github.com/anthropics/prompt-eng-interactive-tutorial) — the lessons, the worked examples, the ten-element complex-prompt framework, and the diagnostic patterns all come from there. If you want to *learn* prompt engineering rather than just have Claude do it for you, work through the tutorial yourself; it's the best free resource on the topic.

This skill repackages that material into the [Anthropic Skills](https://www.anthropic.com/news/skills) format so Claude can apply it reliably on every relevant turn.

Built using Anthropic's [`skill-creator`](https://github.com/anthropics/skills) workflow, which provides the draft → test → iterate → package loop for producing high-quality skills.

## License & attribution

The original tutorial is published by Anthropic. This skill is an unofficial distillation and is not affiliated with or endorsed by Anthropic. All credit for the underlying material belongs to the tutorial's authors; any errors in the distillation are mine.

## Author

**Yazan Aref**
LinkedIn: [linkedin.com/in/yazanaref](https://linkedin.com/in/yazanaref)
Email: [YazanAref079@gmail.com](mailto:YazanAref079@gmail.com)

If you find this useful, or have ideas for what to add or change, get in touch — feedback is welcome.
