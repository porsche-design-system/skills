---
name: a11y-playwright
description: Browser accessibility testing using Playwright and @axe-core/playwright. Keyboard scans, contrast verification, and accessibility tree snapshots.
user-invocable: false
---

# Playwright Accessibility Testing

This skill is a procedure module for `a11y-audit` for browser-based accessibility testing using Playwright and `@axe-core/playwright`.

**Primary path: CLI / Node scripts** via the terminal (`npx playwright`, temporary `.mjs` runners).  
**Optional acceleration:** Playwright MCP tools (`run_playwright_*`) if present in the tool list — use them when available; otherwise always use CLI.

## Execution order

1. Detect: `npx playwright --version` and whether `@axe-core/playwright` is installed (project `node_modules` or installable via npx).
2. If MCP tools `run_playwright_keyboard_scan` (etc.) are available, they may be used instead of writing scripts for the same scan type.
3. Otherwise write and run Node scripts with Playwright (patterns below) via the terminal.
4. If Playwright cannot run, skip behavioral scans and report degraded status with install commands.

## Optional MCP tools (acceleration only)

| Tool | Purpose | Requires @axe-core/playwright |
|------|---------|------------------------------|
| `run_playwright_keyboard_scan` | Tab-order traversal, keyboard trap detection | No |
| `run_playwright_state_scan` | Click triggers, scan revealed content with axe-core | Yes |
| `run_playwright_viewport_scan` | Multi-viewport axe-core + touch target measurement | Yes |
| `run_playwright_contrast_scan` | Computed-style contrast ratio after CSS cascade | No |
| `run_playwright_a11y_tree` | Browser accessibility tree snapshot | No |

Do not treat missing MCP tools as failure when CLI works.

## CLI setup

```bash
# Check Playwright
npx playwright --version

# Prefer project deps; install if needed
npm install -D playwright @axe-core/playwright
npx playwright install chromium
```

### One-shot page axe scan (CLI-oriented)

Write a temporary script (e.g. `.a11y-playwright-scan.mjs`) and run with `node`:

```javascript
import { chromium } from 'playwright';
import AxeBuilder from '@axe-core/playwright';
import fs from 'node:fs';

const url = process.env.A11Y_URL || process.argv[2];
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto(url, { waitUntil: 'networkidle' });
const results = await new AxeBuilder({ page })
  .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
  .analyze();
fs.writeFileSync('ACCESSIBILITY-PLAYWRIGHT-AXE.json', JSON.stringify(results, null, 2));
console.log(JSON.stringify({ violations: results.violations.length, passes: results.passes.length }, null, 2));
await browser.close();
```

```bash
A11Y_URL="http://localhost:3000" node .a11y-playwright-scan.mjs
```

## @axe-core/playwright Patterns

### Full Page Scan

```javascript
import AxeBuilder from '@axe-core/playwright';

const results = await new AxeBuilder({ page })
  .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
  .analyze();
```

### Scoped Element Scan

```javascript
const results = await new AxeBuilder({ page })
  .include('.modal-content')
  .withTags(['wcag2a', 'wcag2aa'])
  .analyze();
```

### Single Rule Verification

```javascript
const results = await new AxeBuilder({ page })
  .include('#hero-image')
  .withRules(['image-alt'])
  .analyze();
expect(results.violations).toEqual([]);
```

### Scan After Interaction

```javascript
await page.click('[aria-expanded="false"]');
await page.waitForSelector('.accordion-content', { state: 'visible' });
const results = await new AxeBuilder({ page })
  .include('.accordion-content')
  .analyze();
```

## Keyboard Traversal Patterns

### Record Tab Sequence

```javascript
const tabStops = [];
for (let i = 0; i < maxTabs; i++) {
  await page.keyboard.press('Tab');
  const info = await page.evaluate(() => {
    const el = document.activeElement;
    return {
      tagName: el?.tagName,
      role: el?.getAttribute('role'),
      name: el?.getAttribute('aria-label') || el?.textContent?.trim().slice(0, 50),
      id: el?.id,
      tabIndex: el?.tabIndex
    };
  });
  tabStops.push(info);
}
```

### Detect Keyboard Traps

A keyboard trap is detected when the same element receives focus after consecutive Tab presses:

```javascript
let trapCount = 0;
let lastSelector = '';
for (const stop of tabStops) {
  const currentSelector = `${stop.tagName}#${stop.id}`;
  if (currentSelector === lastSelector) {
    trapCount++;
    if (trapCount >= 3) { /* TRAP DETECTED */ }
  } else {
    trapCount = 0;
  }
  lastSelector = currentSelector;
}
```

### Focus Management After Modal Open

```javascript
await page.click('[data-modal-trigger]');
await page.waitForSelector('[role="dialog"]', { state: 'visible' });
const focusedRole = await page.evaluate(() =>
  document.activeElement?.closest('[role="dialog"]') ? 'inside-dialog' : 'outside-dialog'
);
// focusedRole should be 'inside-dialog'
```

## Focus Management Test Templates

### Modal Focus Trap Test

```javascript
test('modal traps focus correctly', async ({ page }) => {
  await page.goto(url);
  await page.click('[data-open-modal]');
  await page.waitForSelector('[role="dialog"]', { state: 'visible' });

  // Focus should be inside the dialog
  const inDialog = await page.evaluate(() =>
    document.activeElement?.closest('[role="dialog"]') !== null
  );
  expect(inDialog).toBe(true);

  // Tab through dialog — should not escape
  for (let i = 0; i < 20; i++) {
    await page.keyboard.press('Tab');
    const stillInDialog = await page.evaluate(() =>
      document.activeElement?.closest('[role="dialog"]') !== null
    );
    expect(stillInDialog).toBe(true);
  }

  // Escape should close and return focus to trigger
  await page.keyboard.press('Escape');
  const focusedId = await page.evaluate(() => document.activeElement?.id);
  expect(focusedId).toBe('modal-trigger-id');
});
```

### Skip Link Test

```javascript
test('skip link moves focus to main content', async ({ page }) => {
  await page.goto(url);
  await page.keyboard.press('Tab'); // Focus skip link
  await page.keyboard.press('Enter'); // Activate it
  const focusedId = await page.evaluate(() => document.activeElement?.id);
  expect(focusedId).toBe('main-content');
});
```

## CI Integration

### GitHub Actions with Playwright

```yaml
name: Accessibility Tests
on: [push, pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - name: Start dev server
        run: npm run dev &
        env:
          CI: true
      - name: Wait for server
        run: npx wait-on http://localhost:3000 --timeout 30000
      - name: Run accessibility tests
        run: npx playwright test tests/a11y/
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: a11y-test-results
          path: test-results/
```

### Playwright Config for Accessibility Tests

```javascript
// playwright.config.js (a11y section)
export default {
  testDir: './tests/a11y',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    // Use Chromium only — @axe-core/playwright is Chromium-validated
    browserName: 'chromium',
  },
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
};
```

## Graceful Degradation

### Detection Pattern

```javascript
let _playwrightAvailable = null;
async function isPlaywrightAvailable() {
  if (_playwrightAvailable !== null) return _playwrightAvailable;
  try {
    await import('playwright');
    _playwrightAvailable = true;
  } catch {
    _playwrightAvailable = false;
  }
  return _playwrightAvailable;
}
```

### Degradation Matrix

| Playwright | @axe-core/playwright | Available Scans |
|------------|---------------------|-----------------|
| Yes | Yes | All 5 tools (keyboard, state, viewport, contrast, tree) |
| Yes | No | 3 tools (keyboard, contrast, tree) |
| No | — | None — fall back to code review + axe-core CLI |

### User-Facing Messages

When unavailable:

```yaml
Playwright not installed. Behavioral testing (keyboard traversal, dynamic states,
responsive viewport, rendered contrast) is unavailable.

Install: npm install -D playwright @axe-core/playwright && npx playwright install chromium
```

When partially available:

```yaml
@axe-core/playwright not installed. State scanning and viewport scanning are
unavailable. Keyboard, contrast, and accessibility tree scans will proceed.

Install: npm install -D @axe-core/playwright
```

## WCAG Coverage Map

| WCAG SC | Description | Playwright Tool |
|---------|-------------|-----------------|
| 1.3.1 | Info and Relationships | a11y tree, state scan |
| 1.4.3 | Contrast (Minimum) | contrast scan |
| 1.4.6 | Contrast (Enhanced) | contrast scan |
| 1.4.10 | Reflow | viewport scan |
| 2.1.1 | Keyboard | keyboard scan |
| 2.1.2 | No Keyboard Trap | keyboard scan |
| 2.4.3 | Focus Order | keyboard scan |
| 2.4.7 | Focus Visible | keyboard scan |
| 2.5.5 | Target Size (Enhanced) | viewport scan |
| 2.5.8 | Target Size (Minimum) | viewport scan |
| 4.1.2 | Name, Role, Value | a11y tree, state scan |

## Playwright Scanner Procedures

## Authoritative Sources

- **WCAG 2.2 Specification** — https://www.w3.org/TR/WCAG22/
- **axe-core Rules** — https://github.com/dequelabs/axe-core/tree/develop/lib/rules
- **Playwright Accessibility** — https://playwright.dev/docs/accessibility-testing
- **@axe-core/playwright** — https://github.com/dequelabs/axe-core-npm/tree/develop/packages/playwright

This skill is a procedure module for `a11y-audit` for behavioral accessibility scanning. This skill does not edit source files, configuration, or documentation; return structured findings.

---

## Capabilities

### 1. Full Behavioral Scan

When given a URL and scan profile, execute the following tests in order (CLI scripts or optional MCP equivalents):

1. **Keyboard Flow Mapping** — Tab sequence, traps, unreachable interactive elements (keyboard patterns above).
2. **Dynamic State Scanning** — Click triggers (accordions, menus, modals, tabs); run AxeBuilder on revealed states (`@axe-core/playwright` required).
3. **Responsive Viewport Scanning** — Set viewport widths [320, 768, 1024, 1440]; check reflow/horizontal scroll and touch target sizes; run axe per viewport when `@axe-core/playwright` is available.
4. **Rendered Contrast Verification** — Evaluate computed foreground/background colors after CSS cascade; flag ratios below WCAG thresholds.
5. **Accessibility Tree Snapshot** — Use Playwright accessibility snapshot / `page.accessibility.snapshot()` (or MCP `run_playwright_a11y_tree`) for landmark/heading/role/name checks.

### Viewport + touch targets (CLI sketch)

```javascript
const widths = [320, 768, 1024, 1440];
for (const width of widths) {
  await page.setViewportSize({ width, height: 800 });
  await page.goto(url, { waitUntil: 'networkidle' });
  const overflow = await page.evaluate(() => document.documentElement.scrollWidth > document.documentElement.clientWidth);
  const smallTargets = await page.evaluate(() => {
    const min = 24; // WCAG 2.5.8 CSS px
    return [...document.querySelectorAll('a, button, input, select, textarea, [role="button"]')]
      .map((el) => {
        const r = el.getBoundingClientRect();
        return { name: el.tagName, w: r.width, h: r.height };
      })
      .filter((t) => t.w > 0 && t.h > 0 && (t.w < min || t.h < min));
  });
  // record overflow + smallTargets per width
}
```

### 2. Focus Management Tests

- Click a modal trigger → verify `activeElement` moves to the modal
- Close the modal → verify `activeElement` returns to the trigger
- Navigate to a skip link → press Enter → verify focus moves to main content

### 3. Targeted Scans

- **keyboard-only** / **states-only** / **viewport-only** / **contrast-only** / **tree-only** — run only that procedure

## Output Contract

Return a structured findings object with all scan results:

```text
PLAYWRIGHT BEHAVIORAL SCAN: {url}
=====================================

KEYBOARD FLOW:
  Total Tab Stops: {n}
  Keyboard Traps: {n}
  Unreachable Interactive Elements: {n}
  [Full tab sequence listing]

DYNAMIC STATE SCAN:
  Triggers Tested: {n}
  Violations in Dynamic States: {n}
  [Per-trigger results with axe-core violations]

VIEWPORT SCAN:
  Viewports Tested: 320px, 768px, 1024px, 1440px
  Reflow Failures: {n}
  Undersized Touch Targets: {n}
  [Per-viewport results]

CONTRAST SCAN:
  Elements Analyzed: {n}
  Contrast Failures: {n}
  [Per-element contrast details]

ACCESSIBILITY TREE:
  Total Nodes: {n}
  Role Distribution: {role counts}
  [Tree structure]

BEHAVIORAL CONFIDENCE: {High|Medium|Low}
  (High = all 5 scans completed successfully)
  (Medium = 3-4 scans completed)
  (Low = 1-2 scans completed, others failed)
```

## Graceful Degradation

If Playwright is not installed:
- Report that behavioral scans are unavailable
- List: `npm install -D playwright @axe-core/playwright && npx playwright install chromium`
- Return a "degraded" status so the audit can proceed with static/runtime-only analysis

If @axe-core/playwright is not installed but Playwright is:
- Run keyboard, contrast, and accessibility tree scans
- Skip axe-dependent state/viewport enrichment
- Report partial results with a note about the missing dependency

## WCAG Coverage

| Procedure | WCAG Success Criteria |
|------|----------------------|
| Keyboard Scan | 2.1.1 Keyboard, 2.1.2 No Keyboard Trap, 2.4.3 Focus Order |
| State Scan | All SC in dynamic states (1.3.1, 4.1.2, etc.) |
| Viewport Scan | 1.4.10 Reflow, 2.5.5 Target Size Enhanced, 2.5.8 Target Size Minimum |
| Contrast Scan | 1.4.3 Contrast Minimum, 1.4.6 Contrast Enhanced |
| A11y Tree | Structural SC (1.3.1, 2.4.6, 4.1.2) |

## Playwright Verifier Procedures

## Authoritative Sources

- **WCAG 2.2 Specification** — https://www.w3.org/TR/WCAG22/
- **axe-core Rules** — https://github.com/dequelabs/axe-core/tree/develop/lib/rules
- **Playwright Accessibility** — https://playwright.dev/docs/accessibility-testing
- **@axe-core/playwright** — https://github.com/dequelabs/axe-core-npm/tree/develop/packages/playwright

This skill is a procedure module for fix verification (via `a11y-issue-fixer` / `a11y-audit`). This skill does not edit source files; return structured findings. Apply it after fixes to verify resolution without regressions.

---

## Verification Workflow

When given fix details, follow this sequence:

### Step 1: Receive Fix Context

- `fix_number` — Sequential number in the fix batch
- `rule_id` — axe-core rule ID (e.g., `color-contrast`, `button-name`)
- `selector` — CSS selector of the fixed element
- `url` — Dev server URL
- `fix_type` — contrast | keyboard | aria | structure | state | viewport

### Step 2: Run Targeted Verification

Use CLI scripts (or MCP if available):

| Fix Type | Verification | What to Check |
|----------|------------------|---------------|
| `contrast` | contrast scan script | Computed colors meet threshold |
| `keyboard` | keyboard scan script | Element in tab order; no traps |
| `aria` / `structure` | a11y tree / axe include(selector) | Role, name, state, headings/landmarks |
| `state` | state scan script | Dynamic content accessible after interaction |
| `viewport` | viewport scan script | Reflow and touch targets |

### Step 3: Determine Verdict

- **PASS** — Original violation absent; no new violations
- **FAIL** — Original violation still present
- **REGRESSION** — Original fixed but new violations introduced

### Step 4: Report Results

```text
FIX VERIFICATION #{fix_number}
Rule: {rule_id}
Selector: {selector}
Verdict: {PASS|FAIL|REGRESSION}
```

## Test Code Generation

After a verified PASS, generate a Playwright regression test using AxeBuilder / keyboard patterns from this skill.

## Batch Verification

```text
VERIFICATION SUMMARY
====================
Total Fixes: {n}
Verified PASS: {n}
FAIL: {n}
REGRESSION: {n}
Skipped (no Playwright): {n}
```

