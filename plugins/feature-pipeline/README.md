# feature-pipeline

Claude Code plugin — multi-agent PM → Business Analyst → Architect → Developer pipeline for feature development.

## What it does

Trigger `/feature-pipeline <description>` to orchestrate four specialized agents in sequence:

1. **Product Manager** — writes PM doc (`agent-docs/project-{slug}.md`)
2. **Business Analyst** — writes functional spec (`agent-docs/analyst-{slug}.md`)
3. **Architect** — designs technical solution (`agent-docs/architect-{slug}.md`)
4. **Developer** — implements the feature (`agent-docs/dev-report-{slug}.md`)

Each stage has a Q&A loop (max 3 iterations) where agents ask the previous stage for clarification before advancing.

## Install

```bash
# Install agent definitions
bash hooks/install.sh

# Then install the plugin in Claude Code
claude plugin install .
```

## Uninstall

```bash
bash hooks/uninstall.sh
claude plugin uninstall feature-pipeline
```

## Files

```
.claude-plugin/
  plugin.json          plugin manifest
skills/
  feature-pipeline/
    SKILL.md           /feature-pipeline skill definition
agents/
  architect.md         Software Architect agent
  business-analyst.md  Business Analyst agent
  developer.md         Senior Developer agent
  product-manager.md   Product Manager agent
```
