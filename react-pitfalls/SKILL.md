---
name: react-pitfalls
description: Audit React code for common pitfalls (unnecessary useEffect, redundant state, state mutation, re-render issues, memo/context misuse, list keys, stale closures, ref misuse). Use when user wants to review or improve React performance, mentions slow renders, useEffect problems, or asks to audit React components.
---

# React Pitfalls

Find common React mistakes in TypeScript/JavaScript files. Surface a numbered list. Apply user-selected fixes.

Catalog with detection signals, fixes, and examples: [PITFALLS.md](PITFALLS.md).

## Workflow

### 1. Scope

- No arg → whole repo from cwd.
- Arg → that path or glob.
- Files: `.tsx`, `.jsx`, `.ts`, `.js`.
- Exclude: `node_modules`, `dist`, `build`, `esm`, `.next`, `coverage`, `temp`, `.git`.

Zero matches: print `No React files found in <scope>.` and stop.

### 2. Scan

Delegate the scan to a read-only exploration subagent with this brief:

> Read [PITFALLS.md](PITFALLS.md). Scan every `.tsx`/`.jsx`/`.ts`/`.js` file under `<scope>`, excluding the dirs above. For each match, return: `pitfall_id` (1–29), `file`, `line`, `name`, `risk`, one-line `fix_summary`. Skip false positives.

### 3. Present findings

One line per finding. Sort: `high` → `medium` → `low`, then by file. Renumber from 1; don't reuse catalog ids. Display risk as `[low]` / `[med]` / `[high]`:

```text
1. [high] App.tsx:5     Single context mixing data+actions → split contexts
2. [med]  List.tsx:18   Missing key prop → add stable id key
3. [low]  Cart.tsx:42   Derived state in useState → compute during render
```

Zero findings: print `No React pitfalls found in <scope>.` and stop.

### 4. User picks

Ask: _"Which to fix? (`1,3,5` / `all` / `none`)"_

`all` = every finding; `none` = stop; comma-separated numbers = those picks.

### 5. Apply fixes

Map each pick back to its finding (display numbers ≠ catalog ids), then for each, in order — looking up the recipe in [PITFALLS.md](PITFALLS.md) by the finding's `pitfall_id`:

- **`low` / `medium`** → apply with Edit per the recipe.
- **`high`** → print _"Plan: \<one-line description of the change\>. Apply? (y/n)"_. On `y` apply, on `n` skip.

Match surrounding style. If a fix can't be applied cleanly, report and skip — don't guess.

### 6. Validate

Run the repo's post-edit checks (per its agent instructions; else the nearest `package.json` format/lint/test scripts) for each package with edited files. A failed check → report that fix as applied-with-failures, not success. Skipped → say why.

### 7. Summarize

```text
Applied N fix(es):
- file:line — pitfall name
Skipped M: <reasons>
Validation: <checks run → result>
```
