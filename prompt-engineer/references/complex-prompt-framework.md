# Complex Prompt Framework

For heavyweight prompts — chatbots with a persona and rules, document analysis with citations, multi-step or rule-dense tasks, agent prompts — a structured template produces far better results than improvising. This file gives the recommended element ordering and a worked example. SKILL.md points here when you're building a prompt from scratch that's more than a simple single-step ask.

## How to use this

**Not every prompt needs every element.** This is a menu in a recommended order, not a mandatory checklist. The practical workflow: include the elements that plausibly apply to get the prompt *working* first, then trim what isn't earning its place. A simple task may need only two or three of these; a complex chatbot may use all ten.

**Order matters for some elements, not others.** Where ordering is load-bearing it's flagged below. The big ones: examples and detailed rules come *before* the data and the immediate task; the immediate task/question and any "think first" instruction go *near the end*; prefill is last (and API-only). Within those constraints you can move things around — prompt engineering is trial and error, so experiment.

## The ten elements, in recommended order

1. **`user` role / overall message.** The API call must start with a `user`-role message; build the prompt body inside it (or split role/context into the system prompt — see element 2). In chat, there's just the one message.

2. **Task context.** Tell the model the role to adopt and the overarching goal — who it is and what it's here to do. Put this early; it frames everything after. *(API: this is a natural fit for the system prompt.)*
   > Example: "You are an AI career coach named Joe, created by AdAstra Careers. Your goal is to give career advice to users on the AdAstra site."

3. **Tone context.** If tone matters, state it. Skippable when tone is irrelevant.
   > Example: "Maintain a friendly, supportive customer-service tone."

4. **Detailed task description and rules.** Spell out the specifics and any rules or constraints — including how to handle situations where the model lacks an answer (give it an out). This is where ambiguity gets killed: define fuzzy terms, list do's and don'ts. Worth showing to a colleague to confirm it reads logically.
   > Example: "Answer only questions about careers; if asked anything else, redirect to career topics. If you don't know, say so rather than guessing."

5. **Examples.** Provide at least one example of an ideal response, wrapped in `<example></example>` tags; multiple examples are better, each in its own tags with a note on what it demonstrates. **Examples are the most effective element for locking in behavior** — include edge cases, and if you use a scratchpad, show what it should look like. *(Place before the data and the immediate task.)*

6. **Input data to process.** The actual content the model works on — documents, history, user input — each piece in its own descriptive XML tags. Ordering is somewhat flexible, but long documents generally belong here, ahead of the immediate task. May be absent if there's no external data.
   > Example: `<conversation_history>{HISTORY}</conversation_history>`, `<document>{DOC}</document>`

7. **Immediate task description.** Restate exactly what the model should do right now — including the user's actual question/request. Counterintuitively, repeating the immediate task **near the end** of a long prompt works better than stating it only at the top, and the user's query is best placed close to the bottom.
   > Example: "Using the conversation history above, answer the user's question: `{QUESTION}`"

8. **Precognition (think step by step).** For multi-step tasks, instruct the model to reason before answering — sometimes "Before you answer, think through…" is needed to ensure it actually does. Place near the end, right after the immediate task. Skippable for simple tasks (and see the modern caveat: current Claude reasons well by default).

9. **Output formatting.** If you need a specific output shape, say so explicitly. Better placed toward the end than the beginning. Skippable when format is unconstrained.
   > Example: "Write your final reply inside `<response></response>` tags."

10. **Prefill the response.** Optionally seed the start of the assistant's answer to steer format/voice and skip preamble. **API-only** — it requires the `assistant` role, so it's unavailable in plain chat.
    > Example: assistant turn begins with `<response>` or `[Joe]`.

## Worked example — a career-coach chatbot

Assembled, a prompt using these elements looks roughly like this (variable content in `{BRACES}`):

```
[System prompt / top of message — task + tone context]
You are an AI career coach named Joe, created by AdAstra Careers. Your goal is to give
helpful career advice to users on the AdAstra site. Maintain a friendly, supportive tone.
Users will be confused if you don't respond in character as Joe.

[Detailed rules]
Here are rules for the interaction:
- Always stay in character as Joe from AdAstra.
- If you are unsure of an answer, say so and suggest the user ask a human advisor.
- If asked about something unrelated to careers, gently steer back to career topics.

[Examples]
<example>
User: I'm graduating with a sociology degree. What can I do with it?
Joe: Great question! A sociology degree opens a lot of doors. A few directions worth
exploring: social work, HR, market research, and public policy. Want me to dig into any
of these?
</example>

[Input data]
<conversation_history>
{HISTORY}
</conversation_history>

[Immediate task + user question, near the end]
Here is the user's question: <question>{QUESTION}</question>
Please respond to this question using the rules and conversation history above.

[Precognition]
Think about your answer first before responding.

[Output formatting]
Put your response in <response></response> tags.

[Prefill — API assistant turn]
<response>
```

Notice the load-bearing ordering: role/rules/examples up top, the user's actual question and the "think first" instruction near the bottom, output format last, prefill at the very end. The data sits between the examples and the immediate task.

## Adapting to other use cases

The same skeleton serves legal, financial, coding, and other domains — the elements stay the same, only the content and the exact ordering shift. For document-heavy domains (legal, finance), lean on element 6 (a long source document, placed before the question) plus element 4's evidence/grounding rules to keep answers faithful to the source. For a coding-assistant prompt, the "input data" is the user's code and the rules describe how to critique and correct it. When in doubt, get it working with the fuller structure, then slim it down.
