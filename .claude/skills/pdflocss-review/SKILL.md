---
name: pdflocss-review
description: Evaluate whether a project's CSS/SCSS (or CSS-in-JS / scoped CSS) architecture follows PDFLOCSS — layer separation (foundation/layout/component/project/utility), MindBEMding naming, single-class-per-element, class-on-every-tag — and score it with actionable fixes. Then, only if asked, edit the source to apply those fixes. Use when the user wants a PDFLOCSS or CSS-design review of a specific PR (by number) or the whole project, or asks to fix/improve CSS to match PDFLOCSS.
---

Scores a codebase's CSS against PDFLOCSS and reports concrete fixes; only edits code when asked to.

Full rule catalog and scoring table: [PDFLOCSS-RULES.md](PDFLOCSS-RULES.md) — load it at Step 3, not before.

## Process

### 1. Resolve the target

- A PR number given (by the user, or as this skill's argument) → target is that PR's diff. Get it via `gh pr diff <number>` and `gh pr view <number> --json files,title,body` (see `docs/agents/issue-tracker.md` if present for repo-specific conventions; otherwise these `gh` commands work directly).
- No PR number → target is the whole project's CSS.

### 2. Collect evaluable files

Find files carrying CSS design decisions: `.css`/`.scss`/`.sass`/`.less`, plus CSS-in-JS (styled-components/emotion template literals) and scoped-style blocks (Vue/Svelte SFC `<style>`, CSS Modules `*.module.css`). For a PR target, restrict to files touched by the diff.

**Directory structure and file layout follow whatever the project's framework already dictates** — don't penalize a Next.js/Nuxt/Rails/etc. layout for not matching PDFLOCSS's literal `foundation/layout/object/...` folder names. What matters is whether the underlying role separation (init/structure/reusable-part/page-specific/fine-tuning) is findable somewhere — naming, file split, or folder.

If scoped CSS is in play (CSS Modules, `<style scoped>`, CSS-in-JS), the pseudo-scope PDFLOCSS gets from page+section naming is already provided by the framework — evaluate only what applies *inside* the component (see the "適用範囲の調整" section of the rules file once loaded).

**No evaluable files found** (whole-project scan turns up none, or the PR touches no CSS-bearing file) → stop here. Report plainly that there's nothing to evaluate and why (e.g. "対象PRにCSS変更なし" / "CSS/SCSSファイルが見つからない"). Do not proceed to scoring, do not post a PR comment.

### 3. Evaluate

Load [PDFLOCSS-RULES.md](PDFLOCSS-RULES.md) now. Check every collected file against every rule category that applies to its scope (full-file CSS vs. scoped-component CSS — see the file's applicability notes). For each violation, capture: file:line, the offending code (short quote), which rule it breaks, and its severity (must-fix / should-fix / suggestion) per the rules file's severity guide.

Compute the category scores and total (0–100) per the rules file's table, re-scaled if any category is N/A for this codebase's CSS approach.

### 4. Report

- **PR target** → post the score and issue list as a PR comment: `gh pr comment <number> --body "..."`. Structure: total score, one line per category with its sub-score, then issues grouped by severity (must-fix first), each with file:line and a one-line fix.
- **Whole-project target** → print the same structure directly in this session (no file is written, no comment posted) — it's read from the conversation log, not persisted.

Keep it usable: a score alone or an issue list alone is half the job — both together are what makes this actionable.

### 5. Improve (only when the user asks for fixes)

Don't do this unprompted — Steps 1–4 are the full deliverable unless the user separately asks to have the issues fixed.

When asked: work through the must-fix issues first, then should-fix, editing the source per the rules file's guidance for the correct PDFLOCSS pattern (not just "delete the violation" — replace it with the compliant form: correct prefix, correct selector, correct layer). Leave suggestions unless the user says to include them.

After editing, re-run Step 3's evaluation on the changed files and report the before/after score plus which issues were fixed vs. skipped (and why, if skipped — e.g. a fix would require a framework-level change out of scope).
