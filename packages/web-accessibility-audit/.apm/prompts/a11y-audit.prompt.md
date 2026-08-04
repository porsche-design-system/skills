---
name: a11y-audit
description: Run a full web accessibility audit on a single page URL using axe-core runtime scanning and agent code review. Produces a scored report with remediation steps.
mode: agent
agent: a11y-audit
tools:
  - askQuestions
  - runInTerminal
  - getTerminalOutput
---

# Web Page Accessibility Audit

Run a comprehensive accessibility audit on a single web page. Combines axe-core runtime scanning with agent-driven code review.

## Page to audit

**URL:** `${input:pageUrl}`

## Settings

- **Audit method:** Both (axe-core runtime scan + code review)
- **Thoroughness:** Standard review (phases 0–11; skip discovery questions — settings below)
- **Target standard:** WCAG 2.2 AA
- **Screenshots:** No
- **Report path:** `ACCESSIBILITY-AUDIT.md`
- **Include:** Severity scoring via `a11y-severity-scoring`, confidence levels, framework-specific notes

## Instructions

Use the **a11y-audit** agent workflow and its single phase map (phases 1–12):

1. Skip Phase 0 discovery questions — settings are pre-configured above. Still Read `a11y-framework` if the stack is detectable.
2. Run Phase 9 scanners against the URL first:

   ```bash
   npx @axe-core/cli ${input:pageUrl} --tags wcag2a,wcag2aa,wcag21a,wcag21aa,wcag22aa --save ACCESSIBILITY-SCAN.json
   ```

3. Run code review phases 1–8 by Reading domain skills and applying their checklists:
   - 1: `a11y-alt-text-headings`, `a11y-text-quality` (+ `a11y-media` if media present)
   - 2: `a11y-keyboard` (+ `a11y-modal` if overlays)
   - 3: `a11y-forms`
   - 4: `a11y-contrast` (+ `a11y-design-system` if tokens)
   - 5: `a11y-live-regions`
   - 6: `a11y-aria`
   - 7: `a11y-tables` (skip if none)
   - 8: `a11y-links`
4. Phase 10: Read `a11y-playwright` and run CLI behavioral scans when Playwright is available
5. Phase 11: Read `a11y-severity-scoring` and `a11y-help-url-reference`; write `ACCESSIBILITY-AUDIT.md`
6. Offer Phase 12 interactive fix mode via `a11y-issue-fixer`

## Progress Transparency

Announce each transition:

- **Before a domain pass:** "Running [phase name] using [skill name]..."
- **After completion:** Summarize findings (issue count, severity breakdown)
- **On failure:** "[Skill/phase] encountered an error: [reason]. Skipping this phase and continuing."
