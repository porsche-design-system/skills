---
description: "Structured finding format and audit reliability for accessibility audits"
applyTo: "**/*.{md,agent.md}"
---

## Dependencies

None required. This is a foundational reliability standard for audit outputs.

**Shared terminology:** All severity levels, confidence terms, and role labels used in this file are defined in `a11y-agent-terminology.instructions.md`. When in doubt, consult that file before using a term.

# Accessibility Audit Finding Reliability

These rules apply when producing accessibility audit findings, scores, and remediation reports (especially via the `a11y-audit` agent and its skills). Structured fields keep severity, location, and confidence unambiguous for humans and for follow-up fix/export skills.

---

> **Impact:** Unstructured prose in audit reports causes misinterpretation — one reader's "severe issue" is another's "moderate finding." Structured fields make results actionable.

## 1. Structured Findings

Every accessibility finding MUST include:

- **Rule ID** (e.g., `WCAG-1.1.1`, `color-contrast`, axe rule id)
- **Severity** (`critical` | `serious` | `moderate` | `minor`)
- **Location** (file path + line number, and/or element selector / URL)
- **Description** (one sentence: what is wrong)
- **Remediation** (one sentence: how to fix it)
- **Confidence** (`high` | `medium` | `low`)
- **WCAG criterion** when applicable (id + name + level)
- **Help URL** when available (from `a11y-help-url-reference`)

**Score format** — every scored page/component MUST include:

- **Score** (0-100 integer)
- **Grade** (`A` | `B` | `C` | `D` | `F`)
- **Issue counts** by severity (critical/serious/moderate/minor)
- **Pass/fail verdict** against the target standard (boolean + short reason)

**Action result format** — every state-changing fix MUST report:

- **Action taken** (what was done)
- **Target** (file/selector)
- **Result** (`success` | `failure` | `skipped`)
- **Reason** (why, if failure or skipped)

---

> **Impact:** Silent edits outside the agreed fix scope break trust. Explicit action sets keep remediation auditable.

## 2. Constrained Actions During Audits

**Audit / scan mode** (default for `a11y-audit`):

- May read files, fetch URLs, run non-destructive scanners (axe, Lighthouse, Playwright read-only)
- May produce findings, scores, and reports
- Must not modify source files until the user opts into fix mode

**Fix mode** (via `a11y-issue-fixer` skill / fix prompts):

- May edit source only for agreed issues
- Auto-fixable changes may apply in batch when the user selects that mode
- Human-judgment fixes require per-item approval (`askQuestions`)
- Prefer re-scan after fixes to verify resolution

**Escalation:** If a request is outside web accessibility audit scope (e.g., Office/PDF document remediation), state the limitation and continue with web-only guidance.

---

> **Impact:** Incomplete findings force rework. Validate required fields before writing `ACCESSIBILITY-AUDIT.md` or CSV export.

## 3. Pre-Validate Before Reporting

Before finalizing a report or exporting CSV/JSON:

1. Every finding has severity, confidence, location, description, remediation
2. Scores/grades were computed with `a11y-severity-scoring` (not invented ad hoc)
3. Duplicate findings across skills/scanners were merged
4. Help URLs are attached where the reference skill provides them

If a skill pass returns incomplete findings, re-apply that skill with explicit field requirements rather than guessing.

---

> **Impact:** Progress narration prevents the audit from looking hung during long skill batches.

## 4. Progress Transparency

When loading domain or ops skills:

- **Before:** Announce which skill(s) and what they cover
- **After:** Summarize issue count and severity breakdown
- **On failure:** State the skill/phase error, skip or retry, and continue the audit

Never silently skip a scheduled skill without telling the user.

---

## 5. Single-Agent Package Note

This package ships **one** audit agent (`a11y-audit`). Depth lives in skills loaded with Read — not in sibling specialist agents. Do not invent handoffs to agents that are not in this package.
