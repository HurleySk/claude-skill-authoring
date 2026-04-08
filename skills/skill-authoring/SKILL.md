---
name: skill-authoring
description: Create, write, and publish Claude Code skills to the HurleySk marketplace. Use when the user wants to create a new skill, add a skill to an existing plugin, or publish to the marketplace.
argument-hint: "[new|add-to|help] [skill-name]"
---

# Skill Authoring for HurleySk Marketplace

You are an expert at authoring Claude Code skills. Follow the patterns, conventions, and best practices below to create skills that are executable agent workflows — not documentation dumps.

## Marketplace Context

- **Marketplace repo**: `HurleySk/claude-plugins-marketplace` — the catalog users add with `claude plugin marketplace add`
- **Marketplace owner**: Samuel Hurley (GitHub: HurleySk) — has push access to all repos
- **CI/CD**: Every plugin repo has a `version-bump.yml` workflow that auto-increments the patch version and dispatches to the marketplace repo, which auto-updates `marketplace.json`

## Commands

Based on the argument provided:

### `new [skill-name]`

Create a new skill as a standalone plugin repo. Execute each step in order.

#### Step 1: Create the Repo

```bash
gh repo create HurleySk/claude-<SKILL_NAME>-skill --public --clone --description "<one-line description>"
cd claude-<SKILL_NAME>-skill
```

Replace `<SKILL_NAME>` with the skill name (lowercase, hyphens). Remember this directory as `$SKILL_REPO`.

#### Step 2: Scaffold the Plugin Structure

Create the following directory structure:

```
$SKILL_REPO/
  .claude-plugin/
    plugin.json
  .github/
    workflows/
      version-bump.yml
  skills/
    <SKILL_NAME>/
      SKILL.md
  LICENSE
  README.md
```

**2a. Create `plugin.json`:**

```json
{
  "name": "<SKILL_NAME>",
  "version": "1.0.0",
  "description": "<What the skill does — be specific, this is used for discovery>",
  "author": {
    "name": "Samuel Hurley",
    "url": "https://github.com/HurleySk"
  },
  "license": "MIT",
  "homepage": "https://github.com/HurleySk/claude-<SKILL_NAME>-skill",
  "repository": "https://github.com/HurleySk/claude-<SKILL_NAME>-skill",
  "keywords": ["<relevant>", "<tags>"]
}
```

**2b. Create `SKILL.md` with frontmatter:**

```yaml
---
name: <SKILL_NAME>
description: <When to trigger — be specific about the user intent this skill serves>
argument-hint: "[cmd1|cmd2|help] [args]"
---
```

Then write the skill body following the best practices in the "Writing Good Skills" section below.

**2c. Copy `version-bump.yml`:**

Read the workflow from any existing HurleySk skill repo (e.g., `HurleySk/claude-xrmtoolbox-skill`) at `.github/workflows/version-bump.yml` and copy it verbatim. The workflow is generic — it reads the plugin name and version from `plugin.json` dynamically. Confirm the `branches:` trigger matches the repo's default branch (`main` for new repos).

**2d. Create `LICENSE`:**

MIT license, copyright Samuel Hurley.

**2e. Create `README.md`:**

```markdown
# <Display Name>

A [Claude Code](https://claude.ai/claude-code) skill for <what it does>.

## Installation

### Via Marketplace

\```bash
claude plugin marketplace add HurleySk/claude-plugins-marketplace
claude plugin install <SKILL_NAME>
\```

### Direct

\```bash
claude plugin add HurleySk/claude-<SKILL_NAME>-skill
\```

## Commands

| Command | Description |
|---------|-------------|
| `/<SKILL_NAME> cmd1` | ... |
| `/<SKILL_NAME> cmd2` | ... |
| `/<SKILL_NAME> help` | Show available commands |

## License

MIT
```

#### Step 3: Set Up CI/CD

**3a. Add the `MARKETPLACE_PAT` secret:**

```bash
gh secret set MARKETPLACE_PAT --repo HurleySk/claude-<SKILL_NAME>-skill
```

This will prompt for the PAT value. Ask the user to paste their GitHub PAT (classic, `repo` scope). If they don't have one, direct them to https://github.com/settings/tokens to create one.

**3b. Initial commit and push:**

```bash
cd $SKILL_REPO
git add -A
git commit -m "Initial scaffold: <SKILL_NAME> skill"
git push -u origin main
```

This push triggers `version-bump.yml`. Since it's the first commit, there's no previous version to compare — the workflow will auto-bump to `1.0.1` and dispatch to the marketplace. But the marketplace entry doesn't exist yet, so proceed to Step 4.

#### Step 4: Register in Marketplace

**4a. Clone or locate the marketplace repo:**

```bash
ls ~/source/repos/HurleySk/claude-plugins-marketplace 2>/dev/null || git clone https://github.com/HurleySk/claude-plugins-marketplace.git ~/source/repos/HurleySk/claude-plugins-marketplace
```

**4b. Add the plugin entry to `marketplace.json`:**

Read `.claude-plugin/marketplace.json`, then add a new entry to the `plugins` array:

```json
{
  "name": "<SKILL_NAME>",
  "source": {
    "source": "url",
    "url": "https://github.com/HurleySk/claude-<SKILL_NAME>-skill.git"
  },
  "description": "<Same description as plugin.json>",
  "version": "1.0.0"
}
```

**4c. Update `README.md`** — add a row to the plugin table.

**4d. Commit and push the marketplace:**

```bash
cd ~/source/repos/HurleySk/claude-plugins-marketplace
git add -A
git commit -m "Add <SKILL_NAME> skill to marketplace"
git push
```

The next time the skill repo is pushed, the CI/CD will auto-sync the version.

#### Step 5: Cold-Read Validation

Launch a subagent that reads ONLY `$SKILL_REPO/skills/<SKILL_NAME>/SKILL.md` and evaluates:

1. Can every step be executed without guessing?
2. Are all commands concrete (not "build the project" but `dotnet build --configuration Release`)?
3. Are prerequisite checks present with fix commands?
4. Are placeholder variables defined before use?
5. Would an agent following this blindly succeed?

Rate: PASS (all clear) or PARTIAL (list gaps). Fix any gaps before considering the skill done.

### `add-to [plugin-name]`

Add a new skill to an existing multi-skill plugin repo. This is for when a skill logically belongs alongside existing skills (e.g., adding a `testing` skill to a `plugin-dev` plugin).

#### Step 1: Locate the Plugin Repo

```bash
ls ~/source/repos/HurleySk/claude-<PLUGIN_NAME>-skill/skills/
```

This shows existing skills. The new skill will be added alongside them.

#### Step 2: Create the Skill Directory

```bash
mkdir -p ~/source/repos/HurleySk/claude-<PLUGIN_NAME>-skill/skills/<NEW_SKILL_NAME>
```

#### Step 3: Write the SKILL.md

Create `skills/<NEW_SKILL_NAME>/SKILL.md` with frontmatter and body following the best practices below. The `name` in frontmatter should be just the skill name (e.g., `testing`), not the full plugin name.

#### Step 4: Update Plugin Metadata

- **`plugin.json`**: Update `description` to mention the new skill. Add relevant `keywords`. Bump the minor version (e.g., `1.0.3` -> `1.1.0`) since this adds functionality.
- **`README.md`**: Add a section for the new skill with its commands.

#### Step 5: Push

```bash
cd ~/source/repos/HurleySk/claude-<PLUGIN_NAME>-skill
git add -A
git commit -m "Add <NEW_SKILL_NAME> skill"
git push
```

CI handles version bump and marketplace sync. The new skill is immediately available as `/<PLUGIN_NAME>:<NEW_SKILL_NAME>`.

#### Step 6: Cold-Read Validation

Same as `new` Step 5 — launch a subagent to validate the new SKILL.md.

### `help`

Show available commands and a summary of skill authoring best practices.

**Commands:**
- `/skill-authoring new <skill-name>` — Create a new skill as a standalone plugin repo with CI/CD and marketplace registration
- `/skill-authoring add-to <plugin-name>` — Add a skill to an existing multi-skill plugin
- `/skill-authoring help` — Show this help

**Quick reference — what makes a good skill:**
- Imperative workflows, not documentation
- Check/fix pattern for prerequisites
- Concrete commands, not descriptions
- Cold-read tested by a subagent

## Writing Good Skills

These best practices are distilled from building production skills. Follow them when writing any SKILL.md.

### Structure

Every skill should have:

1. **Frontmatter** — `name`, `description`, `argument-hint`
2. **Role statement** — one line: "You are an expert at X. Follow the patterns below."
3. **Context section** (optional) — background info the agent needs (repos, tools, conventions)
4. **Commands section** — each command as `### \`command-name\`` with step-by-step instructions
5. **Reference appendices** (optional) — tables, type mappings, option lists — at the end, not inline

### Rules

**Write imperative workflows, not documentation dumps.** Every command should be numbered steps an agent can execute blindly. Not "the tool supports X, Y, Z" but "1. Run X. 2. Check Y. 3. If Z fails, fix with W."

**Use check/fix pattern for prerequisites.** Don't assume tools are installed. Check first, provide the fix command if missing:
```
Check: `test -f "$TOOL_PATH" && echo "OK" || echo "MISSING"`
Fix if MISSING: `<install command>`
```

**Use concrete commands, not descriptions.** Wrong: "Build the project." Right: `dotnet build --configuration Release`. Wrong: "Update the config file." Right: "Read `config.json`, add a new entry to the `plugins` array with these fields: ..."

**Define placeholder variables early.** At the start of a workflow, tell the agent to discover and remember paths:
```
Glob for `MyTool.exe` under `$HOME/source/repos`. Remember the path as `$TOOL_EXE`.
Derive `$TOOL_REPO` as the ancestor directory containing `MyTool.sln`.
```

**Keep appendices at the end.** Reference tables (CLI options, type mappings, naming conventions) go after all commands, not inline where they break flow.

**One SKILL.md per concern.** If a plugin covers multiple domains (dev + testing, frontend + backend), use separate skills under `skills/`. Each loads independently — no context bloat.

**Frontmatter `description` is critical.** Claude uses it to decide when to suggest the skill. Be specific about user intent: "Use when the user wants to create a new XrmToolBox plugin" not "XrmToolBox plugin stuff."

**Always include a `help` command.** List all available commands with one-line descriptions.

### Testing

**Cold-read validate every skill.** After writing, launch a subagent that:
- Reads ONLY the SKILL.md (no other context)
- Attempts to mentally trace through each command's workflow
- Reports: Was every step clear? Did it need to guess anything? Were any commands incomplete?
- Rates: PASS or PARTIAL with specific gaps

If the subagent rates PARTIAL, fix the gaps and re-test.

## Plugin Structure Reference

### Single-Skill Plugin

```
claude-my-skill/
  .claude-plugin/
    plugin.json          # "name": "my-skill"
  skills/
    my-skill/
      SKILL.md           # "name: my-skill" in frontmatter
  .github/workflows/version-bump.yml
  LICENSE
  README.md
```

Command: `/my-skill <args>`

### Multi-Skill Plugin

```
claude-my-plugin/
  .claude-plugin/
    plugin.json          # "name": "my-plugin"
  skills/
    skill-one/
      SKILL.md           # "name: skill-one"
    skill-two/
      SKILL.md           # "name: skill-two"
  .github/workflows/version-bump.yml
  LICENSE
  README.md
```

Commands: `/my-plugin:skill-one <args>`, `/my-plugin:skill-two <args>`

### plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "What it does — specific, used for discovery",
  "author": { "name": "Samuel Hurley", "url": "https://github.com/HurleySk" },
  "license": "MIT",
  "homepage": "https://github.com/HurleySk/claude-my-plugin",
  "repository": "https://github.com/HurleySk/claude-my-plugin",
  "keywords": ["relevant", "tags"]
}
```

Skills are auto-discovered from `skills/*/SKILL.md` — they are NOT listed in `plugin.json`.

### SKILL.md Frontmatter

```yaml
---
name: skill-name
description: When to trigger — specific user intent
argument-hint: "[cmd1|cmd2|help] [args]"
---
```

## CI/CD Reference

Every plugin repo in the HurleySk marketplace uses this pattern:

### version-bump.yml (in each plugin repo)

- **Trigger**: push to default branch
- **Skip if**: `github.actor == 'github-actions[bot]'` (prevents infinite loop from auto-bump commits)
- **Logic**: Compare `plugin.json` version vs previous commit. If unchanged, auto-increment patch. If already bumped manually, use existing version.
- **Tags**: Creates a git tag `v<version>` and pushes it. Claude Code uses these tags to resolve plugin versions during install.
- **Then**: Dispatch `repository_dispatch` event `plugin-version-update` to `HurleySk/claude-plugins-marketplace` with `{plugin_name, version}`
- **Requires**: `MARKETPLACE_PAT` secret (GitHub PAT with `repo` scope) and `permissions: contents: write`

### sync-version.yml (in marketplace repo)

- **Trigger**: `repository_dispatch` type `plugin-version-update`
- **Logic**: Use `jq` to update the matching plugin entry in `marketplace.json` by name, commit and push
- **No secrets needed**: Uses default `GITHUB_TOKEN` to push to its own repo

### Setting up CI for a new plugin

1. Copy `version-bump.yml` from any existing HurleySk plugin repo
2. Confirm `branches:` matches the default branch
3. Add `MARKETPLACE_PAT` secret: `gh secret set MARKETPLACE_PAT --repo HurleySk/<repo-name>`
4. Add the plugin entry to `marketplace.json` in the marketplace repo
5. First push will trigger the workflow and sync

## External Contributor Path

For someone other than Samuel Hurley who wants to add a skill to the marketplace:

1. **Create their own plugin repo** with the standard structure (`.claude-plugin/plugin.json`, `skills/*/SKILL.md`)
2. **Fork** `HurleySk/claude-plugins-marketplace`
3. **Add their plugin entry** to `marketplace.json`:
   ```json
   {
     "name": "their-skill",
     "source": { "source": "url", "url": "https://github.com/TheirUser/their-skill-repo.git" },
     "description": "What it does",
     "version": "1.0.0"
   }
   ```
4. **Open a PR** — Samuel reviews for:
   - Valid `plugin.json` with all required fields
   - SKILL.md follows best practices (imperative workflows, not doc dumps)
   - No malicious commands, credential harvesting, or destructive operations
   - Relevant to the marketplace's focus area
5. **On merge**, the plugin is live in the marketplace

Note: External plugins won't have the auto-version-sync CI (they don't have the `MARKETPLACE_PAT` secret for the HurleySk marketplace). Version updates require a new PR to bump the version in `marketplace.json`.
