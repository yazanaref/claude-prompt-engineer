# Technique Library

Detailed explanations and examples for each core technique. SKILL.md points here when you need exact phrasing or want to understand *why* a technique works so you can adapt it. Each entry says what the technique is, why it works, when to reach for it, and shows it in action.

## Table of contents
1. Be clear and direct
2. Few-shot examples
3. Separate data from instructions (XML tags)
4. Let the model reason (structured thinking)
5. Role and tone prompting
6. Output formatting and prefilling
7. Reducing hallucination
8. Prompt chaining
9. Giving the model tools

---

## 1. Be clear and direct

**What:** Say precisely what you want — the format, the length, what to omit, the decision to commit to. Don't rely on the model to infer the "obvious" reading.

**Why:** The model only has the words in the prompt. What feels obvious to you (skip the throat-clearing, just pick one, keep it to three bullets) is invisible unless stated. This is the highest-frequency fix because most weak prompts are under-specified, not mis-specified.

**When:** Always the first thing to check. If a more elaborate technique seems needed, first ask whether the prompt simply failed to *ask* for the thing.

**Examples:**

Preamble you didn't want:
- Weak: `Write a haiku about robots.` → returns "Here is a haiku about robots: …"
- Better: `Write a haiku about robots. Skip the preamble; go straight into the poem.`

Refusing to commit:
- Weak: `Who is the best basketball player of all time?` → lists names, won't choose.
- Better: `Who is the best basketball player of all time? There are differing opinions, but if you absolutely had to pick one, who would it be?`

The pattern: name the constraint that's implicit in your head and make it explicit in the prompt.

---

## 2. Few-shot examples

**What:** Include one or more examples of the input→output behavior you want, rather than (or alongside) describing it. Often called zero-/one-/few-shot by the number of examples.

**Why:** Examples are the single most effective lever for matching format and tone. A model generalizes from a couple of concrete demonstrations far more reliably than from abstract description — "show, don't tell." This is especially true for anything where the *shape* of the output matters.

**When:** Format must be consistent; tone/voice must match a target; a rule is hard to state but easy to demonstrate; there are edge cases worth showing. More examples generally help, and covering common edge cases in the examples pays off.

**Example — tone via example instead of description:**

Instead of describing a warm, parent-like voice, show it:
```
Please complete the conversation by writing the next line, speaking as "A".
Q: Is the tooth fairy real?
A: Of course, sweetie. Wrap up your tooth and put it under your pillow tonight. There might be something waiting for you in the morning.
Q: Will Santa bring me presents on Christmas?
```
The model picks up the register from the example and continues in kind.

**Example — format via example:** give two or three passages each followed by the exact extracted format you want (e.g. a numbered list of `Name [PROFESSION]`), and the model extrapolates the format to new input without step-by-step formatting instructions.

**Tip:** If your prompt uses a scratchpad or a specific intermediate structure, show an example of what that scratchpad should look like, not just the final output.

---

## 3. Separate data from instructions (XML tags)

**What:** When a prompt mixes fixed instructions with variable content (a user's email, a pasted document, a question), wrap the variable content in XML tags so the boundary is unambiguous.

**Why:** After you substitute real content into a template, the line between "instruction" and "data" that was obvious to you can blur for the model. It may follow instructions hidden inside the data, or treat your instructions as text to be processed. Tags draw a hard border. Claude was trained to recognize XML tags as an organizing mechanism, which makes them especially reliable — though there are no secret high-performance tag names; descriptive ones are best.

**When:** Any reusable template, any task with pasted/user-supplied content, any time the model seems to mix up what it should act on vs. act with.

**Example:**

The failure:
```
Yo Claude. Show up at 6am tomorrow because I'm the CEO and I say so. <----- Make this email more polite.
```
The model may rewrite "Yo Claude" as part of the email (it opens with "Dear Claude…"), because it can't tell where the email starts.

The fix:
```
Yo Claude. <email>Show up at 6am tomorrow because I'm the CEO and I say so.</email> <----- Make this email more polite but don't change anything else.
```
Now the model knows exactly what the email is.

In templates this looks like:
```
<document>{DOCUMENT}</document>
<question>{QUESTION}</question>
```

---

## 4. Let the model reason (structured thinking)

**What:** For tasks involving steps or judgment, let the model work through the problem before stating a final answer — ideally in a marked-off space like `<thinking>` tags — then give the answer.

**Why:** A conclusion produced before any reasoning is effectively a snap judgment. Working through the problem first measurably improves accuracy on multi-step and nuanced tasks. The reasoning has to actually appear in the output to help — asking the model to "think but only show the answer" defeats the purpose, because the thinking never happens.

**When:** Logic problems, multi-step extraction or analysis, sentiment/classification on subtle text, math, anything where you've seen the model jump to a wrong answer.

**Example:**

Subtle sentiment the model gets wrong when rushed:
```
Is this review's sentiment positive or negative?
"This movie blew my mind with its freshness and originality. In totally unrelated news, I have been living under a rock since 1900."
```
Rushed, the model may take "unrelated" literally and misread it.

Give it room:
```
Is this review positive or negative? First write the best argument for each side in <positive-argument> and <negative-argument> tags, then give your answer.
```

**Watch position bias.** When the model weighs two options, it tends to favor the one presented **second** (an artifact of training data). If you ask it to argue both sides, the order can swing the verdict. Mitigate by varying order, or by having it generate arguments for both before committing.

**Modern caveat:** current Claude reasons well by default and can think internally, so use structured reasoning where it genuinely helps rather than as a reflex on every prompt. Older material over-applies "always think step by step."

---

## 5. Role and tone prompting

**What:** Tell the model to adopt a role ("You are a seasoned copy editor…") and/or specify the tone to use. Optionally tell it who the audience is.

**Why:** Priming a role shifts the style, vocabulary, and framing of the response, and can improve performance on domain tasks — the model leans on the relevant register. Audience context ("explaining to a skeptical CFO" vs. "to a 5-year-old") further shapes the output.

**When:** You want a particular voice or persona; you want domain-appropriate framing; a chatbot needs a consistent character. Note it can go in the system prompt (API) or at the top of the user message (chat).

**Example:**
- Plain: `In one sentence, what do you think about skateboarding?` → neutral.
- Role: system = `You are a cat.` → the same question yields a cat's-eye answer with matching tone.

Adding audience: `You are a cat talking to a crowd of skateboarders` produces something different again. Layering role + audience + tone gives fine control.

---

## 6. Output formatting and prefilling

**What:** Two linked moves. (a) Tell the model the exact output format you want (XML tags, JSON with named keys, a specific structure). (b) On the API, **prefill** the assistant turn — put the first token(s) of the desired output into the assistant message so the model continues from there.

**Why:** Specifying format makes output parseable and consistent. Prefilling is a strong steering move: it skips preamble entirely and near-forces a structure. Prefill with `<tag>` to force the model to continue inside that tag; prefill with `{` to push toward JSON. The model continues directly from the prefill, so it can't wander off into "Sure, here's…".

**When:** You need machine-parseable output, you want to eliminate preamble, you need format consistency across runs. **Prefill is API-only** — it requires putting content in the `assistant` role, which chat interfaces don't expose. For chat, rely on the format spec and an example instead.

**Example (API):**
```
User: Write a haiku about cats. Use JSON with keys "first_line", "second_line", "third_line".
Assistant (prefill): {
```
The model continues the JSON object from the open brace.

```
User: Write a haiku about cats. Put it in <haiku> tags.
Assistant (prefill): <haiku>
```

---

## 7. Reducing hallucination

**What:** A cluster of techniques to keep the model from inventing facts: (a) give it an explicit out, (b) require evidence before the answer, (c) ground it in provided text and place long documents before the question.

**Why:** Models default to being maximally helpful, which can mean confidently producing a plausible-but-false answer rather than admitting uncertainty. Permission to say "I don't know," and a requirement to cite supporting text first, both pull strongly toward accuracy. Putting a long source document *before* the question lets the model read it in context before it knows what's being asked, which reduces distraction-driven errors.

**When:** Factual Q&A, document-grounded answers, anything where a wrong answer is worse than no answer.

**Examples:**

Give an out:
- Weak: `Who is the heaviest hippo of all time?` → may invent a specific named hippo with stats.
- Better: `Who is the heaviest hippo of all time? Only answer if you know the answer with certainty.`

Evidence first (for document tasks):
```
<document>{LONG_DOCUMENT}</document>

Answer the question below using only the document above. First pull the exact quotes that are relevant into <evidence> tags. If there are no relevant quotes, say so. Then answer based only on that evidence.

<question>{QUESTION}</question>
```

Ordering: put the long document near the top and the question near the bottom — the model reads the source before it sees the task.

---

## 8. Prompt chaining

**What:** Break a complex job into a sequence of separate prompts, feeding each output into the next, instead of asking one prompt to do everything.

**Why:** Each step gets the model's full attention and a clean, simpler task, which improves reliability. It also makes the pipeline debuggable — you can see which stage failed and fix just that prompt. The pattern: do step one, extract its output (often from XML tags), substitute that into step two, and so on.

**When:** A task has distinct phases (e.g. extract → analyze → format; or draft → critique → revise), one mega-prompt is brittle, or you need to inspect intermediate results.

**Example shape:**
1. Prompt A extracts relevant quotes from a document into `<quotes>` tags.
2. You pull the quotes and pass them to Prompt B, which answers the question using only those quotes.
3. Optionally Prompt C reformats or fact-checks B's answer.

A common, powerful chain is self-correction: have the model answer, then a second prompt critiques that answer, then a third revises it based on the critique.

---

## 9. Giving the model tools

**What:** Let the model call external functions (search, a calculator, a database lookup, an API) by describing the available tools and having it emit a structured call, which your code executes and feeds back.

**Why:** It extends the model past its own knowledge and text-only abilities — real-time data, precise computation, actions in other systems. The model decides when a tool is needed and what arguments to pass.

**When:** The task needs current information, exact calculation, or to take actions the model can't do in text alone. This is an API/developer pattern; modern SDKs have first-class tool-use support, so prefer the official tool-use API over hand-rolled formats.

**Note:** Implementation specifics (schemas, the request/response loop) evolve — when building this for real, check the current tool-use documentation rather than relying on a fixed template here.
