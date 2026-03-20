# Claude Skills

Custom skills for Claude Code: product requirements discovery and iterative improvement loops.

## Plugins

### product-requirements

Product requirements discovery and PRD generation through structured interviews. Conducts a thorough requirements interview, writes a comprehensive PRD, has it reviewed by a second expert, and creates an AGENTS.md file.

- **Command:** `/product-requirements`
- **Auto-triggers on:** "requirements", "PRD", "product spec", "app idea", describing what you want to build

### improve-loop

Automated iterative improvement loop for any project. Reads project docs, proposes an evaluation strategy with optional expert panel, then runs a generate-evaluate-fix loop to iteratively improve output quality.

- **Command:** `/improve-loop`
- **Auto-triggers on:** "improve loop", "make it better", "iterate on quality", "the output isn't good enough"

## Installation

Register this repo as a Claude Code plugin marketplace:

```
/plugin marketplace add DavidJBianco/claude-skills
```

Then install individual plugins:

```
/plugin install product-requirements@claude-skills
/plugin install improve-loop@claude-skills
```

Or use `/plugin > Discover` to browse and install.

## License

MIT
