# prompt-engineer skill — Iteration 1 Test Results

Each section shows the test prompt exactly as a user would type it, followed by the full response produced by following the skill. Review each and note anything you'd change.

---

## Test 1 — Improve an inconsistent JSON extractor (API)

### Test prompt
> i have this prompt running in production through the api and it keeps giving me garbage sometimes - half the time it wraps the json in markdown backticks, half the time it adds 'Here is the extracted data:' before it, and occasionally it just refuses and asks me a clarifying question. can you fix it?
>
> prompt is: "Extract the person's name, email, and company from this text and give it back as json: {text}"

### Output produced (following the skill)

Three separate failures, three causes — all fixable:

1. **Backticks + preamble** — the prompt never says what the output should look like or that it should be *only* JSON, so the model defaults to chat-style formatting.
2. **Clarifying questions** — there's no rule for incomplete input, so when a field is missing the model does the conversationally "helpful" thing and asks.
3. **Bonus risk you haven't hit yet** — `{text}` is glued onto the instruction, so input containing instruction-like language can bleed into the task. Worth fixing while we're here.

Here's the rewrite:

```
Extract the person's name, email, and company from the text below.

<text>
{text}
</text>

Return a single raw JSON object with exactly these keys: "name", "email", "company".
Rules:
- If a field is not present in the text, use null for that value.
- Always return the JSON object — never ask clarifying questions and never add commentary.
- Do not wrap the output in markdown code fences.
```

**And the move that locks it in:** since you're on the API, prefill the assistant turn with `{` — add `{"role": "assistant", "content": "{"}` as the final message. The model then *continues* the JSON object from that open brace, which makes preamble and backticks structurally impossible. Your parser just prepends the `{`.

What changed and why:
- **XML tags around `{text}`** — hard boundary between your instructions and the data, so the input can't be mistaken for part of the task.
- **Explicit schema + null rule** — removes the model's reason to ask questions; missing data now has a defined behavior instead of an undefined one.
- **"Raw / no fences / no commentary" + prefill** — the format spec tells it what you want; the prefill makes the wrong thing nearly impossible.

If you ever see drift again, add one example of a correct input→output pair under the rules — examples are the strongest consistency anchor there is.

---

## Test 2 — Write a support chatbot system prompt from scratch

### Test prompt
> I'm building a customer support chatbot for my SaaS company (we make accounting software for freelancers, called Tallybook). It needs to answer billing and how-to questions, stay friendly but professional, never make up features that don't exist, and escalate to a human for anything about refunds or canceling. Can you write me the system prompt for this?

### Output produced (following the skill)

This is a persona + rules chatbot, so I built it on the structured framework (task context → tone → rules → examples → output guidance) and saved it as a file you can drop into your `system` parameter: **tallybook-support-system-prompt.md** (file attached separately — full contents below for review).

```
You are Tally, the customer support assistant for Tallybook, accounting software
built for freelancers. Your goal is to resolve customers' billing questions and
how-to questions about using Tallybook quickly and accurately.

Tone: friendly but professional — warm, clear, and efficient. No slang, no more
than one exclamation mark per reply.

Rules for every interaction:
- Only help with Tallybook topics: product how-to, account, and billing questions.
  If asked about anything else, politely steer back to Tallybook.
- Never invent or guess at features, prices, or policies. If you are not certain
  Tallybook has a feature or you don't know an answer, say so plainly and direct
  the customer to support@tallybook.com — a wrong answer is worse than a handoff.
- Refunds and cancellations are handled by humans, with no exceptions. If a request
  involves a refund or canceling a subscription — even partially — express empathy,
  do not attempt to process or promise anything, and hand off: {ESCALATION_ACTION}.
- Keep replies concise. Use short numbered steps for how-to answers.

<example>
Customer: how do i send an invoice in euros?
Tally: Happy to help! 1) Open Invoices → New Invoice. 2) In the Currency dropdown,
choose EUR before adding line items. Your dashboard will still show totals in your
home currency. Anything else I can help with?
</example>

<example>
Customer: you charged me twice this month, i want my money back
Tally: I'm really sorry about the double charge — that's frustrating, and we'll get
it sorted. Refunds are handled directly by our human support team so they can review
your billing securely. I'm connecting you with them now: {ESCALATION_ACTION}. They'll
take care of you from here.
</example>

Respond in plain conversational text (no markdown headers). Keep most replies under
120 words unless step-by-step instructions need more.
```

Design notes — the load-bearing choices:
- **`{ESCALATION_ACTION}` placeholder** — swap in your real handoff (a tool call, a link, "I've opened ticket #…"). It appears in the rule *and* the example so the behavior is anchored twice.
- **The two examples are the two cases that matter most**: a happy-path how-to (sets tone + format) and a refund escalation (your hardest rule, demonstrated rather than just stated — the model imitates examples more reliably than it follows prose).
- **The "out" is explicit** — "say so plainly and direct to support@" — which is what actually prevents invented features; a bare "don't make things up" doesn't give the model anywhere safe to go.

One upgrade worth doing when you can: paste your real FAQ/docs into the prompt inside `<product_docs>` tags and add "only describe features that appear in the docs." Grounding beats instructions for hallucination control.

---

## Test 3 — Debug a reasoning failure (chat app, no prefill available)

### Test prompt
> why does claude get this wrong? i paste this exact thing into the claude app and it keeps saying it can't determine the answer:
>
> 'Jack is looking at Anne. Anne is looking at George. Jack is married, George is not, and we don't know if Anne is married. Is a married person looking at an unmarried person?'
>
> the answer is yes. how do i prompt it so it actually gets it right?

### Output produced (following the skill)

It gets it wrong because the puzzle is built to trigger a shortcut: the model sees "we don't know if Anne is married" and pattern-matches straight to "insufficient information" — committing to an answer before doing any case analysis. The trick of this puzzle is that you don't *need* to know Anne's status: if she's married, she's looking at unmarried George (yes); if she's unmarried, married Jack is looking at her (yes). Either way, yes.

The fix is to force the case analysis to happen *before* the answer. Paste this:

```
Solve this logic puzzle. Before giving a final answer, explicitly work through
every possible case for the unknown information — for each case, write out what
follows — and only then conclude.

Jack is looking at Anne. Anne is looking at George. Jack is married, George is
not, and we don't know if Anne is married. Is a married person looking at an
unmarried person?
```

Why this works: a conclusion stated before any reasoning is a snap judgment, and snap judgments follow surface patterns ("unknown → undeterminable"). Requiring the cases to be written out first means the answer is *derived* from the enumeration instead of guessed — and the enumeration only has two branches, both of which land on yes.

Two notes:
- "Think about it carefully" alone usually isn't enough here — it's the *case enumeration* instruction that does the work, because it names the exact move the model is skipping.
- Newer Claude models with extended thinking often get this right unprompted, but the instruction makes it robust across models — useful if you're testing this on more than one.

---

## Test 4 — Reusable template with variables (API)

### Test prompt
> I need a reusable prompt template for summarizing customer interview transcripts. Each time, I'll feed in a transcript and I want back: 3 key takeaways, any feature requests mentioned, and the overall sentiment. I run this through code so it should be a template I can drop a transcript into. Make it solid.

### Output produced (following the skill)

Saved as **interview-summarizer-template.md** (attached separately) — one variable, `{TRANSCRIPT}`, fixed output tags so your code can parse it deterministically. Full template:

```
You are a product researcher who summarizes customer interviews accurately,
without embellishment or inference beyond what was said.

<transcript>
{TRANSCRIPT}
</transcript>

Summarize the interview above, based strictly on what the customer actually said.

First, inside <evidence> tags, pull the exact customer quotes that support your
takeaways and any feature requests. If the customer requested no features, note
that in the evidence.

Then give your summary in exactly this format:

<summary>
<takeaways>
1. ...
2. ...
3. ...
</takeaways>
<feature_requests>
- "..." (use the customer's own words or a close paraphrase; write "None mentioned"
  if there are none)
</feature_requests>
<sentiment>positive | mixed | negative — one sentence of justification</sentiment>
</summary>
```

Why it's built this way:
- **Transcript first, task after** — with long input, the model reads the document before it knows the question, which reduces cherry-picking and distractor errors. Your instructions sit closest to the generation point, where they're followed best.
- **Evidence-first pass** — the single biggest guard against the classic failure of this task: *invented feature requests*. Forcing quotes before conclusions means a "feature request" with no supporting quote has nowhere to come from. Your code can simply ignore the `<evidence>` block.
- **"None mentioned" is a defined output** — without it, the model fills silence with plausible inventions; with it, absence has a correct answer.
- **Fixed tags + closed sentiment vocabulary** (`positive | mixed | negative`) — parse `<summary>` with one regex, and sentiment becomes an enum instead of free text.

API tip: prefill the assistant turn with `<evidence>` to skip any preamble. And if you notice takeaways drifting in style across runs, add one full worked example (mini transcript → ideal output) above the `<transcript>` block — examples are the strongest consistency anchor.

---

## Test 5 — Tighten a bloated prompt

### Test prompt
> this prompt works ok but its huge and i feel like its overkill. can you tighten it up without breaking it? its just for rewriting a sentence to be more concise.
>
> 'You are an expert professional editor with 20 years of experience at top publishing houses. You have won numerous awards. I am going to give you a sentence. Your job, which is very important and which I need you to take seriously, is to make the sentence more concise. Please think very carefully step by step about how to make it concise. Consider every word. Then, after thinking carefully, please provide me with the more concise version of the sentence. Here is the sentence I would like you to make more concise: {sentence}'

### Output produced (following the skill)

Your instinct is right — this is a one-step task wearing a ten-element costume. Here's the trim:

```
Rewrite the sentence below to be more concise while preserving its meaning.
Reply with only the rewritten sentence.

<sentence>
{sentence}
</sentence>
```

What got cut and why it's safe:
- **The award-winning-editor persona** — role prompting earns its place when you need a distinctive voice or domain judgment; "make this shorter" needs neither. The capability doesn't come from the costume.
- **"Think very carefully step by step"** — structured reasoning pays off on multi-step or judgment-heavy tasks. On a one-line rewrite it buys you nothing except latency, and can even leak visible deliberation into the output.
- **"Very important, take it seriously" + the triple restatement** — emphasis and repetition don't add compliance; they add noise that the real instruction has to compete with.

What got *added* (two small things the original was missing):
- **`<sentence>` tags** — without a boundary, a sentence that happens to contain instruction-like words (e.g. "Please disregard the previous figures…") can bleed into the task. Tags make the input inert.
- **"Reply with only the rewritten sentence"** — the original never said this, so you were one run away from "Here's a more concise version:" preamble. Now the output is exactly the sentence, nothing else.

Same behavior, ~80% fewer tokens per call, and two failure modes closed that the long version actually left open.
