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
- **Thoroughness:** Standard review (all phases)
- **Target standard:** WCAG 2.2 AA
- **Screenshots:** No
- **Report path:** `ACCESSIBILITY-AUDIT.md`
- **Include:** Severity scoring, confidence levels, framework-specific notes

## Instructions

Use the **a11y-audit** agent workflow:

1. Skip Phase 0 discovery - settings are pre-configured above
2. Run axe-core against the URL first:

   ```bash
   npx @axe-core/cli ${input:pageUrl} --tags wcag2a,wcag2aa,wcag21a,wcag21aa --save ACCESSIBILITY-SCAN.json
   ```

3. Run code review phases (1-8) by Reading domain skills (`a11y-aria`, `a11y-keyboard`, etc.) and applying their checklists
4. Compile findings with severity scoring and confidence levels
5. Compute page accessibility score (0-100) and letter grade
6. Write the full report to `ACCESSIBILITY-AUDIT.md`
7. Offer interactive fix mode for auto-fixable issues

## Progress Transparency

This workflow loads domain skills per phase. Announce each transition:

- **Before a domain pass:** "Running [phase name] using [skill name]..."
- **After completion:** Summarize findings (issue count, severity breakdown)
- **On failure:** "[Skill/phase] encountered an error: [reason]. Skipping this phase and continuing."
