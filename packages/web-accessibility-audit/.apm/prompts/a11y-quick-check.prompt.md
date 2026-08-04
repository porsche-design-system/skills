---
name: a11y-quick-check
description: Quick accessibility triage of a web page. Runtime axe-core scan only - no code review. Fast pass/fail verdict with score.
mode: agent
tools:
  - askQuestions
  - runInTerminal
  - getTerminalOutput
---

# Quick Web Accessibility Check

Fast triage - run axe-core against a live URL and get a pass/fail verdict. No code review, no screenshots. Fastest way to check a page.

## Page to check

**URL:** `${input:pageUrl}`

## Settings

- **Audit method:** Runtime scan only (axe-core)
- **Thoroughness:** Quick profile — phases **0 (skipped) → 1 (light) → 9 → 11** only
- **Target standard:** WCAG 2.2 AA
- **Report:** Inline (no separate file)

## Instructions

Use the **a11y-audit** agent **quick** profile:

1. Skip Phase 0 discovery — settings are pre-configured above
2. Optionally skim structure (Phase 1) only if the URL HTML is already available; do **not** run phases 2–8 or 10
3. Run axe-core against the URL (Phase 9):

   ```bash
   npx @axe-core/cli ${input:pageUrl} --tags wcag2a,wcag2aa,wcag21a,wcag21aa,wcag22aa
   ```

4. Parse results; score with `a11y-severity-scoring` if useful; report inline:

```text
Quick Check: ${input:pageUrl}
Score: [0-100] ([grade])
Violations: [count]
Critical / Serious / Moderate / Minor: [counts]
Top issues: [up to 5 rule ids + short descriptions]
Verdict: Pass / Needs work / Fail
```

5. Offer a full standard or deep-dive audit if the user wants remediation depth
