# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Claude Code plugin (`managing-jj-repos`) that provides Jujutsu (jj) version control skills. It is a pure documentation/skill plugin — there is no build system, compiled code, or test suite. Development means editing Markdown files.

## Plugin Structure

```
.claude-plugin/plugin.json   # Plugin manifest (name, version, metadata)
skills/
  jj/                        # Daily operations skill (auto-triggered)
    SKILL.md                 # Skill entrypoint and quick reference
    WORKFLOWS.md             # Step-by-step checklists
    examples/                # Real-world scenario walkthroughs
  jj-migrate/                # Git→jj migration skill (auto-triggered)
    SKILL.md
    GIT-COMPARISON.md        # Full git-to-jj command translation table
  jj-reference/              # Background knowledge (always loaded, not directly invocable)
    SKILL.md
    REFERENCE.md             # Complete command reference
    GLOSSARY.md              # Terminology guide
  jj-troubleshoot/           # Problem-solving skill (manual invocation only)
    SKILL.md
    TROUBLESHOOTING.md       # Error messages and recovery procedures
```

## Skill Conventions

- Each skill has a `SKILL.md` with YAML frontmatter: `name`, `description` (used for auto-trigger matching).
- `jj-troubleshoot` is **manual-only** — its description intentionally avoids trigger keywords to prevent auto-invocation during normal operations.
- `jj-reference` is background knowledge loaded by other skills, not a standalone invocable skill.
- Skills follow progressive disclosure: `SKILL.md` contains quick reference; deeper docs (WORKFLOWS.md, REFERENCE.md, etc.) are loaded on demand.
- Skills should stay under 500 lines each.

## Testing the Plugin Locally

```bash
claude --plugin-dir ./managing-jj-repos
```

Then invoke skills manually:
```
/jj
/jj-migrate
/jj-troubleshoot
```
