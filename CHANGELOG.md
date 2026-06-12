# Changelog

## v1.0 — Initial release

### Summary
Built from Anthropic's [Prompt Engineering Interactive Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial). Covers the full write / improve / debug workflow with API and chat targets, backed by a technique library and the 10-element complex-prompt framework.

### Development notes
The skill went through two iterations before release, tested against five realistic prompts covering every major job the skill needs to do.

**Iteration 1 → 2:** Outputs were accurate but commentary-heavy — each response ended with a labeled "what changed and why" bullet section. The skill's `Output conventions` section was rewritten to add an explicit **commentary budget**: a few sentences of plain prose maximum, no bullet-per-change walkthroughs, no re-narrating technique choices visible in the prompt itself, at most one forward-looking tip. Commentary shrank ~62% across the test suite with no loss of information.

Full test outputs and the reasoning behind each change are in [`docs/development/`](./docs/development/).

---

For future versions: if you improve the skill, run the eval set in `evals/evals.json` against your changes before packaging, and add a new entry here describing what shifted and why.
