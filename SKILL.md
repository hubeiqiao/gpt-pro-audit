---
name: gpt-pro-audit
description: "Use when the user asks to send a plan, document, diff, website finding, or implementation proposal to ChatGPT GPT-5.5 Pro (Extended Thinking) through Chrome for an external audit - packages context, runs the browser workflow, verifies the response, and applies only accepted findings."
---

# GPT Pro Audit

Use this skill to get an external audit from the strongest available ChatGPT reasoning model through the user's authenticated Chrome session, preferably ChatGPT GPT-5.5 Pro (Extended Thinking) when available.

**REQUIRED SUB-SKILL:** Use `Chrome:Chrome` for all Chrome browser work, or the local Chrome connector/skill with equivalent capabilities.

## Core Rule

Treat ChatGPT as an external reviewer, not an authority. Package enough context for a useful audit, then verify every important claim against code, local evidence, or primary docs before changing anything.

## When To Use

- User mentions `ChatGPT GPT-5.5 Pro (Extended Thinking)`, `GPT-5.5-Pro`, `GPT Pro`, `Extended Thinking`, `ChatGPT audit`, `final audit`, or asks to use `@Chrome` for a review.
- A plan or implementation needs an adversarial second opinion from the user's ChatGPT account.
- The artifact is long enough that normal in-thread review may miss context.

Never send credentials, API keys, tokens, auth cookies, private keys, seed phrases, unredacted secrets, or live production logs to ChatGPT.

For private, proprietary, customer, transcript, billing, or production-derived data, send only the minimum necessary excerpt after the user explicitly confirms what will be sent. Redact names, emails, account IDs, hostnames, file paths, and other identifying details unless they are necessary for the audit.

## Context Package

Before opening ChatGPT, assemble one prompt with:

1. **Intent:** what the artifact is trying to achieve.
2. **Artifact:** full plan/doc/diff text, or sanitized repo-relative file paths plus pasted contents. Avoid absolute paths unless necessary; redact usernames, private repo names, client names, and local machine paths.
3. **Constraints:** product rules, non-goals, safety limits, localization/market assumptions, "do not change" boundaries.
4. **Evidence:** commands run, source links, browser findings, current metrics, or known blockers.
5. **Audit focus:** 5-10 specific risks to check.
6. **Required output format:** verdict, blockers, minor fixes, rejected/uncertain claims, final approval condition.

Before submission, perform a sensitivity pass:

- replace secrets with `[REDACTED_SECRET]`
- prefer synthetic examples over real customer data
- strip auth headers, cookies, tokens, and environment variable values
- avoid sending raw production logs; send redacted excerpts only

If the artifact is a local file, try file upload first. If upload fails, paste the content into the prompt and tell the user the upload was blocked.

## Prompt Template

```text
You are an adversarial external reviewer using ChatGPT GPT-5.5 Pro (Extended Thinking).

Intent:
<what this plan/change is meant to achieve>

Artifact:
<full pasted content or clear file contents>

Constraints and non-goals:
<repo/product/user constraints>

Evidence already gathered:
<commands, docs, metrics, screenshots, source links>

Audit focus:
- Correctness: will this achieve the goal?
- Regression risk: what could break?
- Missing steps: what is not executable enough?
- Security/privacy/data risk: what must not be sent, exposed, indexed, or logged?
- Current best practice: identify stale assumptions and cite primary sources where possible.

Output format:
1. Verdict: APPROVED / APPROVED WITH MINOR CHANGES / REVISE / BLOCKED
2. Blocking findings, each with concrete section/file references and fix
3. Minor findings
4. Claims you are uncertain about or that require primary-source verification
5. Exact approval condition after fixes
```

## Chrome Workflow

1. Read and follow `Chrome:Chrome`.
2. Connect to Chrome and open or reuse `https://chatgpt.com/`.
3. Confirm the selected model is the strongest available ChatGPT reasoning option for the user's account, preferably ChatGPT GPT-5.5 Pro (Extended Thinking) when available. Record the exact visible model and effort setting. If the requested model or effort is unavailable, report what is available and ask whether to continue.
4. Upload the artifact file when possible. If Chrome file upload fails, use the exact Chrome skill guidance for enabling file uploads, then fall back to pasted content if the user still wants the audit now.
5. Submit the context package. Wait for the model to finish; long Pro thinking is expected.
6. Extract the full response text and keep the ChatGPT conversation URL for local handoff. Do not paste the URL into public issues, PRs, logs, or docs unless the user asks and the conversation contains no sensitive content.
7. Before ending browser work, call `browser.tabs.finalize({ keep })`. Keep the ChatGPT tab as `deliverable` only when the conversation itself is useful to the user.

## Review Handling

For each ChatGPT finding:

- Accept only findings that are technically correct for this codebase or plan.
- Verify current/search-engine/legal/API claims with primary sources before repeating them as facts.
- If a finding conflicts with user constraints, reject it and explain why.
- Patch the artifact only after deciding the finding is valid.
- Re-audit or targeted-check the patched sections when findings were blocking.

## Final Response

Report:

- artifact reviewed
- actual model/session used, if visible
- ChatGPT verdict
- findings accepted and patched
- findings rejected or left unverified
- verification commands/checks run
- remaining blockers, including Chrome upload/auth/model-access issues

Keep it concise. Do not paste the full ChatGPT response unless the user asks.

## Common Mistakes

- Sending a plan without product constraints, causing generic advice.
- Treating ChatGPT's current-events claims as verified.
- Forgetting to include the user's "do not change" or "no regression" boundaries.
- Letting file upload failure stop the audit when paste fallback is acceptable.
- Leaving Chrome tabs unmanaged after the audit.
- Applying every external suggestion without checking code reality.
