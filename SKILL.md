---
name: gpt-pro-audit
description: "Use when the user asks to audit a plan, document, diff, website finding, or implementation proposal with the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option through Chrome - automatically packages codebase/project context ChatGPT cannot see, runs up to 5 review rounds until accepted, verifies the response, and applies only accepted findings."
---

# GPT Pro Audit

Use this skill to automatically get an external audit from the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option through the user's authenticated Chrome session.

## Prerequisites

- The user has a ChatGPT Pro account with access to ChatGPT GPT-5.5 Pro (Extended Thinking) or the strongest available Pro reasoning option.
- Chrome is installed and enabled in the Codex app, with the Chrome connector/plugin available to the agent.
- The user is already signed in to ChatGPT in that Chrome profile, or is ready to sign in before the audit starts.

**REQUIRED SUB-SKILL:** Use `Chrome:Chrome` for all Chrome browser work, or the local Chrome connector/skill with equivalent capabilities.

## Core Rule

Treat ChatGPT as an external reviewer, not an authority. ChatGPT does not know the user's codebase, local files, current branch, project constraints, or prior evidence unless you provide them. Automatically package enough context for a useful audit, then verify every important claim against code, local evidence, or primary docs before changing anything.

## When To Use

- User mentions `ChatGPT GPT-5.5 Pro (Extended Thinking)`, `GPT-5.5-Pro`, `GPT Pro`, `Extended Thinking`, `ChatGPT audit`, `final audit`, or asks to use `@Chrome` for a review.
- A plan or implementation needs an adversarial second opinion from the user's ChatGPT account.
- The artifact is long enough that normal in-thread review may miss context.
- The user wants a stronger model to audit Codex/Claude work without manually explaining the whole repo.

Never send credentials, API keys, tokens, auth cookies, private keys, seed phrases, unredacted secrets, or live production logs to ChatGPT.

For private, proprietary, customer, transcript, billing, or production-derived data, send only the minimum necessary excerpt after the user explicitly confirms what will be sent. Redact names, emails, account IDs, hostnames, file paths, and other identifying details unless they are necessary for the audit.

## Automatic Context Package

Before opening ChatGPT, automatically assemble one prompt with:

1. **Intent:** what the artifact is trying to achieve.
2. **Artifact:** full plan/doc/diff text, or sanitized repo-relative file paths plus pasted contents. Avoid absolute paths unless necessary; redact usernames, private repo names, client names, and local machine paths.
3. **Codebase context:** relevant repo structure, touched files, current branch/status when useful, related tests, key functions, public contracts, and neighboring patterns. Prefer concise excerpts over dumping unrelated files.
4. **Constraints:** product rules, non-goals, safety limits, localization/market assumptions, "do not change" boundaries.
5. **Evidence:** commands run, source links, browser findings, current metrics, or known blockers.
6. **Audit focus:** 5-10 specific risks to check.
7. **Required output format:** verdict, blockers, minor fixes, rejected/uncertain claims, final approval condition.

Do not send a naked plan or diff when repo context is available. If context cannot be gathered, explicitly tell ChatGPT what is missing and ask it to separate confirmed findings from assumptions.

Before submission, perform a sensitivity pass:

- replace secrets with `[REDACTED_SECRET]`
- prefer synthetic examples over real customer data
- strip auth headers, cookies, tokens, and environment variable values
- avoid sending raw production logs; send redacted excerpts only

If the artifact is a local file, try file upload first. If upload fails, paste the content into the prompt and tell the user the upload was blocked.

## Prompt Template

```text
You are an adversarial external reviewer using the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option.

You do not know this codebase except for the context below. Treat missing context as unknown, not as permission to assume.

Intent:
<what this plan/change is meant to achieve>

Artifact:
<full pasted content or clear file contents>

Codebase context:
<relevant repo structure, touched files, code excerpts, tests, contracts, and local patterns>

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

## Multi-Round Audit Loop

This is not a one-shot review. Continue rounds until ChatGPT accepts the revised plan or a real blocker prevents progress, with a hard cap of 5 audit rounds to control token use.

For each round:

1. Submit the current artifact plus codebase context.
2. Capture ChatGPT's verdict and findings.
3. Verify each important finding against local code, evidence, or primary docs.
4. Apply only findings that are valid and compatible with user constraints.
5. Prepare a follow-up prompt with the revised artifact, what changed, what was rejected, and any unresolved evidence gaps.
6. Ask ChatGPT to re-audit only the revised plan and prior blocking findings.

Stop only when one of these is true:

- ChatGPT returns `APPROVED`.
- ChatGPT returns `APPROVED WITH MINOR CHANGES` and the user accepts that as enough.
- Round 5 is complete. If the plan is still not accepted, report the remaining blockers and ask the user before spending more tokens on another round.
- A prerequisite, privacy issue, model-access issue, or unresolved factual gap blocks further audit.
- The user explicitly stops the loop.

Track round count and final acceptance status. Do not report the plan as accepted after a `REVISE` or `BLOCKED` verdict, including after the 5-round cap.

## Chrome Workflow

1. Read and follow `Chrome:Chrome`.
2. Confirm Chrome is available in the Codex app and the user's ChatGPT Pro account is signed in. If Chrome is unavailable or the user lacks ChatGPT Pro access, stop and report the missing prerequisite.
3. Connect to Chrome and open or reuse `https://chatgpt.com/`.
4. Select the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option for the user's account. Record the exact visible model and effort setting. If the requested model or effort is unavailable, report what is available and ask whether to continue.
5. Upload the artifact file when possible. If Chrome file upload fails, use the exact Chrome skill guidance for enabling file uploads, then fall back to pasted content if the user still wants the audit now.
6. Submit the context package. Wait for the model to finish; long Pro thinking is expected.
7. Run the multi-round audit loop until the revised plan is accepted, a stopping condition is reached, or 5 rounds have completed.
8. Extract the final response text and keep the ChatGPT conversation URL for local handoff. Do not paste the URL into public issues, PRs, logs, or docs unless the user asks and the conversation contains no sensitive content.
9. Before ending browser work, call `browser.tabs.finalize({ keep })`. Keep the ChatGPT tab as `deliverable` only when the conversation itself is useful to the user.

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
- number of audit rounds
- actual model/session used, if visible
- final ChatGPT verdict and whether the plan was accepted
- findings accepted and patched
- findings rejected or left unverified
- verification commands/checks run
- remaining blockers, including Chrome upload/auth/model-access issues

Keep it concise. Do not paste the full ChatGPT response unless the user asks.

## Common Mistakes

- Sending a plan without product constraints, causing generic advice.
- Stopping after one review even when ChatGPT returns `REVISE` or `BLOCKED`.
- Treating ChatGPT's current-events claims as verified.
- Forgetting to include the user's "do not change" or "no regression" boundaries.
- Letting file upload failure stop the audit when paste fallback is acceptable.
- Leaving Chrome tabs unmanaged after the audit.
- Applying every external suggestion without checking code reality.
