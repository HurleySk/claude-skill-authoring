---
name: skill-review
description: Use when assessing an existing skill or skill set for quality, gaps, and enhancement opportunities — performs a comprehensive multi-dimensional review covering safety, coverage, platform compatibility, resilience, usability, cross-skill consistency, and documentation.
argument-hint: "[assess|checklist|help] [skill-path-or-plugin-name]"
---

# Skill Review — Comprehensive Quality Assessment

You are an expert skill reviewer. Your job is to rigorously assess Claude Code skills for gaps, inconsistencies, and enhancement opportunities across every dimension that matters. You don't just check if a skill works — you check if it's production-grade.

This review methodology was distilled from real-world skill assessment that uncovered critical safety bypasses, platform incompatibilities, and resilience failures in production skills.

## Commands

### `assess`

Full multi-dimensional assessment of a skill or skill set. Follow these phases exactly.

#### Phase 1: Discovery

Identify what you're reviewing.

1. If the user provided a path, read the SKILL.md file(s) directly
2. If the user provided a plugin name, find the plugin:
   ```
   Glob for skills/*/SKILL.md under the plugin directory
   Read .claude-plugin/plugin.json for metadata
   Read README.md for documented behavior
   ```
3. Catalog every skill in the plugin:
   - Name, description, commands
   - What tools/hooks/agents it creates or configures
   - What artifacts it generates (files, configs, scripts)

#### Phase 2: Multi-Dimensional Analysis

For EACH skill, evaluate across these 8 dimensions. Launch up to 3 Explore agents in parallel for thorough analysis of the skill content.

##### Dimension 1: Coverage Completeness

Ask: "What should this skill protect against / enable that it currently doesn't?"

- **Tool coverage**: Does it cover ALL relevant Claude Code tools? Check each:
  - `Bash` and `PowerShell` (both exist — covering one but not the other is a gap)
  - `Write`, `Edit`, `NotebookEdit` (all are distinct tools)
  - `Read` (for audit trails on sensitive file access)
  - MCP tools (document if not interceptable)
  - `Agent` (subagent dispatch — can subagents bypass the skill's controls?)
- **Operation coverage**: What dangerous operations exist in the skill's domain that aren't handled?
  - For safety skills: destructive commands, cloud CLI, package installs, secret exposure, file deletion
  - For dev skills: error paths, edge cases, rollback scenarios
  - For workflow skills: interruption handling, partial completion, retry logic
- **Pattern coverage**: Are regex/glob patterns complete? Common gaps:
  - Patterns that only match one variant (`--force` but not `--force-with-lease`)
  - Patterns with trailing `\s` that miss end-of-line cases
  - Hardcoded examples from the original project that don't generalize

##### Dimension 2: Platform Compatibility

Ask: "Will this skill work on every platform where the plugin can be installed?"

- **Plugin restrictions**: Does `plugin.json` declare platform restrictions? If not, the skill must work everywhere.
- **Shell scripts**: Are `.sh` scripts the only option? Windows users need Git Bash or WSL.
  - Can scripts be rewritten in Node.js for true cross-platform?
  - Are there PowerShell equivalents needed?
- **Path separators**: Does the skill handle `\` vs `/` correctly?
- **Command availability**: Does it assume commands exist without checking? (`node`, `jq`, `grep -E` vs `grep -P`, etc.)
- **Environment variables**: Are OS-specific env vars used? (`$HOME` vs `$USERPROFILE`, etc.)

##### Dimension 3: Resilience & Failure Modes

Ask: "What happens when things go wrong? Does it fail safely?"

- **Fail-open vs fail-closed**: When a prerequisite is missing (tool not installed, config file missing/corrupt, env var unset), does the skill:
  - Silently continue (FAIL-OPEN = critical gap for safety skills)
  - Error with diagnostics (CORRECT for most skills)
  - Block all operations (FAIL-CLOSED = correct for safety-critical skills)
- **Error suppression**: Look for `2>/dev/null`, `|| true`, `catch{}` without error handling. Each suppression should be justified — is the suppressed error truly benign?
- **Dependency validation**: Does the skill verify its dependencies exist before using them?
  - Runtime dependencies (node, python, specific CLIs)
  - File dependencies (config files, rule files, scripts it references)
  - Environment dependencies (`$CLAUDE_PROJECT_DIR`, `$HOME`, etc.)
- **Corrupt input handling**: What if config files have invalid JSON? What if hook input is malformed?
- **Partial execution**: If the skill's workflow is interrupted mid-way, what state is left behind? Can the user recover?

##### Dimension 4: Cross-Skill Consistency

Ask: "Do the skills in this plugin agree with each other?"

- **Shared configuration**: Do multiple skills reference the same config file? Do they all read from it, or do some hardcode values?
- **Terminology**: Are the same concepts called the same things across skills?
- **Handoffs**: When one skill's output feeds another skill's input, is the format documented and machine-readable? Or is it narrative ("pass the findings as context")?
- **Lessons learned**: Does one skill's lessons section contradict another skill's implementation? (e.g., "never use 2>/dev/null" in one skill while another skill uses it)
- **Completeness parity**: If one skill covers a tool/platform, do related skills also cover it? (e.g., Bash hooks exist but PowerShell hooks don't)

##### Dimension 5: Usability & Lifecycle

Ask: "Can users manage this skill over time, not just set it up?"

- **Lifecycle commands**: Does the skill provide:
  - Setup/install
  - Status/health check (what's currently active?)
  - Validation/test (are things working?)
  - Disable/enable (temporary bypass without destroying config)
  - Update/reconfigure (change settings without full reinstall)
  - Uninstall/reset (clean removal)
- **Discoverability**: Can a user quickly understand what the skill protects or enables? Is there a summary view?
- **Maintenance**: When the underlying system changes (new environments, new tools, new patterns), how does the user update? Is it documented?
- **Escape hatches**: For blocking/safety skills — can users do intentional work that the skill would normally prevent? Is the mechanism documented?

##### Dimension 6: Workflow Quality

Ask: "Could an agent follow this blindly and succeed?"

- **Step completeness**: Is every step concrete enough to execute without guessing?
- **Variable definitions**: Are placeholders defined before use? Can they be derived?
- **Prerequisite checks**: Does the skill verify prerequisites with check/fix patterns?
- **Branching logic**: When a step has conditionals (if X, do Y), are both branches fully specified?
- **Testing instructions**: Does the skill include verification steps that confirm it worked?
- **Error recovery**: When a step fails, does the skill say what to do?

##### Dimension 7: Security Posture

Ask: "Could this skill be abused, or does it create new attack surfaces?"

- **Credential handling**: Does the skill handle secrets? Are they stored safely?
- **Command injection**: Do hook scripts interpolate user-controlled content into shell commands or JSON strings without escaping?
- **Scope creep**: Do patterns match too broadly? (e.g., matching `rm -f tempfile` when the intent was to catch `rm -rf /`)
- **False confidence**: Does the skill create the appearance of safety without actually providing it? (e.g., fail-open behavior with no warning)
- **Bypass vectors**: Can the skill's controls be circumvented by using a different tool, different syntax, or different path?

##### Dimension 8: Documentation Quality

Ask: "Does the skill explain itself well enough for users AND for agents?"

- **Frontmatter description**: Is it specific enough for Claude to know when to suggest the skill?
- **Argument hint**: Does it list all available commands?
- **Help command**: Does it exist? Does it list all commands with descriptions?
- **Architecture diagrams**: For complex skills, is the flow visible?
- **Lessons learned**: Are hard-won insights documented so future contributors don't repeat mistakes?
- **Prerequisites**: Are runtime dependencies listed?
- **Known limitations**: Are gaps honestly documented?
- **Examples**: Are there concrete examples for non-obvious operations?

#### Phase 3: Cross-Cutting Analysis

After reviewing each dimension, synthesize cross-cutting findings:

1. **Priority ranking**: Which gaps have the highest impact? Use this framework:
   - **Critical** = safety bypass, data loss, or silent failure that creates false confidence
   - **High** = significant functionality gap, platform exclusion, or broken cross-skill integration
   - **Medium** = missing lifecycle commands, documentation gaps, or usability friction
   - **Low** = polish, edge cases, or nice-to-haves

2. **Effort estimation**: For each finding:
   - **Low** = single edit to one SKILL.md section (add a pattern, fix a comment)
   - **Medium** = new section or command in one SKILL.md, or changes across 2-3 files
   - **High** = new skill file, major restructuring, or cross-platform rewrite

3. **Quick wins**: Identify findings that are Critical/High priority AND Low effort — these should be fixed immediately.

#### Phase 4: Report

Write the assessment to `output/skill-review-<plugin-name>.md` with this structure:

```markdown
# Skill Review: <plugin-name>

**Date:** <date>
**Skills reviewed:** <list>
**Overall grade:** <A/B/C/D> (see grading rubric below)

## Executive Summary
2-3 sentences: overall quality, most critical finding, top recommendation.

## Dimension Scores
| Dimension | Score | Key Finding |
|-----------|-------|-------------|
| Coverage | X/5 | ... |
| Platform | X/5 | ... |
| Resilience | X/5 | ... |
| Cross-Skill | X/5 | ... |
| Usability | X/5 | ... |
| Workflow | X/5 | ... |
| Security | X/5 | ... |
| Documentation | X/5 | ... |

## Critical Findings
(Findings that must be fixed — safety bypasses, silent failures, platform breaks)

## High-Priority Findings
(Significant gaps that should be fixed)

## Medium-Priority Findings
(Operational improvements)

## Low-Priority Findings
(Polish and completeness)

## Enhancement Recommendations
| # | Enhancement | Priority | Effort | Files |
|---|-------------|----------|--------|-------|
(Prioritized table of concrete recommendations)

## Quick Wins
(Critical/High priority + Low effort — fix these first)
```

**Grading rubric:**
- **A** = Production-grade. Minor polish items only. All 8 dimensions score 4+.
- **B** = Solid with gaps. No critical issues but has high-priority findings. Most dimensions score 3+.
- **C** = Functional but incomplete. Has critical or multiple high-priority gaps. Some dimensions score 2 or below.
- **D** = Needs significant work. Multiple critical issues or fundamental design problems.

#### Phase 5: Present to User

Summarize key findings:
- Overall grade and what's driving it
- Top 3 critical/high findings
- Quick wins they can fix immediately
- Ask if they want to proceed with implementing the enhancements

### `checklist`

Quick pass/fail checklist for a skill. Faster than a full assessment — useful for validating a skill before publishing.

Run through each item. Report PASS, WARN, or FAIL for each.

#### Skill Structure
- [ ] Has valid frontmatter (name, description, argument-hint)
- [ ] Description is specific about user intent (not generic like "does X stuff")
- [ ] Has a `help` command listing all available commands
- [ ] Commands use `### \`command-name\`` format with step-by-step instructions
- [ ] Steps are imperative and concrete (not "update the config" but specific instructions)

#### Completeness
- [ ] All placeholder variables are defined before use
- [ ] Prerequisites are checked with fix commands
- [ ] Error/failure paths are handled (not just happy path)
- [ ] Testing/verification steps are included
- [ ] All relevant tools are covered (Bash AND PowerShell, Write AND Edit AND NotebookEdit)

#### Resilience
- [ ] No unguarded `2>/dev/null` on security-critical operations
- [ ] Missing dependencies cause clear errors, not silent bypass
- [ ] Config files are validated before use (exist + valid format)
- [ ] Environment variables are checked before use

#### Safety
- [ ] No command injection via string interpolation in shell scripts
- [ ] No hardcoded secrets or credentials
- [ ] Patterns aren't so broad they cause false positives / alert fatigue
- [ ] Patterns aren't so narrow they miss obvious variants

#### Documentation
- [ ] README lists all commands
- [ ] Architecture is explained for complex skills
- [ ] Prerequisites/dependencies are documented
- [ ] Known limitations are documented
- [ ] Lessons learned section exists (for non-trivial skills)

#### Cross-Skill (if multi-skill plugin)
- [ ] Shared config files are used consistently
- [ ] Terminology is consistent across skills
- [ ] Skill handoffs use machine-readable formats
- [ ] Tool/platform coverage is consistent across skills

Report: **X/Y passed**, list of WARN and FAIL items with specific remediation.

### `help`

**Available commands:**

| Command | Description |
|---------|-------------|
| `/skill-authoring:skill-review assess [path]` | Full 8-dimension assessment with graded report |
| `/skill-authoring:skill-review checklist [path]` | Quick pass/fail validation checklist |
| `/skill-authoring:skill-review help` | Show this help |

**What it reviews:**

The `assess` command evaluates skills across 8 dimensions:
1. **Coverage** — Does it handle all relevant tools, operations, and patterns?
2. **Platform** — Will it work on Windows, macOS, and Linux?
3. **Resilience** — Does it fail safely when things go wrong?
4. **Cross-Skill** — Do skills in the same plugin agree with each other?
5. **Usability** — Can users manage it over time (status, test, disable, uninstall)?
6. **Workflow** — Can an agent follow it blindly and succeed?
7. **Security** — Does it create new attack surfaces or false confidence?
8. **Documentation** — Are prerequisites, limitations, and lessons documented?

**Grading:**
- **A** = Production-grade (all dimensions 4+/5)
- **B** = Solid with gaps (no critical issues, most dimensions 3+/5)
- **C** = Functional but incomplete (critical or multiple high-priority gaps)
- **D** = Needs significant work

## Review Methodology Reference

This section documents the thinking patterns that make reviews thorough. Internalize these — they're the difference between surface-level checks and catching real issues.

### Think Like an Attacker

For every mechanism the skill creates, ask: "How would I bypass this?"
- Different tool? (Bash hook but no PowerShell hook)
- Different syntax? (`git push --force-with-lease` vs `--force`)
- Different path? (MCP tool that writes files outside hook matchers)
- Remove the guard? (Delete the config file — does the skill fail-open?)
- Corrupt the guard? (Invalid JSON in rules file — does it silently allow?)

### Think Like a Platform

Don't assume the user's environment matches the developer's:
- Windows users exist. Git Bash exists but is not guaranteed.
- `node` is not always installed. `jq` is rarely installed.
- `grep -E` behaves differently across implementations.
- `\x27` hex escapes in regex work on GNU grep but may not on BSD/Git Bash grep.
- Path separators differ. `$HOME` vs `$USERPROFILE`.

### Think Like a Maintainer

Skills that are set-and-forget eventually become stale:
- Environments change. New prod indicators, new risky directories.
- Teams change. New people need to understand what's protected.
- Tools change. New Claude Code tool matchers, new MCP servers.
- Does the skill support updating without full reinstall?
- Does the skill report its own health?

### Think Across Skills

In multi-skill plugins, skills must be coherent:
- If one skill writes a config file, other skills should read it (not hardcode their own values).
- If one skill covers Bash, related skills should cover PowerShell too.
- If one skill has a "lessons learned" section saying "don't do X," no other skill should do X.
- Handoffs between skills must be machine-readable, not narrative.

### Think About Failure

The most dangerous failure mode is **silent success** — the skill appears to work but actually isn't protecting anything:
- `2>/dev/null` on a critical operation that might fail
- `|| true` after a check that should block on failure
- `if [ -f "$FILE" ]` that silently skips all logic when the file is missing
- `exit 0` as the default case in a safety hook (fail-open)

For safety-critical skills, the correct default is DENY, not ALLOW.

### Common Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| Hardcoded project-specific examples in templates | Confuses new users, not generalizable | Use clearly marked placeholders with comments |
| `2>/dev/null` on all node/command calls | Hides real errors, can silently bypass safety | Only suppress expected benign messages |
| Narrative handoffs ("pass the findings as context") | Not machine-readable, agents can't automate | Use structured JSON files with defined schemas |
| Single tool matcher coverage | Leaves other tools completely unguarded | Cover all relevant tools or document the gap |
| Fail-open defaults in safety hooks | Creates false confidence — worse than no hook at all | Fail-closed: deny when uncertain |
| No lifecycle commands beyond setup | Users can't maintain, validate, or remove the skill | Add status, test, disable/enable, uninstall |
| Patterns that only match one variant | Attackers/mistakes use the unmatched variant | Enumerate all known variants, document gaps |
