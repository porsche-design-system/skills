---
name: a11y-severity-scoring
description: Compute web accessibility scores (0-100, A-F grades) with severity scoring, confidence levels, and remediation tracking across audits.
user-invocable: false
---

# Web Severity Scoring

This skill is the **sole scoring authority** for `a11y-audit`. Do not use a different formula from memory or from the agent body.

## Severity Scoring Formula

```text
Page Score = 100 - (sum of weighted findings)

Weights:
  Critical (confirmed, all three sources):   -18 points
  Critical (high confidence, both sources):  -15 points
  Critical (high confidence, single source): -10 points
  Critical (medium confidence):               -7 points
  Critical (low confidence):                  -3 points
  Serious (high confidence):                  -7 points
  Serious (medium confidence):                -5 points
  Serious (low confidence):                   -2 points
  Moderate (high confidence):                 -3 points
  Moderate (medium confidence):               -2 points
  Moderate (low confidence):                  -1 point
  Minor:                                      -1 point

Floor: 0 (minimum score)
```

### Scoring Profiles

Use a profile to tune strictness by context while keeping comparable grade bands:

| Profile | Intended Use | Multiplier |
|---------|--------------|------------|
| balanced (default) | Standard product delivery | 1.0 |
| strict | Regulated/public-sector releases | 1.15 |
| advisory | Early design and prototyping | 0.8 |

Apply the profile multiplier to each final deduction after confidence handling.

### Formula

```pseudocode
page_score = 100
for each finding:
    base = lookup(severity, confidence_level, source_count)  // from table above
    multiplier = 1.2 if confidence_level == "confirmed" else 1.0
    deduction = base × multiplier
    page_score = max(0, page_score - deduction)
```

The values in the lookup table above are **base deductions** (pre-multiplier).
"Confirmed" findings (validated by all three sources: axe-core + agent review + Playwright) apply an additional 1.2× multiplier.

**Example:** One Critical finding at confirmed confidence = 18 (base) × 1.2 = **21.6 points** deducted → page score 78.

### Calibration Layer (v2)

To reduce false-positive inflation and stabilize trends, apply a calibration coefficient by rule family:

```text
calibrated_deduction = deduction × calibration_coefficient(rule_family)
```

Recommended initial coefficients:

| Rule Family | Coefficient | Rationale |
|-------------|-------------|-----------|
| Keyboard/focus | 1.1 | High functional impact at runtime |
| Forms/labels/errors | 1.05 | High completion risk for core tasks |
| Semantics/structure | 1.0 | Baseline scoring |
| Link text/context | 0.9 | Higher context variance |
| Content quality (alt/link clarity) | 0.85 | Needs human review more often |

Update coefficients quarterly from confirmed outcomes. Avoid changing coefficients more than +/-0.1 per cycle.

## Score Grades

| Score | Grade | Meaning |
|-------|-------|---------|
| 90-100 | A | Excellent - minor or no issues, meets WCAG AA |
| 75-89 | B | Good - some issues, mostly meets WCAG AA |
| 50-74 | C | Needs Work - multiple issues, partial WCAG AA compliance |
| 25-49 | D | Poor - significant accessibility barriers |
| 0-24 | F | Failing - critical barriers, likely unusable with AT |

## Confidence Levels

| Level | Weight | When to Use |
|-------|--------|-------------|
| Confirmed | 120% | Validated by all three sources: axe-core + agent review + Playwright behavioral testing |
| High | 100% | Confirmed by axe-core + agent, or definitively structural (missing alt, no labels, no lang) |
| Medium | 70% | Found by one source, likely issue (heading edge cases, questionable ARIA, possible keyboard traps) |
| Low | 30% | Possible issue, needs human review (alt text quality, reading order, context-dependent link text) |

### Source Correlation

Issues found by both axe-core AND agent review are automatically upgraded to **high confidence** regardless of individual confidence ratings.

Issues found by all three sources (axe-core + agent review + Playwright behavioral testing) are upgraded to **confirmed confidence** with a 1.2x weight multiplier. This applies when:

- axe-core reports a violation
- Agent code review identifies the same issue
- Playwright behavioral scan confirms the issue at runtime (e.g., keyboard trap confirmed by actual Tab traversal, contrast failure confirmed by rendered CSS computation)

When Playwright is not available, the maximum achievable confidence remains **High (100%)**. The confirmed tier is additive — it never downgrades findings.

### Confidence Drift Guard

Track predicted confidence versus post-triage outcome and compute drift:

```text
drift = abs(predicted_confidence_score - observed_confirmation_rate)
```

Operational guideline:

- drift <= 0.10: stable
- drift 0.11-0.20: tune coefficients and source mapping
- drift > 0.20: freeze profile changes and run rule-level review

## Scorecard Format

### Single Page

```markdown
## Accessibility Score

| Metric | Value |
|--------|-------|
| Page | [URL] |
| Score | [0-100] |
| Grade | [A-F] |
| Critical | [count] |
| Serious | [count] |
| Moderate | [count] |
| Minor | [count] |
```

### Multi-Page

```markdown
## Accessibility Scorecard

| Page | Score | Grade | Critical | Serious | Moderate | Minor |
|------|-------|-------|----------|---------|----------|-------|
| / | 82 | B | 0 | 2 | 3 | 1 |
| /login | 91 | A | 0 | 0 | 2 | 1 |
| /dashboard | 45 | D | 2 | 4 | 3 | 2 |
| **Average** | **72.7** | **C** | **2** | **6** | **8** | **4** |
```

## Cross-Page Pattern Classification

| Pattern Type | Definition | Remediation ROI |
|-------------|-----------|-----------------|
| Systemic | Same issue on every audited page | Highest - usually layout/nav, fix once |
| Template | Same issue on pages sharing a component | High - fix the shared component |
| Page-specific | Unique to one page | Normal - fix individually |

## Remediation Tracking

### Change Classification

| Status | Definition |
|--------|-----------|
| Fixed | Issue was in previous report but no longer present |
| New | Issue not in previous report, appears now |
| Persistent | Issue remains from previous report |
| Regressed | Issue was previously fixed but has returned |

### Progress Metrics

- **Issue reduction:** `(fixed / previous_total) * 100`
- **Score change:** `current_score - previous_score`
- **Pages improved:** count of pages with higher scores than previous audit
- **Trend:** improving (score up 5+), stable (within 5), declining (score down 5+)

### Normalized Trend Metric (Cross-Audit)

When audit scope changes between runs, use normalized change:

```text
normalized_score = raw_score - (scope_variance_penalty)
scope_variance_penalty = min(10, abs(previous_pages - current_pages) * 0.8)
```

Use normalized score for trend charts and use raw score for release gates.

## Output Metadata (Recommended)

Include these fields in generated score artifacts for reproducibility:

```yaml
scoring:
  model: a11y-severity-scoring-v2
  profile: balanced
  calibrationVersion: 2026-q2
  confidenceSources:
    - axe-core
    - agent-review
    - playwright
  failThresholds:
    critical: 1
    score: 75
```

This metadata allows deterministic re-runs and audit-to-audit comparisons.

## Issue Severity Categories

### Critical

- No keyboard access to essential functionality
- Missing form labels on required fields
- Images conveying critical information have no alt text
- Color is the sole means of conveying information
- Keyboard traps with no escape

### Serious

- Missing skip navigation
- Poor heading hierarchy (skipped levels)
- Focus not visible on interactive elements
- Form errors not programmatically associated
- Missing ARIA on custom widgets

### Moderate

- Redundant ARIA on semantic elements
- Suboptimal heading structure (multiple H1s)
- Missing autocomplete on identity fields
- Links to new tabs without warning
- Missing table captions

### Minor

- Redundant title attributes
- Suboptimal button text
- Missing landmark roles where semantic elements exist
- Decorative images with non-empty alt text

## Cross-Page Analyzer Procedures (from bridge)

## Authoritative Sources

- **WCAG 2.2 Specification** — https://www.w3.org/TR/WCAG22/
- **axe-core Rules** — https://github.com/dequelabs/axe-core/tree/develop/lib/rules
- **axe DevTools** — https://www.deque.com/axe/devtools/

This skill is a procedure module for `a11y-audit` for cross-page accessibility analysis. It receives aggregated scan findings from multiple web pages and identifies patterns, computes scores, and generates analysis summaries.

## Capabilities

### Pattern Detection
- Identify issues that repeat across every audited page (systemic - usually layout/nav)
- Detect issues shared by pages using the same template/layout component (template-level)
- Isolate issues unique to individual pages (page-specific)
- Flag the highest ROI fixes (systemic issues that affect all pages)

### Severity Scoring

Compute a weighted accessibility risk score (0-100) for each page:

```text
Page Score = 100 - (sum of weighted findings)

Weights:
  Critical (high confidence, both sources):  -15 points
  Critical (high confidence, single source): -10 points
  Critical (medium confidence):               -7 points
  Serious (high confidence):                  -7 points
  Serious (medium confidence):                -5 points
  Moderate (high confidence):                 -3 points
  Moderate (medium confidence):               -2 points
  Minor:                                      -1 point

Floor: 0
```

### Score Grades

| Score | Grade | Meaning |
|-------|-------|---------|
| 90-100 | A | Excellent - meets WCAG AA |
| 75-89 | B | Good - mostly meets WCAG AA |
| 50-74 | C | Needs Work - partial compliance |
| 25-49 | D | Poor - significant barriers |
| 0-24 | F | Failing - unusable with AT |

### Cross-Page Pattern Classification

| Type | Definition | Fix Strategy |
|------|-----------|-------------|
| Systemic | Same issue on every page | Fix in shared layout - highest ROI |
| Template | Same issue on pages sharing a component | Fix the shared component |
| Page-specific | Unique to one page | Fix individually |

### Accessibility Tree Diffing

When Playwright accessibility tree snapshots are available from `a11y-playwright`, compare structural consistency across pages:

1. **Landmark consistency** — Verify the same landmark roles (banner, navigation, main, contentinfo) appear on every page. Flag pages where a landmark is missing that exists on all other pages.
2. **Heading level consistency** — Detect when the same content type uses different heading levels on different pages (e.g., page title is H1 on homepage but H2 on subpages).
3. **ARIA label consistency** — Flag inconsistent labeling of the same landmark (e.g., `aria-label="Main navigation"` on some pages but `aria-label="Nav"` on others).
4. **Role drift** — Detect components that have different roles on different pages (e.g., `role="navigation"` on homepage but `role="list"` on subpages for the same nav component).

Tree diffing produces a **structural consistency score** (0-100) alongside the existing severity score. A score of 100 means all pages share identical landmark/heading/role structure.

### Keyboard Flow Comparison

When Playwright keyboard scan results are available, compare tab-order sequences across pages:

1. **Navigation order consistency** — Check that shared navigation elements (header nav, skip links, footer links) appear in the same relative tab order across all pages.
2. **Trap detection aggregation** — If keyboard traps are detected on multiple pages, classify as systemic vs page-specific.
3. **Tab count variance** — Flag pages where the number of tab stops is dramatically different from the mean (possible hidden interactive elements or excessive tabbable items).
4. **Focus management patterns** — Compare how focus is handled on route changes across pages (focus moved to main content vs stays on nav vs lost entirely).

### Remediation Tracking

When baseline report data is provided:
- Classify findings as Fixed, New, Persistent, or Regressed
- Calculate progress metrics (% reduction, score change, trend)
- Generate comparison summaries

## Output Format

Return structured analysis including:
- Cross-page pattern summary with frequencies
- Per-page severity scores and grades
- Overall average score and grade
- Pattern classification (systemic / template / page-specific)
- Remediation progress (if baseline provided)
- Scorecard table ready for inclusion in the audit report

---

## Reliability

### Role

This skill is a procedure module for `a11y-audit`. This skill does not edit source files; return structured findings. It aggregates per-page findings from web scanners into cross-page patterns, scores, and scorecards. It does NOT modify files or re-scan pages.

### Output Contract

Your output MUST include:
- `patterns`: list of cross-page patterns, each with frequency, severity, affected pages, and classification (`systemic` | `template` | `page-specific`)
- `scores`: per-page score (0-100) and grade (A-F)
- `overall_score`: average score and grade
- `scorecard`: table with page URL, score, grade, issue counts by severity
- `remediation_delta`: (if baseline provided) fixed/new/persistent/regressed counts
- `tree_diff`: (if Playwright data available) structural consistency score, landmark/heading/role inconsistencies
- `keyboard_comparison`: (if Playwright data available) tab-order consistency, trap aggregation, focus management patterns

### Progress Transparency

When used by the a11y-audit agent:
- **Announce start:** "Analyzing patterns across [N] scanned pages"
- **Announce completion:** "Cross-page analysis complete: [N] systemic patterns, [N] template patterns, overall score [score]/100 ([grade])"
- **On failure:** "Analysis incomplete: received findings from [N] of [M] expected pages. Proceeding with available data."

Return structured findings to the orchestrating audit agent.

