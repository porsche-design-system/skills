# Web Accessibility Audit

Deep-dive web accessibility audit package for GitHub Copilot, Claude Code, and Cursor: **one agent** (`a11y-audit`) that loads domain and ops skills on demand, plus initiation/fix prompts and always-on instructions.

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

After `apm install`, start an audit with a prompt. Every entry point uses the **a11y-audit** agent; domain and ops skills are loaded by the agent (skills are `user-invocable: false` so they do not clutter the `/` menu).

| Start with | Use when |
|------------|----------|
| `a11y-audit` | Full single-page audit (axe + code review) |
| `a11y-quick-check` | Fast axe-only triage |
| `a11y-audit-multi-page` | Compare several pages |
| `a11y-component-library-audit` | Audit a component library directory |
| `a11y-lead` | Review specific UI files (a11y-audit review mode) |
| `a11y-fix-issues-interactive` | Interactive remediation from an audit report |
| `a11y-fix` | Apply auto-fixable and guided fixes (a11y-audit + `a11y-issue-fixer` skill) |

## Architecture

```text
User → a11y-* prompts → a11y-audit (sole agent)
                              └─ Read domain + ops skills by phase (0–12)
```

### Phase map

| Phase | Focus | Skills |
|-------|--------|--------|
| 0 | Discovery | `a11y-framework`, `a11y-web-scanning` |
| 1 | Structure | `a11y-alt-text-headings`, `a11y-text-quality`, `a11y-media` (when media present) |
| 2 | Keyboard | `a11y-keyboard`, `a11y-modal` |
| 3 | Forms | `a11y-forms` |
| 4 | Visual | `a11y-contrast`, `a11y-design-system` (when tokens) |
| 5 | Live regions | `a11y-live-regions` |
| 6 | ARIA widgets | `a11y-aria` |
| 7 | Tables | `a11y-tables` |
| 8 | Links | `a11y-links` |
| 9 | Scanners | `a11y-web-scanning`, `a11y-lighthouse` |
| 10 | Behavioral | `a11y-playwright` (CLI primary; MCP optional) |
| 11 | Report | `a11y-severity-scoring`, `a11y-help-url-reference`, testing skills |
| 12 | Fix / verify | `a11y-issue-fixer` |

**Thoroughness:** Quick = 0→1→9→11; Standard = 0–11; Deep dive = Standard + `a11y-cognitive` + quiet batching + design-system when tokens exist.

## Inventory

### Agents (1)

| Agent | Role |
|-------|------|
| `a11y-audit` | Sole orchestrator: full audit, file review, fixes, export, scanners via skills |

### Domain skills (15)

| Skill | Purpose |
|-------|---------|
| `a11y-aria` | ARIA roles, states, properties |
| `a11y-alt-text-headings` | Images, alt text, headings, landmarks, language of page/parts |
| `a11y-media` | Video/audio captions, descriptions, media alternatives (WCAG 1.2.x) |
| `a11y-keyboard` | Tab order and focus |
| `a11y-modal` | Dialogs and overlays |
| `a11y-forms` | Forms, labels, validation |
| `a11y-contrast` | Color contrast and visual design |
| `a11y-live-regions` | Dynamic content and status |
| `a11y-tables` | Data tables and grids |
| `a11y-links` | Link text quality |
| `a11y-text-quality` | Non-visual text quality |
| `a11y-cognitive` | Cognitive SC and COGA |
| `a11y-design-system` | Token and focus-ring contrast |
| `a11y-testing-coach` | Testing guidance |
| `a11y-wcag-guide` | WCAG 2.2 criteria |

### Ops skills (9)

| Skill | Purpose |
|-------|---------|
| `a11y-web-scanning` | URL crawling, axe-core CLI, page inventory |
| `a11y-severity-scoring` | Severity, grade scoring, cross-page analysis (sole scoring source) |
| `a11y-framework` | Framework-specific patterns |
| `a11y-playwright` | Playwright CLI behavioral scan + fix verification (MCP optional) |
| `a11y-testing-strategy` | Manual and automated strategy |
| `a11y-help-url-reference` | WCAG/ARIA reference URLs |
| `a11y-lighthouse` | Lighthouse a11y integration |
| `a11y-issue-fixer` | Apply guided / auto fixes (sole fix policy) |
| `a11y-csv-reporter` | CSV/JSON export of findings |

### Prompts (7) — audit initiation + fix

| Prompt file | Invokes as | Purpose |
|-------------|------------|---------|
| `a11y-audit.prompt.md` | `a11y-audit` | Full single-page audit |
| `a11y-quick-check.prompt.md` | `a11y-quick-check` | Fast axe-only check |
| `a11y-audit-multi-page.prompt.md` | `a11y-audit-multi-page` | Multi-page scorecard |
| `a11y-component-library-audit.prompt.md` | `a11y-component-library-audit` | Component library audit |
| `a11y-lead.prompt.md` | `a11y-lead` | File/component review via a11y-audit |
| `a11y-fix-issues.prompt.md` | `a11y-fix-issues-interactive` | Interactive fix from audit report |
| `a11y-fix.prompt.md` | `a11y-fix` | Apply fixes via a11y-audit + `a11y-issue-fixer` skill |

### Instructions (7) — always-on coding rules

| Instruction | Applies to | Purpose |
|-------------|------------|---------|
| `a11y-baseline` | HTML/JSX/Vue/Svelte/Astro | WCAG 2.2 AA baseline |
| `a11y-semantic-html` | HTML/JSX/Vue/Svelte/Astro | Landmarks and native structure |
| `a11y-aria-patterns` | HTML/JSX/Vue/Svelte/Astro | ARIA widgets and keyboard patterns |
| `a11y-css` | CSS/SCSS/Less | Focus, motion, contrast |
| `a11y-testing` | Test/spec files | Accessible query and keyboard tests |
| `a11y-finding-reliability` | Agent/markdown files | Structured findings format |
| `a11y-agent-terminology` | Agent/markdown files | Shared severity and role terms |

## Updating from accessibility-agents

Historical extract tooling may live outside this package. Prefer updating skills and the agent in-tree. If an extract script exists in the monorepo or a sibling `accessibility-agents` checkout, use that project's docs; this package does not ship a required extract script.

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

Compare/CI-setup prompts, enforcement hooks, custom Playwright MCP servers, and `.a11y-web-config.json` are deferred. Phase 10 uses **Playwright CLI** (`npx playwright` / `@axe-core/playwright`); MCP tools are optional acceleration only. Use the accessibility-agents web-audit installer if you need hooks/MCP today.

## License

MIT — see the [repository LICENSE](../../LICENSE).
