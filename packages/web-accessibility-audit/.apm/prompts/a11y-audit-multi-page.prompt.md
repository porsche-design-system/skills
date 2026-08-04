---
name: a11y-audit-multi-page
description: Audit multiple web pages and compare their accessibility scores. Generates a scorecard showing which pages need the most attention.
mode: agent
tools:
  - askQuestions
  - runInTerminal
  - getTerminalOutput
---

# Multi-Page Web Accessibility Comparison

Audit multiple pages of a web application and generate a comparative scorecard. Identifies systemic issues (shared across pages) vs page-specific issues.

## Instructions

Use the **a11y-audit** agent workflow (phase map 0–12):

1. Prefer askQuestions when available; otherwise ask the same structured options in chat:
   - "What is the base URL of your application?"
   - "Which pages should I audit? List the paths (e.g., /, /login, /dashboard, /settings)"
   - "What framework/tech stack?" - Options: React, Vue, Angular, Next.js, Svelte, Vanilla HTML/CSS/JS
   - "Audit method?" - Options: Runtime scan only, Code review only, Both
   - "Thoroughness?" - Quick / Standard / Deep dive

2. For each page, run the selected audit method per the agent phase map:
   - **Runtime scan (Phase 9):** `npx @axe-core/cli <URL> --tags wcag2a,wcag2aa,wcag21a,wcag21aa,wcag22aa`
   - **Code review (Phases 1–8):** Read domain skills and apply checklists
   - **Behavioral (Phase 10):** Read `a11y-playwright` (CLI) when URL + Playwright available

3. Compute per-page severity scores (0-100) and letter grades via **`a11y-severity-scoring` only**

4. Generate the comparative report to `ACCESSIBILITY-AUDIT.md` including:
   - **Page Scorecard** - side-by-side comparison table
   - **Systemic Issues** - problems found on every page (layout/nav issues)
   - **Template Issues** - problems from shared components (fix once, fix everywhere)
   - **Page-Specific Issues** - unique to individual pages
   - **Remediation Priority** - ordered by ROI (systemic fixes first)

5. Ask: "Would you like me to fix the systemic issues that affect all pages?" (Phase 12 / `a11y-issue-fixer`)

## Progress Transparency

Announce each transition:

- **Before a page pass:** "Auditing page [N/total]: [URL] - running [phase/skill]..."
- **After completion:** Summarize per-page results before moving to cross-page analysis
- **On failure:** "Phase/skill failed on [URL]: [reason]. Continuing with remaining pages."
