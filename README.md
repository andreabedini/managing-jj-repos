# Managing Jujutsu (jj) Repositories Plugin

Comprehensive Claude Code plugin for working with Jujutsu (jj) version control. Provides specialized skills for daily operations, Git migration, troubleshooting, and reference documentation.

## Features

- **4 specialized skills** for focused, context-aware assistance
- **Comprehensive documentation** covering workflows, troubleshooting, and reference material
- **Git migration support** with command translations and mental model guidance
- **Real-world examples** demonstrating common scenarios
- **Progressive disclosure** - loads detailed docs only when needed

## Skills Included

### `/jj` - Daily Operations
Primary skill for working with jj. Covers:
- Essential commands and workflows
- Quick reference for common operations
- Revset examples
- Safety warnings
- Best practices

**Triggers:** Mentions of jj, jujutsu, change IDs, working copy, bookmarks, revsets, or jj operations.

### `/jj-migrate` - Git Migration
Helps Git users transition to jj. Includes:
- Mental model differences
- Git-to-jj command translations
- Common misconceptions
- Migration strategies
- Workflow pattern comparisons

**Triggers:** Git migration, transitioning from Git, Git commands in jj context.

### `/jj-troubleshoot` - Problem Solving
Solves common problems and errors. Covers:
- Error message solutions
- Conflict resolution
- Recovery scenarios
- Diagnostic commands

**Invocation:** Manual only (`/jj-troubleshoot`) - prevents auto-triggering on normal operations.

### `jj-reference` - Background Knowledge
Comprehensive reference material (not directly invocable):
- Complete command reference
- Terminology glossary
- Revset syntax
- Template syntax

**Always loaded** as background knowledge for other skills.

## Installation

### For Local Development/Testing
```bash
claude --plugin-dir ./managing-jj-repos
```

### From Marketplace
Add the marketplace:
```bash
/plugin marketplace add <marketplace-url>
```

Install the plugin:
```bash
/plugin install managing-jj-repos@<marketplace-name>
```

## Usage

### Getting Started
Just start working with jj, and Claude will automatically invoke the appropriate skill:

```
How do I create a commit in jj?
```

Claude loads `/jj` skill and provides guidance.

### Manual Invocation
Invoke skills directly:

```bash
/jj                    # Main operations help
/jj-migrate            # Git migration guide
/jj-troubleshoot       # Problem solving
```

### With Arguments
Pass context to skills:

```bash
/jj how do I rebase onto main?
/jj-migrate explain staging area differences
/jj-troubleshoot bookmark conflict
```

## Documentation Structure

```
managing-jj-repos/
├── skills/
│   ├── jj/                          # Main operations
│   │   ├── SKILL.md
│   │   ├── WORKFLOWS.md             # Step-by-step checklists
│   │   └── examples/
│   │       ├── feature-branch.md
│   │       └── conflict-resolution.md
│   ├── jj-migrate/                  # Git migration
│   │   ├── SKILL.md
│   │   └── GIT-COMPARISON.md        # Complete command translations
│   ├── jj-troubleshoot/             # Problem solving
│   │   ├── SKILL.md
│   │   └── TROUBLESHOOTING.md       # Error messages & solutions
│   └── jj-reference/                # Background knowledge
│       ├── SKILL.md
│       ├── REFERENCE.md             # Complete command reference
│       └── GLOSSARY.md              # Terminology guide
└── .claude-plugin/
    └── plugin.json                  # Plugin manifest
```

## Examples

See `skills/jj/examples/` for real-world scenarios:
- **feature-branch.md** - Complete feature branch workflow with multiple commits
- **conflict-resolution.md** - Step-by-step conflict resolution walkthrough

## Contributing

Contributions welcome! Please ensure:
- Skills remain focused and under 500 lines
- Documentation uses progressive disclosure
- Examples demonstrate real-world scenarios
- Safety warnings are prominent

## License

MIT

## Resources

- [Official jj documentation](https://martinvonz.github.io/jj/)
- [jj GitHub repository](https://github.com/martinvonz/jj)
- [Agent Skills standard](https://agentskills.io)

## Version

1.0.0
