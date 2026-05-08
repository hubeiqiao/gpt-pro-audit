---
name: gpt-pro-audit
description: "Use when the user asks to audit a plan, document, diff, website finding, or implementation proposal with the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option through Chrome - starts automatically after invocation, packages codebase/project context ChatGPT cannot see, runs up to 5 review rounds until accepted, verifies the response, and applies only accepted findings."
---

# GPT Pro Audit

Use this skill to automatically get an external audit from the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option through the user's authenticated Chrome session.

## Prerequisites

- The user has a ChatGPT Pro account with access to ChatGPT GPT-5.5 Pro (Extended Thinking) or the strongest available Pro reasoning option.
- Chrome is installed and enabled in the Codex app, with the Chrome connector/plugin available to the agent.
- The user is already signed in to ChatGPT in that Chrome profile, or is ready to sign in before the audit starts.
- Unless Temporary Chat or chat history controls are explicitly enabled in ChatGPT, the audit will create a normal ChatGPT conversation in the user's account history.

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

Generate a unique review ID before preparing the audit so concurrent reviews do not collide:

```bash
REVIEW_ID=$(uuidgen | tr '[:upper:]' '[:lower:]' | head -c 8)
```

When local files are useful, use session-scoped temp paths such as `/tmp/gpt-pro-audit-${REVIEW_ID}-context.md`, `/tmp/gpt-pro-audit-${REVIEW_ID}-state.md`, and `/tmp/gpt-pro-audit-${REVIEW_ID}-round-N.md`. If there is no plan, diff, document, or artifact in the current conversation, ask the user what they want reviewed before opening ChatGPT.

Maintain `/tmp/gpt-pro-audit-${REVIEW_ID}-state.md` throughout the run with: artifact path/name, sanitized payload summary, approximate size, ChatGPT conversation URL, visible model, round count, each verdict, accepted changes, rejected findings, unresolved blockers, and whether ChatGPT history or Temporary Chat was used. Update it before submission and after every round so the loop can resume after context compaction or browser slowdown.

Before opening ChatGPT, automatically assemble one prompt with:

1. **Intent:** what the artifact is trying to achieve.
2. **Artifact:** full plan/doc/diff text, or sanitized repo-relative file paths plus pasted contents. Avoid absolute paths unless necessary; redact usernames, private repo names, client names, and local machine paths.
3. **Codebase context:** relevant repo structure, touched files, current branch/status when useful, related tests, key functions, public contracts, and neighboring patterns. Prefer concise excerpts over dumping unrelated files.
4. **Constraints:** product rules, non-goals, safety limits, localization/market assumptions, "do not change" boundaries.
5. **Evidence:** commands run, source links, browser findings, current metrics, or known blockers.
6. **Audit focus:** 5-10 specific risks to check.
7. **Required output format:** exact verdict line, blockers, minor fixes, rejected/uncertain claims, final approval condition.

Do not send a naked plan or diff when repo context is available. If context cannot be gathered, explicitly tell ChatGPT what is missing and ask it to separate confirmed findings from assumptions.

Before submission, perform a sensitivity pass:

- replace secrets with `[REDACTED_SECRET]`
- prefer synthetic examples over real customer data
- strip auth headers, cookies, tokens, and environment variable values
- avoid sending raw production logs; send redacted excerpts only

For local files, choose the transport using the Transport Strategy below and tell the user when file upload is blocked or paste fallback will be used.

## Invocation Consent

Invoking this skill is consent to start the audit for the selected non-sensitive artifact. Do not ask the user to say "go", "proceed", or "confirm" before the first submission just because ChatGPT will receive the plan/doc/diff.

Before the first submission, send a short disclosure/status update, then continue automatically. Include:

- what artifact names or sections will be sent
- approximate payload size and whether it is full, partial, or targeted
- what sensitive data categories were excluded
- the visible ChatGPT model/effort
- whether the audit will create a normal ChatGPT history record or use Temporary Chat/no-history mode

Stop and ask for explicit approval only if one of these is true:

- the payload includes private, proprietary, customer, transcript, billing, production-derived, or otherwise sensitive data
- a secret, credential, auth token, cookie, private key, or live production log may be included even after redaction
- the payload is broader than the artifact the user asked to audit
- the requested ChatGPT model/effort is unavailable and continuing would use a weaker or different mode
- a very large full-document paste is needed because upload failed and a targeted package would be materially worse

Do not ask again for later rounds unless a new artifact or new sensitive data category would be sent.

## Transport Strategy

Choose the lowest-friction transport that still gives ChatGPT enough context:

1. Estimate payload size before using Chrome.
2. Prefer file upload for large local artifacts. If upload fails once with a permissions/browser error, record the blocker and switch strategy; do not repeatedly retry hidden inputs, chooser paths, and alternate buttons.
3. Use paste fallback for small or moderate payloads. For large paste fallback, keep the first prompt compact and focused: artifact summary, relevant excerpts, repo context, constraints, and exact questions. Ask the user before pasting a very large full document.
4. For follow-up rounds after `VERDICT: REVISE`, send only the revised sections, prior blocking findings, accepted/rejected change list, and unresolved gaps unless ChatGPT explicitly needs the whole artifact again.
5. Do not send tiny "test" messages to validate the composer. Verify controls by page state and accessibility tree only.
6. If a browser action times out after a submit attempt, inspect the conversation URL, composer state, and latest assistant message before retrying. Never blindly submit the same long prompt twice.

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
1. Verdict line: end with exactly `VERDICT: APPROVED`, `VERDICT: REVISE`, or `VERDICT: BLOCKED`
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
4. Actively revise the plan or artifact based on valid findings. Do not merely pass messages between the user and ChatGPT.
5. Skip any revision that contradicts explicit user requirements, and note why it was skipped.
6. Summarize revisions for the user, one bullet per accepted finding.
7. Prepare a follow-up prompt with the revised artifact, what changed, what was rejected, and any unresolved evidence gaps.
8. Continue in the same ChatGPT conversation so prior review context is preserved.
9. Ask ChatGPT to re-audit only the revised plan and prior blocking findings.
10. Update the state file before moving to the next round or reporting status.

If the ChatGPT tab/session loses context, start a fresh conversation only after including: original intent, latest revised artifact, prior round verdicts, accepted changes, rejected findings, and remaining blockers.

If the local agent context is compacted or interrupted, resume from `/tmp/gpt-pro-audit-${REVIEW_ID}-state.md` before taking further action. If the `REVIEW_ID` is unknown, search `/tmp/gpt-pro-audit-*-state.md` by artifact name and latest timestamp.

Stop only when one of these is true:

- ChatGPT returns exactly `VERDICT: APPROVED`.
- ChatGPT gives no exact verdict but has only positive feedback and no actionable findings.
- Round 5 is complete. If the plan is still not accepted, report the remaining blockers and ask the user before spending more tokens on another round.
- A prerequisite, privacy issue, model-access issue, or unresolved factual gap blocks further audit.
- The user explicitly stops the loop.

Track round count and final acceptance status. Do not report the plan as accepted after a `REVISE` or `BLOCKED` verdict, including after the 5-round cap.

After `VERDICT: REVISE` with valid blocking findings, patching the artifact is not completion. The next required action is another audit round or a targeted re-check of the revised sections. If Chrome/browser access fails before that re-check, report the audit as incomplete rather than done.

Present each round in this shape:

```text
## GPT Pro Audit — Round N

[ChatGPT feedback summary or excerpt]

### Revisions
- [What changed and why]

VERDICT: APPROVED
# or: VERDICT: REVISE / VERDICT: BLOCKED
```

## Chrome Workflow

1. Read and follow `Chrome:Chrome`.
2. Confirm Chrome is available in the Codex app and the user's ChatGPT Pro account is signed in. If Chrome is unavailable or the user lacks ChatGPT Pro access, stop and report the missing prerequisite.
3. Connect to Chrome and open or reuse `https://chatgpt.com/`.
4. If Temporary Chat/no-history mode is available and appropriate, prefer it for sensitive audits; otherwise tell the user the audit will be visible in their normal ChatGPT history before first submission.
5. Select the best available ChatGPT GPT-5.5 Pro (Extended Thinking) option for the user's account. Record the exact visible model and effort setting. If the requested model or effort is unavailable, report what is available and ask whether to continue.
6. Upload or paste according to the Transport Strategy.
7. Submit the context package. Wait for the model to finish; long Pro thinking is expected. If ChatGPT remains in a finalizing/thinking state, keep waiting or ask it to continue in the same conversation; do not resubmit the whole payload.
8. Run the multi-round audit loop until the revised plan is accepted, a stopping condition is reached, or 5 rounds have completed.
9. Extract the final response text and keep the ChatGPT conversation URL for local handoff. Do not paste the URL into public issues, PRs, logs, or docs unless the user asks and the conversation contains no sensitive content.
10. Before ending browser work, call `browser.tabs.finalize({ keep })`. Keep the ChatGPT tab as `deliverable` only when the conversation itself is useful to the user.

## Cleanup

Remove session-scoped temp files after the final result:

```bash
rm -f /tmp/gpt-pro-audit-${REVIEW_ID}-context.md /tmp/gpt-pro-audit-${REVIEW_ID}-state.md /tmp/gpt-pro-audit-${REVIEW_ID}-round-*.md
```

## Review Handling

For each ChatGPT finding:

- Accept only findings that are technically correct for this codebase or plan.
- Verify current/search-engine/legal/API claims with primary sources before repeating them as facts.
- If a finding conflicts with user constraints, reject it and explain why.
- Patch the artifact only after deciding the finding is valid.
- Re-audit or targeted-check the patched sections when findings were blocking.
- For untracked files, do not rely on `git diff -- path`; use `git status --short`, read the file directly, and compare against the saved state or prior temp copy.

## Final Response

Report:

- artifact reviewed
- number of audit rounds
- actual model/session used, if visible
- whether a normal ChatGPT history record was created or Temporary Chat/no-history was used
- final ChatGPT verdict and whether the plan was accepted
- findings accepted and patched
- findings rejected or left unverified
- verification commands/checks run
- remaining blockers, including Chrome upload/auth/model-access issues

Keep it concise. Do not paste the full ChatGPT response unless the user asks.

## Common Mistakes

- Sending a plan without product constraints, causing generic advice.
- Passing messages between the agent and ChatGPT without actively revising the artifact.
- Stopping after one review even when ChatGPT returns `REVISE` or `BLOCKED`.
- Calling the audit done after patching a `REVISE` finding without sending the revised sections back.
- Surprising the user with a normal ChatGPT history record instead of disclosing it before submission.
- Retrying file upload paths repeatedly after the first browser permission failure.
- Blindly resubmitting a long prompt after a Chrome timeout.
- Starting a fresh ChatGPT conversation for later rounds without including prior verdicts and changes.
- Treating ChatGPT's current-events claims as verified.
- Forgetting to include the user's "do not change" or "no regression" boundaries.
- Letting file upload failure stop the audit when paste fallback is acceptable.
- Leaving Chrome tabs unmanaged after the audit.
- Applying every external suggestion without checking code reality.
