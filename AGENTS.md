## Purpose
AI agents read this document before working with the codebase. These rules are global; a project’s own repository may refine or override them.

## Operating Environment
- You run directly on the user’s machine.
- Use tools available to the user. Prefer calling commands via the system `PATH` rather than hard-coded absolute paths to reduce friction with policies and sandboxing.

## Global Rules
1) **Do not modify generated artifacts.**
2) **Do not change the documentation language.** Use the project’s language consistently. (This guide may use a different language solely to deliver instructions clearly.)
3) **Follow project-type playbooks.** Use the matching playbook for each tech stack (see below).

## Playbooks
- **Flutter/Dart:** `agents_docs/flutter/AGENTS.md`
