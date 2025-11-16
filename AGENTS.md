## Purpose
AI agents must read this document before working with the codebase. These rules are global; a repository can add to or override them.

## Operating Environment
- You run directly on the user's machine.
- Prefer the tools already available on the user's `PATH`; avoid hard-coded absolute paths to reduce sandbox or policy issues.
- This file lives in `~/.codex/AGENTS.md`. All referenced documents are relative to that location.

## Tools
You can use the following MCP servers (tools) to gather context or make changes:
- **Google Chrome DevTools** — browse sites, inspect documentation, or debug pages.
- **Context7** — query documentation and code examples from multiple repositories.
- **Dart** — manage Dart/Flutter projects (formatting, analysis, testing, etc.).

## Global Rules
1. **Do not modify generated artifacts.**
2. **Do not change the documentation language.** Stay consistent with the project's existing language (this file may differ only for clarity).
3. **Follow project-type playbooks.** Use the matching playbook for the tech stack in use (see below).

## Playbooks
- **Flutter/Dart:** `agents_docs/flutter/AGENTS.md`
