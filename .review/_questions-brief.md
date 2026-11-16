# Brief for resolving cross-unit questions (phase 3)

The unit reviewers of this audit each ended their report with questions about code outside their unit. Your prompt names one file in `.review/_questions/` holding the questions whose answers lie in one area of the code. Each line gives the unit that asked (in brackets) and the question as its reviewer wrote it.

Context: `.review/00-map.md` (architecture and conventions) and `.review/_reviewer-brief.md` (the severity scale and the findings table format). Commit under review: `9c6c5448a514ebcddca5a776933793e25179a796`, the files as they are on disk.

## Hard rules

- Read-only. The only file you may create or edit is your own result file, named in your prompt. Do not modify any source file or any other file in `.review/`.
- No builds, no tests, no scripts of this repository, no network. Reading, searching and read-only `git` commands only. Do not spawn further agents.

## What to do for each question

1. Merge questions that ask the same thing (several reviewers asked about the same line); answer them once and list every asker.
2. Check whether a unit result file already answers it: the unit that owns the code was often sent the question and answered it in its findings or in its "Cross-unit questions" section. Search `.review/[0-9][0-9]-*.md` for the file name, function or keyword. If it is answered there, cite that file and give the answer in one or two sentences; verify it against the code only when the answer looks doubtful.
3. Otherwise read the code and answer it, with `repo/relative/path:line` evidence. Lines are very long in this tree: read by byte ranges when a line is cut off, and name the function.
4. Give each question one outcome:
   - `no issue`: the code does the safe or intended thing.
   - `existing finding`: it confirms or extends a finding already in a unit file (cite the unit file and the row's location).
   - `new finding`: a defect no unit file reports. Give it a severity from the scale in `_reviewer-brief.md` and add it to the "New findings" table.
   - `open`: it cannot be decided from this tree (say what is missing, for example the KiCad fork's source or a product decision).
5. Do not report what you have not traced.

## Output: write your result file in exactly this layout

```
# 91 questions: <area>

Questions in the file: <n>. After merging duplicates: <m>.
Outcomes: no issue <a>, existing finding <b>, new finding <c>, open <d>.

## Answers
| # | Asked by | Question | Answer | Evidence | Outcome |
|---|---|---|---|---|---|
| 1 | 02, 09 | <the question in one sentence> | <the answer> | <path:line; or the unit file that answers it> | no issue |

## New findings
| Severity | Status | Location | Problem | Impact | Fix |
|---|---|---|---|---|---|

## Contract mismatches between units
- <interfaces where the two sides disagree, one line each with both locations; or "none found">
```

Table rules: one row per merged question; no line breaks inside a cell; a literal pipe inside a cell is written `\|`; Severity is Critical, High, Medium or Low; Status is Confirmed or Possible.

## Final message

Under 200 words: the path of your file, the outcome counts, each new finding of High severity in one line, and anything that contradicts a finding in a unit file.
