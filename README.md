# Porsche Design System Skills

Monorepo of shareable [APM](https://microsoft.github.io/apm/) packages for AI agent skills.

## Requirements

- [APM](https://microsoft.github.io/apm/) (`curl -sSL https://aka.ms/apm-unix | sh`)

## Packages

| Package | Path | Description | Install |
|---------|------|-------------|---------|
| `web-accessibility-audit` | [`packages/web-accessibility-audit`](packages/web-accessibility-audit) | Deep-dive web accessibility audit skills, agents, prompts, and instructions | `apm install porsche-design-system/skills/packages/web-accessibility-audit` |

## Install a package

```bash
apm install porsche-design-system/skills/packages/web-accessibility-audit#v1.0.0
```

Or declare it in your project's `apm.yml`:

```yaml
dependencies:
  apm:
    - git: https://github.com/porsche-design-system/skills.git
      path: packages/web-accessibility-audit
      ref: v1.0.0
```

Local development:

```bash
apm install ./packages/web-accessibility-audit
```

## Adding a package

1. Create `packages/<name>/apm.yml` and `packages/<name>/.apm/` (skills, agents, prompts, instructions, etc.).
2. Document the package in this README and add a package-level `README.md`.
3. Validate from inside the package directory:

```bash
cd packages/<name>
apm compile --validate -t copilot,claude,cursor
```

4. Optionally smoke-test with `apm install ./packages/<name>` from a scratch project.

## Attribution

Web accessibility content is adapted from [Community-Access/accessibility-agents](https://github.com/Community-Access/accessibility-agents), maintained in the Porsche fork [porsche-design-system/accessibility-agents](https://github.com/porsche-design-system/accessibility-agents).

## License

MIT — see [LICENSE](LICENSE).
