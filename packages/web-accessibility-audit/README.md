# Web Accessibility Audit

Deep-dive web accessibility audit skills, specialist agents, and audit-initiation prompts for GitHub Copilot, Claude Code, and Cursor.

Adapted from the [porsche-design-system/accessibility-agents](https://github.com/porsche-design-system/accessibility-agents) web-audit bundle.

## Install

```bash
apm install porsche-design-system/skills/packages/web-accessibility-audit#v1.0.0
```

Or in your project's `apm.yml`:

```yaml
dependencies:
  apm:
    - git: https://github.com/porsche-design-system/skills.git
      path: packages/web-accessibility-audit
      ref: v1.0.0
```

## Usage

After `apm install`, start an audit with a prompt (Copilot `/` commands / Claude commands) or invoke an agent:

| Start with | Use when |
|------------|----------|
| `web-accessibility-wizard` | Full single-page audit (axe + code review) |
| `quick-web-check` | Fast axe-only triage |
| `audit-web-multi-page` | Compare several pages |
| `component-library-audit` | Audit a component library directory |
| `review-a11y` / `@accessibility-lead` | Review specific UI files with specialists |
| `fix-web-issues-interactive` | Interactive remediation from an audit report |
| `fix-web-issues` / `@web-issue-fixer` | Apply auto-fixable and guided fixes |

Specialists cover ARIA, keyboard, forms, contrast, modals, tables, links, cognitive accessibility, design-system tokens, testing, WCAG reference, and scanner bridges (axe, Lighthouse, Playwright).

## Inventory

### Agents (23)

| Agent | Role |
|-------|------|
| `accessibility-lead` | Orchestrator and final review |
| `web-accessibility-wizard` | Guided multi-phase web audit |
| `aria-specialist` | ARIA roles, states, properties |
| `alt-text-headings` | Images, alt text, headings, landmarks |
| `keyboard-navigator` | Tab order and focus |
| `modal-specialist` | Dialogs and overlays |
| `forms-specialist` | Forms, labels, validation |
| `contrast-master` | Color contrast and visual design |
| `live-region-controller` | Dynamic content and status |
| `tables-data-specialist` | Data tables and grids |
| `link-checker` | Link text quality |
| `testing-coach` | Testing guidance |
| `wcag-guide` | WCAG 2.2 criteria |
| `text-quality-reviewer` | Non-visual text quality |
| `cognitive-accessibility` | Cognitive SC and COGA |
| `design-system-auditor` | Token and focus-ring contrast |
| `cross-page-analyzer` | Multi-page audit scoring |
| `web-issue-fixer` | Remediation |
| `web-csv-reporter` | CSV reporting |
| `scanner-bridge` | axe / scanner integration |
| `lighthouse-bridge` | Lighthouse integration |
| `playwright-scanner` | Playwright scanning |
| `playwright-verifier` | Playwright verification |

### Skills (10)

| Skill | Purpose |
|-------|---------|
| `web-scanning` | URL crawling, axe-core CLI, page inventory |
| `web-severity-scoring` | Severity and grade scoring |
| `framework-accessibility` | Framework-specific patterns |
| `playwright-testing` | Playwright a11y verification |
| `testing-strategy` | Manual and automated strategy |
| `help-url-reference` | WCAG/ARIA reference URLs |
| `github-a11y-scanner` | GitHub Action scanner setup |
| `lighthouse-scanner` | Lighthouse a11y integration |
| `cognitive-accessibility` | Cognitive criteria guidance |
| `design-system` | Token and contrast patterns |

### Prompts (7) — audit initiation + fix

| Prompt file | Invokes as | Purpose |
|-------------|------------|---------|
| `web-accessibility-wizard.prompt.md` | `web-accessibility-wizard` | Full single-page audit |
| `quick-web-check.prompt.md` | `quick-web-check` | Fast axe-only check |
| `audit-web-multi-page.prompt.md` | `audit-web-multi-page` | Multi-page scorecard |
| `component-library-audit.prompt.md` | `component-library-audit` | Component library audit |
| `accessibility-lead.prompt.md` | `review-a11y` | File/component review via lead |
| `fix-web-issues.prompt.md` | `fix-web-issues-interactive` | Interactive fix from audit report |
| `web-issue-fixer.prompt.md` | `fix-web-issues` | Apply fixes via web-issue-fixer agent |

Specialist, compare, and CI-setup prompts from accessibility-agents are not included.

## Updating from accessibility-agents

From the monorepo root:

```bash
node scripts/extract-from-accessibility-agents.mjs \
  --source /path/to/accessibility-agents
```

Defaults: `--source` from `A11Y_AGENTS_SOURCE` or `../APM/accessibility-agents`; `--package packages/web-accessibility-audit`.

## Package author checks

```bash
cd packages/web-accessibility-audit
apm compile --validate -t copilot,claude,cursor
```

Smoke-test as a consumer:

```bash
# from a scratch project
apm init
apm install /absolute/path/to/skills/packages/web-accessibility-audit -t copilot,claude,cursor
```

## Not included in v1

Specialist, compare, and CI-setup prompts, always-on instructions, enforcement hooks, MCP servers, and `.a11y-web-config.json` are deferred. Use the accessibility-agents web-audit installer if you need those today.

## License

MIT — see the [repository LICENSE](../../LICENSE).
