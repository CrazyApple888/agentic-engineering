# feature-pipeline

Claude Code and Codex plugin — multi-agent PM → Business Analyst → Architect → Developer pipeline for feature development.

## What it does

Trigger `/feature-pipeline <description>` to orchestrate four specialized agents in sequence:

1. **Product Manager** — writes PM doc (`agent-docs/project-{slug}.md`)
2. **Business Analyst** — writes functional spec (`agent-docs/analyst-{slug}.md`)
3. **Architect** — designs technical solution (`agent-docs/architect-{slug}.md`)
4. **Developer** — implements the feature (`agent-docs/dev-report-{slug}.md`)

Each stage has a Q&A loop (max 3 iterations) where agents ask the previous stage for clarification before advancing.

## Install

The plugin is shared from this GitHub repository:

```text
CrazyApple888/vibe-features
```

### Claude Code

Add the repository as a Claude Code plugin marketplace, then install the plugin:

```bash
/plugin marketplace add CrazyApple888/vibe-features
/plugin install feature-pipeline
```

### Codex

Add the repository as a Codex plugin marketplace:

```bash
codex plugin marketplace add CrazyApple888/vibe-features
```

Then install `feature-pipeline` from the `Vibe Features` marketplace in the Codex app.

To update the marketplace later:

```bash
codex plugin marketplace upgrade vibe-features
```

The repository includes the Codex metadata Codex needs to discover the plugin:

- `plugins/feature-pipeline/.codex-plugin/plugin.json`
- `.agents/plugins/marketplace.json`

If you are developing locally before publishing, clone the repository and add the local checkout as the Codex marketplace source:

```bash
git clone https://github.com/CrazyApple888/vibe-features.git
codex plugin marketplace add ./vibe-features
```

## Uninstall

```bash
claude plugin uninstall feature-pipeline
```

## Files

```
.claude-plugin/
  plugin.json          plugin manifest
.codex-plugin/
  plugin.json          Codex plugin manifest
skills/
  feature-pipeline/
    SKILL.md           /feature-pipeline skill definition
agents/
  architect.md         Software Architect agent
  business-analyst.md  Business Analyst agent
  developer.md         Senior Developer agent
  product-manager.md   Product Manager agent
```
