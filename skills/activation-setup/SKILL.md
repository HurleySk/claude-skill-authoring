---
name: activation-setup
description: Use when the user wants to add prompt-based skill activation or guardrail enforcement to their project. Analyzes CLAUDE.md, recommends extractions, and orchestrates skill-engine setup.
argument-hint: "[analyze|setup|help]"
---

# Activation Setup — Skill Activation Infrastructure

You are an expert at analyzing Claude Code projects and setting up activation infrastructure. You analyze repos, recommend which CLAUDE.md sections to extract into targeted skills, and orchestrate the skill-engine plugin for hook installation.

## Prerequisites

- **skill-engine plugin** — required for `setup` command (not needed for `analyze`). If not installed:
  ```
  claude install hurleysk-marketplace/skill-engine
  ```

## Commands

### `analyze`

Analyze the current repo and produce an activation recommendation. Read-only — no changes made.

#### Step 1: Measure CLAUDE.md

Read `CLAUDE.md` in the project root. Measure line count and identify `##`-level section boundaries.

```
If CLAUDE.md < 100 lines:
  → Report: "CLAUDE.md is small enough that activation infrastructure isn't needed."
  → Stop.

If CLAUDE.md >= 100 and < 200 lines:
  → Note: "CLAUDE.md is moderate. Activation could help but isn't critical."

If CLAUDE.md >= 200 lines:
  → Note: "CLAUDE.md is large. Activation will significantly reduce context load."
```

#### Step 2: Catalog Sections

List every `##` section with:
- Header text
- Line count
- Assessment: KEEP (always relevant, <15 lines), CANDIDATE (>30 lines, topical), or SMALL (15-30 lines, borderline)

Sections that should typically KEEP:
- Project overview / description
- Safety rules / critical constraints
- Navigation guidance ("For Claude agents", "How to use this repo")
- Skill dispatch references

Sections that are typically CANDIDATE:
- API/CLI reference tables (>30 lines)
- Task format specifications
- Environment mapping / config tables
- Deployment workflows
- File/component maps

#### Step 3: Check Existing Hooks

```bash
ls .claude/hooks/ 2>/dev/null
cat .claude/settings.json 2>/dev/null | grep -c "PreToolUse"
```

Assess what enforcement already exists:
- **Custom PreToolUse hooks found** → recommend `activate.sh` only (not `enforce.sh`). Custom hooks are likely more sophisticated than generic skill-engine enforcement. Don't replace them.
- **No existing hooks** → recommend both `activate.sh` and `enforce.sh`
- **skill-engine already installed** → skip hook setup, focus on rules and extractions

#### Step 4: Scan for Rule Candidates

Check the repo for common file patterns that could trigger activation rules:

| Pattern | Check | Suggested Rule |
|---|---|---|
| `.sql` files | `find . -name "*.sql" -maxdepth 3 \| head -1` | SQL standards (suggest or block) |
| `pipeline/` dir | `ls pipeline/ 2>/dev/null` | Pipeline guidance (suggest) |
| `*.config` files | `find . -name "*.config" -maxdepth 3 \| head -1` | Config safety (warn) |
| `tasks/` dir | `ls tasks/ 2>/dev/null` | Task runner reference (suggest) |
| Large CLAUDE.md sections | From Step 2 analysis | Per-section suggest rules |

#### Step 5: Present Recommendation

Output a structured recommendation:

```markdown
## Activation Analysis

**CLAUDE.md:** X lines (LARGE/MODERATE/SMALL)
**Existing hooks:** Y PreToolUse hooks found / none

### Extraction Candidates
| Section | Lines | Action | Reason |
|---|---|---|---|
| ## Section Name | 45 | EXTRACT | Topical, >30 lines |
| ## Another Section | 12 | KEEP | Small, always relevant |
...

### Hook Recommendation
- Install: activate.sh only / both activate.sh + enforce.sh
- Reason: Custom hooks exist / no existing enforcement

### Draft Rules
(Show draft skill-rules.json entries for each extraction candidate)

### Estimated Impact
- CLAUDE.md: X lines → ~Y lines (Z% reduction)
- Skills created: N
```

Wait for user review before proceeding.

### `setup`

Execute the activation recommendation. Requires skill-engine plugin.

#### Step 1: Check Prerequisites

Verify skill-engine is installed:
```bash
ls ~/.claude/plugins/cache/hurleysk-marketplace/skill-engine/ 2>/dev/null
```

If not found:
```
The skill-engine plugin is required for activation setup.
Install it with: claude install hurleysk-marketplace/skill-engine
```
Stop and wait for the user to install it.

#### Step 2: Run Analysis

If `analyze` hasn't been run in this session, run it now. Present the recommendation and get user approval before proceeding.

#### Step 3: Install Hooks

Delegate to `/skill-engine:setup` for hook installation.

If custom hooks exist, tell the setup skill to install `activate.sh` only:
- Copy `activate.sh` + `lib/engine.js` to `.claude/hooks/skill-engine/`
- Add `UserPromptSubmit` entry to `.claude/settings.json`
- Do NOT install `enforce.sh` or add PreToolUse entries

If no custom hooks exist, let the setup skill install both.

#### Step 4: Extract Skills

For each approved extraction candidate:

1. Create `.claude/skills/{kebab-case-name}/` directory
2. Read the section from CLAUDE.md (identify by `##` header boundary)
3. Write to `.claude/skills/{name}/SKILL.md` with frontmatter:
   ```yaml
   ---
   name: {kebab-case-name}
   description: {one-line description}. Use when {trigger context}.
   ---
   ```
4. Copy section content verbatim — no rewriting

#### Step 5: Write skill-rules.json

Create `.claude/skills/skill-rules.json` with activation rules for each extracted skill.

Rule template:
```json
{
  "type": "domain",
  "enforcement": "suggest",
  "priority": "medium",
  "description": "{what this skill covers}",
  "skillPath": "./{name}/SKILL.md",
  "triggers": {
    "prompt": {
      "keywords": ["{key terms from the section}"],
      "intentPatterns": ["{regex matching user intent}"]
    }
  },
  "skipConditions": { "sessionOnce": true }
}
```

Guidelines for trigger quality:
- **Keywords**: 3-7 specific terms from the section content. Avoid generic words ("update", "fix").
- **Intent patterns**: 1-2 regex patterns matching how users ask for this topic. Use `(verb1|verb2).*?(noun1|noun2)` format.
- **Priority**: `high` for safety-related or frequently-needed skills, `medium` for general guidance, `low` for rarely-needed reference.

#### Step 6: Slim CLAUDE.md

For each extracted section:
1. Remove the section content from CLAUDE.md
2. Replace with a 2-3 line summary + pointer: "For details, see `.claude/skills/{name}/SKILL.md`."

Add a "Detailed Reference Skills" pointer section listing all extracted skills.

#### Step 7: Test Activation

Test each rule with a sample prompt:
```bash
echo '{"prompt":"<sample matching prompt>","session_id":"test","cwd":"'$(pwd)'"}' \
  | bash .claude/hooks/skill-engine/activate.sh
```

Verify each skill appears in the output with correct priority.

### `help`

Show the activation infrastructure overview:

```markdown
## Activation Infrastructure

**What it does:** The skill-engine plugin adds prompt-based activation to your project.
When you mention topics in your prompt, relevant skills from `.claude/skills/` are
suggested automatically. Optional guardrail enforcement can block or warn on risky
file edits.

**When to use it:**
- Your CLAUDE.md exceeds 200 lines (context overload)
- You want Claude to surface the right guidance based on what you're working on
- You need enforcement beyond what CLAUDE.md provides passively

**Workflow:**
1. `/skill-authoring:activation-setup analyze` — scan your repo, get recommendations
2. `/skill-authoring:activation-setup setup` — install hooks, extract skills, create rules
3. `/skill-engine:rules add` — add or modify individual rules later

**Requires:** `skill-engine` plugin (`claude install hurleysk-marketplace/skill-engine`)
```
