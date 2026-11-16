# Brief for a unit reviewer

You are reviewing one unit of the Backplane repository as part of a whole-codebase audit. Your prompt names your unit number, its name and its file list.

- Repository: the current working directory.
- Commit under review: `9c6c5448a514ebcddca5a776933793e25179a796` on `main`. Review the files as they are on disk; do not pull or check out anything.
- Read `.review/00-map.md` first, sections "What the project is", "Architecture", "Conventions", "Stated intent" and "Cross-cutting concerns". It is the shared context for every reviewer.

## Hard rules

- Do NOT modify any source file. The only file you may create or edit is your own result file, `.review/NN-<unit-name>.md` (the exact name is in your prompt). Do not touch `coverage.md` or any other reviewer's file.
- Read-only work only: read files, search, `git log`, `git blame`, `git show`. Do not build, do not run tests, do not start any program of this repository (no hub, no scripts from `scripts/`), do not install anything, do not use the network. There is no Rust toolchain on this machine.
- Never write under `~/.backplane*` or `~/.local/share/backplane`.
- Do not spawn further agents.

## What to do

1. Read every file of your unit in full. Lines are very long in this tree (one line can hold a whole function), so read by byte ranges or with a wrapped view when a line is cut off; do not skip a file because it looks short in lines.
2. Read the callers and callees that cross the unit boundary, including code in other units, far enough to confirm or refute what you suspect. Use search to find them.
3. Check for:
   - logic errors and edge cases (null, empty, boundaries, off-by-one, integer overflow, time and time zones);
   - error handling and partial-failure state (what is left behind when step 2 of 3 fails; `.unwrap()` or `.expect()` on data from outside the process);
   - async and concurrency issues (locks held across awaits, lock order, races between actors, lost wakeups, broadcast lag);
   - resource leaks, missing timeouts, unbounded loops, queues and buffers;
   - data integrity (validation, atomic writes, idempotency, migrations, events that must stay decodable forever);
   - security (input handling, authentication and authorisation, secrets in logs or URLs, injection into shells or arguments, path traversal, unsafe deserialisation, loopback trust);
   - contract mismatches between this unit and its callers or the other side of a wire format;
   - weak or missing tests for the risky paths.
4. Also note maintainability problems that have a concrete cost: dead or unreachable code, duplicated logic, misleading names or comments, excessive complexity, stale TODO/FIXME/HACK markers, deviations from the project's own conventions (the rules in `AGENTS.md` and in the map).
5. Trace the code path to confirm each bug. Label each finding **Confirmed** (you traced the path and it fails as described) or **Possible** (state the exact condition that would trigger it). Do not report what you have not traced. Prefer fewer, well-substantiated findings over volume. Style-only nitpicks go in one short list at the end.

Severity:
- **Critical**: data loss or corruption of the event log or a person's design files, remote code execution, authentication bypass, a secret disclosed to another party, in a realistic configuration.
- **High**: a wrong result or broken feature a normal user will meet, a crash or hang of the hub, a security weakness that needs an unusual but plausible condition, an unrecoverable state.
- **Medium**: a wrong result in an edge case, a leak or unbounded growth over long uptimes, a missing check with limited impact, a misleading contract.
- **Low**: minor, cosmetic, or only maintainability.

## Output: write `.review/NN-<unit-name>.md` in exactly this layout

```
# NN <unit-name>

Commit: 9c6c5448a514ebcddca5a776933793e25179a796
Files read in full: <list>
Boundary code read: <files in other units you read, briefly>
Not read or only skimmed: <list with reason, or "none">

## Summary
<five to ten sentences: what the unit does, its overall quality, the main risks>

## Findings
| Severity | Status | Location | Problem | Impact | Fix |
|---|---|---|---|---|---|
| High | Confirmed | native/host/src/x.rs:123 | <what is wrong> | <concrete scenario> | <fix; a short snippet in backticks if useful> |

## Improvements
| Priority | Area | Proposal | Why | Effort | Trade-off |
|---|---|---|---|---|---|
| P1 | <area> | <proposal> | <what it fixes or enables> | S | <cost or risk> |

## Feature gaps
| Gap | Kind | Evidence | Note |
|---|---|---|---|
| <stub, unimplemented branch, half-finished feature, asymmetric API, option parsed but ignored> | stub / partial / asymmetric / ignored-option | file:line | <one sentence> |

## Missing tests
- <specific test case to add, most important first>

## Cross-unit questions
- <a question another unit's code must answer, with the file:line that raises it>

## Style nitpicks
- <short list, or "none">
```

Table rules, so the files can be merged by script: one finding per row; no line breaks inside a cell; write a literal pipe inside a cell as `\|`; Location is `repo/relative/path:line` (one primary location; mention others in Problem); Severity is one of Critical, High, Medium, Low; Status is Confirmed or Possible; Priority is P1, P2 or P3; Effort is S, M or L. Because lines are long, add the function or item name in Problem when a line number alone would not find it.

## What to send back as your final message

A short report, under 250 words: the path of the file you wrote, the count of findings by severity, the two or three most important findings in one line each with their location, and your cross-unit questions. The full detail belongs in the file, not in the message.
