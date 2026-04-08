# Claude Code Skill Authoring

A [Claude Code](https://claude.ai/claude-code) skill for creating and publishing new skills to the [HurleySk marketplace](https://github.com/HurleySk/claude-plugins-marketplace).

## What It Does

Teaches Claude how to scaffold, write, and publish Claude Code skills following best practices. Includes CI/CD setup for automated version bumping and marketplace synchronization.

## Installation

### Via Marketplace

```bash
claude plugin marketplace add HurleySk/claude-plugins-marketplace
claude plugin install skill-authoring
```

### Direct

```bash
claude plugin add HurleySk/claude-skill-authoring
```

## Usage

```
/skill-authoring new my-awesome-skill
/skill-authoring add-to existing-plugin
/skill-authoring help
```

## License

MIT
