---
name: jj-reference
description: Background knowledge about Jujutsu (jj) terminology, complete command reference, and revset syntax. Provides comprehensive reference material for jj operations.
user-invocable: false
---

# Jujutsu Reference Documentation

Background knowledge skill that provides comprehensive reference material. This skill is always loaded but not directly invocable - it provides context for other jj operations.

## Purpose

This skill contains detailed reference information that Claude can access when needed:
- Complete terminology definitions
- Full command reference with all options
- Revset syntax and functions
- Template syntax

## Documentation Files

### [REFERENCE.md](REFERENCE.md)
Complete command reference organized by category:
- Initialization & Git Integration
- Viewing & Navigation
- Creating & Editing Commits
- Rewriting History
- Bookmarks
- Working Copy Management
- Recovery & Operations
- Configuration
- Revset Syntax
- Template Syntax

### [GLOSSARY.md](GLOSSARY.md)
Comprehensive terminology guide covering:
- Core concepts (Change, Change ID, Working Copy, etc.)
- History & rewriting terms
- Conflict terminology
- Workflow terms
- Git integration concepts
- Comparison to Git concepts
- Symbols & notation

## When This is Loaded

This skill is automatically available as background knowledge when working with jj. Other skills can reference this documentation:
- `/jj` loads this for command reference
- `/jj-migrate` references glossary for term definitions
- `/jj-troubleshoot` uses command reference for solutions

## Progressive Disclosure

The detailed reference material is only loaded when specifically needed, keeping context focused on the immediate task while ensuring comprehensive information is available.
