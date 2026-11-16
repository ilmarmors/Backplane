# .review — whole-codebase audit of Backplane

An AI-made audit of every tracked file at commit `9c6c5448a514ebcddca5a776933793e25179a796` (`main`, level with `origin/main` on 2026-11-16). No source file was changed by the audit.

This README has two parts:

1. [Finish remarks](#1-finish-remarks): the auditor's summary at the end of the run.
2. [Prompt for verifying the review](#2-prompt-for-verifying-the-review): a prompt to give a second agent, which checks the findings and turns them into an action plan.

## Files

| File | What it holds |
|---|---|
| `REVIEW.md` | The report: start here. High findings in full, confirmed Medium ones in short form. |
| `findings-all.md` | The complete register (1 MB): every finding, including possible Medium and all Low. |
| `00-map.md` | Map of the tree and how it was cut into units. |
| `coverage.md` | Every review unit: files, model, status (read in full or sampled). |
| `NN-<unit>.md` | One per review unit, `01` to `61`. |
| `90-cross-cutting.md` | Cross-unit pass, re-traces, project-level health. |
| `91-questions-*.md`, `_questions/` | The 489 cross-unit questions and how each was resolved. |
| `95-missing-features.md` | Features that are promised, partial or inferred, each with evidence. |
| `_reviewer-brief.md`, `_unit-prompts.md`, `_questions-brief.md`, `_verify.md`, `_notes-phase3.md` | The briefs and working notes the audit ran on. |

---

## 1. Finish remarks

The audit is finished: all five phases are done and the report is at `.review/REVIEW.md`. No source file was touched; the tree is still clean at `9c6c5448a` on main, level with `origin/main`.

### Result

No Critical finding was confirmed. After merging duplicates there are **79 High** (69 confirmed, 10 possible with a stated condition), **608 Medium** (423 confirmed) and **744 Low** findings.

The three main risks:

1. **Agents are not contained by the product's own approvals.** Every agent is told where the hub's home is and can read the pairing token there, then act as the person, including answering its own approval cards (`native/host/src/agents/proc.rs:299`). An agent can also unlock its own enforced rules by sending itself a message, and both browsers that agents drive run without Chromium's sandbox.
2. **Design work can be lost on ordinary paths.** Any hub stop, restart or update kills open KiCad editors without saving (`native/host/src/kicad/embed.rs:871`). A second, headless KiCad can write behind an open editor. A failed write to the event log is reported only to its caller while agents carry on.
3. **Outside input and links are trusted too far.** A downloaded KiCad project can take the hub down in at least six ways, because files are parsed inside the hub with no limits. Machines paired by an `http://` address exchange the vault and account tokens in clear. The release workflow signs and publishes any `v*` tag on any commit.

The three improvements with the most leverage:

- draw one boundary around agents;
- move parsing of untrusted files into the existing `bp-plot` child process under limits;
- make failure a visible, tested state (commit failures, multi-step jobs, editor shutdown).

### What was covered

- 61 review units: 56 read in full, 5 sampled (client screens, phone view code, test harness). `coverage.md` has the file list, model and status of each.
- Models: Fable 5.2 on 39 high-risk units and phases 3 to 5; Opus 5.5 on 10 medium-risk units; Sonnet 5.5 on 12 low-risk units and samples.
- Re-trace: the lead re-traced all 20 High findings from Opus and Sonnet units at the cited lines. 12 stayed High, 8 were lowered to Medium, none was refuted.
- Cross-unit questions: 489 resolved against the code. 281 confirmed existing findings, 62 were no issue, 72 led to new findings (one High), 12 stay open; some of those need the KiCad fork's source, which is not in this tree. Two unit findings were refuted and removed.
- Missing features: 20 promised, 37 partial, 15 inferred, each with evidence.

### What could not be verified

- No Rust code was compiled or run. The audit machine (a Mac) had no cargo, so every Rust finding comes from reading; "Confirmed" means the code path was traced, not reproduced.
- Tests that did run: iOS unit tests (all 19 suites pass), the message-catalog checks (pass), and the order server's pytest (24 pass, 8 fail only because `python-multipart` was not installed).
- Not run: Android tests (Java 11 on the machine, the build needs 17), anything needing bun, `cargo audit`, clippy, shellcheck.

### Two things to know

- `REVIEW.md` is 449 KB: it holds the High findings in full and the confirmed Medium ones in short form. The possible Medium and all Low findings are only in `findings-all.md` (1 MB), which is the complete register.
- The run hit the Fable usage limit once during phase 3. The stopped reviewers were resumed from where they were once the limit reset, and nothing was lost.

---

## 2. Prompt for verifying the review

Give the text below to a fresh agent in a checkout of this branch.

> **Note for this branch:** here `.review/` is committed on purpose (branch `review` of the fork). The Step 1 instruction to add `.review/` to `.git/info/exclude` applies to working copies where the audit was dropped in unversioned. On this branch, skip it or treat it as already satisfied.

```text
The .review/ folder in this repo contains a code audit produced by another AI agent. Your job is to verify it and turn it into an action plan. Do NOT modify any source files yet. The only files you may create or edit are inside .review/.

Step 1: Orient

- Read .review/REVIEW.md first, then .review/00-map.md and .review/coverage.md. The numbered files (NN-<unit>.md, 90-cross-cutting.md, 95-missing-features.md) hold per-module detail; open them when you need more context on a finding.
- Find the branch and commit SHA the audit was made against (recorded in 00-map.md or at the top of REVIEW.md). Compare it with the current HEAD (git log, git diff --stat <SHA>..HEAD). Tell me whether the code has changed since the audit and which audited files are affected.
- Make sure .review/ is listed in .git/info/exclude so it is not committed.

Step 2: Verify the findings

Work through the findings in REVIEW.md from Critical down to Low. For each one:
- Open the referenced file and locate the code by function name and description, not line number alone, since lines may have shifted.
- Trace the code path yourself and decide: Verified (the problem exists as described), Partly right (real problem, but the description, severity, or fix is wrong; say how), Not reproducible (explain why the audit is wrong), or Already fixed (changed since the audit).
- Treat every finding as a hypothesis, including those marked Confirmed. Do not accept the audit's reasoning without checking the code.
- For Verified findings, check whether the proposed fix is correct and complete, and note anything it would break.

Run the project's tests, linters, and type checks and compare the results with those recorded in the audit.

Save your results to .review/VERIFICATION.md as you go, one entry per finding: original severity | your verdict | file:line (current) | reasoning | adjusted severity | recommended fix.

Step 3: Assess improvements and missing features

- For each proposed improvement and missing feature, say whether you agree it is worth doing, and whether the stated effort and trade-off are realistic given the actual code.
- Flag anything important the audit missed that you noticed while verifying. Check coverage.md for units that were skipped or only sampled and tell me which of those look risky.

Step 4: Action plan

Write .review/PLAN.md:
- Fix now: verified Critical and High findings, in the order they should be fixed, each with the fix approach and the test that would prove it
- Fix soon: verified Medium findings and quick-win improvements
- Backlog: Low findings, larger improvements, and missing features, ordered by value
- Rejected: findings you could not reproduce, with reasons
- Open questions: anything that needs a decision from the maintainers

Then stop and give me a short summary: how many findings were verified versus rejected, the top 3 things to fix first, and anything in the audit you disagree with. Wait for my go-ahead before changing any source code.

Never print secrets, keys, or credentials in full in your notes or replies, even if the audit files quote them; refer to them by file and line.
```
