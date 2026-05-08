# gpt-pro-audit

[![install with skills.sh](https://img.shields.io/badge/skills.sh-install%20gpt--pro--audit-111111)](https://skills.sh/hubeiqiao/gpt-pro-audit/gpt-pro-audit)

## About

A reusable Codex/Claude skill that automatically packages codebase context and sends plans, diffs, documents, and implementation proposals to the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option through the user's authenticated Chrome session for an external audit.

This skill sends selected content to ChatGPT/OpenAI through the user's authenticated account; do not use it for secrets or unredacted sensitive data.

Invoking the skill starts the audit automatically for the selected non-sensitive artifact. It does not require a separate "go" confirmation unless sensitive/private data, a broader-than-requested payload, or a model/access mismatch is involved.

ChatGPT does not know your codebase, local files, current branch, or project constraints. This skill is designed to gather and provide that context automatically so the audit is grounded instead of generic.

![gpt-pro-audit overview](assets/gpt-pro-audit-hero.jpg)

The skill focuses on:

- packaging enough context for a useful external review
- starting automatically after invocation for non-sensitive artifacts
- automatically giving ChatGPT the repo/project context it cannot see
- using the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option
- running up to 5 review rounds until GPT Pro accepts the revised plan
- actively revising the artifact between rounds instead of just relaying feedback
- resuming smoothly after browser slowdowns or context compaction with a local audit state file
- disclosing whether the audit will create a normal ChatGPT history record before submission
- avoiding repeated upload retries and using targeted follow-up prompts for later rounds
- using the Chrome plugin workflow safely
- avoiding secret/private-data leaks
- treating ChatGPT as an external reviewer, not an authority
- verifying external claims before applying changes
- reporting accepted, rejected, and unresolved findings clearly

## Files

- `SKILL.md` - the skill to install or copy into an agent skills folder.

## Usage

Install from skills.sh-compatible source:

```bash
npx skills add hubeiqiao/gpt-pro-audit
```

Or copy `SKILL.md` into a skill folder named `gpt-pro-audit`, then invoke it when you want a ChatGPT GPT-5.5 Pro (Extended Thinking) audit through Chrome.

Example trigger:

```text
Use gpt-pro-audit to send this plan to ChatGPT GPT-5.5 Pro (Extended Thinking) for final audit.
```

## Requirements

- A ChatGPT Pro account with access to ChatGPT GPT-5.5 Pro (Extended Thinking), or the strongest available Pro reasoning option.
- Chrome installed and enabled in the Codex app, with the Chrome connector/plugin available to the agent.
- An authenticated ChatGPT session in that Chrome profile.
- User approval only when sensitive/private/proprietary data, a broader-than-requested payload, or a model/access mismatch is involved.
- Awareness that, unless Temporary Chat or no-history mode is used, the audit appears in the user's normal ChatGPT history.

## License

MIT. See [LICENSE](LICENSE).
