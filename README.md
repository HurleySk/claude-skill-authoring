# Claude Code Skill Authoring

A [Claude Code](https://claude.ai/claude-code) skill set for creating, reviewing, and publishing skills to the [HurleySk marketplace](https://github.com/HurleySk/claude-plugins-marketplace).

## What It Does

Two complementary skills:

- **skill-authoring** — Scaffold, write, and publish Claude Code skills following best practices. Includes CI/CD setup for automated version bumping and marketplace synchronization.
- **skill-review** — Comprehensive 8-dimension quality assessment for skills. Evaluates coverage, platform compatibility, resilience, cross-skill consistency, usability, workflow quality, security posture, and documentation. Produces graded reports with prioritized enhancement recommendations.

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

## Skills

### `skill-authoring` — Create & Publish Skills

| Command | Description |
|---------|-------------|
| `/skill-authoring new <name>` | Create a new skill as a standalone plugin repo |
| `/skill-authoring add-to <plugin>` | Add a skill to an existing multi-skill plugin |
| `/skill-authoring help` | Show available commands |

### `skill-review` — Assess Skill Quality

| Command | Description |
|---------|-------------|
| `/skill-authoring:skill-review assess [path]` | Full 8-dimension assessment with graded report |
| `/skill-authoring:skill-review checklist [path]` | Quick pass/fail validation checklist |
| `/skill-authoring:skill-review help` | Show available commands |

**Assessment dimensions:**
1. Coverage completeness (tools, operations, patterns)
2. Platform compatibility (Windows, macOS, Linux)
3. Resilience & failure modes (fail-open vs fail-closed)
4. Cross-skill consistency (shared config, terminology, handoffs)
5. Usability & lifecycle (status, test, disable, uninstall)
6. Workflow quality (concrete steps, error recovery)
7. Security posture (injection, bypass vectors, false confidence)
8. Documentation quality (prerequisites, limitations, lessons)

**Grades:** A (production-grade), B (solid with gaps), C (functional but incomplete), D (needs work)

## License

MIT
