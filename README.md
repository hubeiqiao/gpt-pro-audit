# gpt-pro-audit

A reusable Codex/Claude skill for sending plans, diffs, documents, and implementation proposals to ChatGPT GPT-5.5 Pro (Extended Thinking) through the user's authenticated Chrome session for an external audit.

This skill sends selected content to ChatGPT/OpenAI through the user's authenticated account; do not use it for secrets or unredacted sensitive data.

The skill focuses on:

- packaging enough context for a useful external review
- using the Chrome plugin workflow safely
- avoiding secret/private-data leaks
- treating ChatGPT as an external reviewer, not an authority
- verifying external claims before applying changes
- reporting accepted, rejected, and unresolved findings clearly

## Files

- `SKILL.md` - the skill to install or copy into an agent skills folder.

## Usage

Install or copy `SKILL.md` into a skill folder named `gpt-pro-audit`, then invoke it when you want a ChatGPT GPT-5.5 Pro (Extended Thinking) audit through Chrome.

Example trigger:

```text
Use gpt-pro-audit to send this plan to ChatGPT GPT-5.5 Pro (Extended Thinking) for final audit.
```

## Requirements

- A working Chrome plugin/connector in the agent environment.
- An authenticated ChatGPT session with access to ChatGPT GPT-5.5 Pro (Extended Thinking).
- User approval before sending any sensitive, private, or proprietary data.

## License

No license has been selected yet. Add a license before encouraging broad public reuse.
