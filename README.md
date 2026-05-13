# Vibe Features

Vibe Features is a Claude Code and Codex plugin marketplace for feature-development workflows.

## Plugins

### feature-pipeline

Multi-agent PM -> Business Analyst -> Architect -> Developer pipeline for feature development.

Trigger `/feature-pipeline <description>` to create a product document, functional spec, technical architecture, and implementation report while moving through bounded clarification loops between stages.

See [plugins/feature-pipeline/README.md](plugins/feature-pipeline/README.md) for full usage and file layout details.

## Install

### Claude Code

Add this repository as a Claude Code plugin marketplace, then install the plugin:

```bash
/plugin marketplace add CrazyApple888/vibe-features
/plugin install feature-pipeline
```

### Codex

Add this repository as a Codex plugin marketplace:

```bash
codex plugin marketplace add CrazyApple888/vibe-features
```

Then install `feature-pipeline` from the `Vibe Features` marketplace in the Codex app.

To update the marketplace later:

```bash
codex plugin marketplace upgrade vibe-features
```

## Local Development

Clone the repository and add the local checkout as a marketplace source:

```bash
git clone https://github.com/CrazyApple888/vibe-features.git
codex plugin marketplace add ./vibe-features
```

Marketplace metadata is stored in:

- `.agents/plugins/marketplace.json`
- `.claude-plugin/marketplace.json`

Plugin source lives under `plugins/`.

## License

MIT. See [LICENSE](LICENSE).
