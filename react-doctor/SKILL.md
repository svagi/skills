---
name: react-doctor
description: Audit React code for common pitfalls (unnecessary useEffect, redundant state, state mutation, re-render issues, memo/context misuse, list keys, stale closures, ref misuse). Use when user wants to review or improve React performance, mentions slow renders, useEffect problems, or asks to audit React components.
---

# React Doctor

Find common React mistakes in TypeScript/JavaScript files. Surface a numbered list. Apply user-selected fixes.

Catalog with detection signals, fixes, and examples: [PITFALLS.md](PITFALLS.md).

## Workflow

### 1. Scope

- No arg → whole repo from cwd.
- Arg → that path or glob.
- Files: `.tsx`, `.jsx`, `.ts`, `.js`.
- Exclude: `node_modules`, `dist`, `build`, `.next`, `coverage`, `.git`.

Zero matches: print `No React files found in <scope>.` and stop.

### 2. Scan

Spawn `Agent` (`subagent_type=Explore`) with this brief:

> Read [PITFALLS.md](PITFALLS.md). Scan every `.tsx`/`.jsx`/`.ts`/`.js` file under `<scope>`, excluding the dirs above. For each match, return: `pitfall_id` (1–29), `file`, `line`, `name`, `risk`, one-line `fix_summary`. Skip false positives.

### 3. Present findings

One line per finding. Sort: `high` → `medium` → `low`, then by file. Renumber from 1; don't reuse catalog ids. Display risk as `[low]` / `[med]` / `[high]`:

```
1. [high] App.tsx:5     Single context mixing data+actions → split contexts
2. [med]  List.tsx:18   Missing key prop → add stable id key
3. [low]  Cart.tsx:42   Derived state in useState → compute during render
```

Zero findings: print `No React pitfalls found in <scope>.` and stop.

### 4. User picks

Ask: *"Which to fix? (`1,3,5` / `all` / `none`)"*

`all` = every finding; `none` = stop; comma-separated numbers = those picks.

### 5. Apply fixes

For each pick, in order:

- **`low` / `medium`** → apply with Edit per recipe in [PITFALLS.md](PITFALLS.md).
- **`high`** → print *"Plan: \<one-line description of the change\>. Apply? (y/n)"*. On `y` apply, on `n` skip.

Match surrounding style. If a fix can't be applied cleanly, report and skip — don't guess.

### 6. Summarize

```
Applied N fix(es):
- file:line — pitfall name
Skipped M: <reasons>
```
