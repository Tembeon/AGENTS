# Agents global rules

This repository contains my global rules for AI agents. The guidance is meant to be language- and framework-agnostic so it can improve daily workflows across multiple stacks.

Any text-based agent CLI (Gemini, Codex, Claude Code, etc.) can load these instructions.

## Using

Treat these files as global defaults. Place them wherever your agent CLI expects its top-level rules:

- Codex CLI in `~/.codex/AGENTS.md`
- Gemini CLI in `~/.gemini/<configured-context-filename>` (e.g., `~/.gemini/GEMINI.md`)
- Claude Code in `~/.claude/CLAUDE.md`

## MCP servers

MCP servers expand an LLM’s abilities beyond editing text. Install whichever ones match your workflow:

| Tool                                                                     | Install                                                                                                     | Description                                                                                        |
|--------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| [Context7](https://github.com/upstash/context7)                          | [GitHub](https://github.com/upstash/context7?tab=readme-ov-file#%EF%B8%8F-installation)                     | Up-to-date docs and code snippets to cover features that an LLM might not know yet.                |
| [Chrome DevTools](https://github.com/ChromeDevTools/chrome-devtools-mcp) | [GitHub](https://github.com/ChromeDevTools/chrome-devtools-mcp?tab=readme-ov-file#mcp-client-configuration) | Gives agents a browser to read, debug, and inspect live sites - great for docs and visual checks.  |
| [Dart & Flutter MCP](https://docs.flutter.dev/ai/mcp-server)             | [GitHub](https://github.com/dart-lang/ai/tree/main/pkgs/dart_mcp_server#set-up-your-mcp-client)             | Project-aware commands for formatting, analysis, testing, and device control in Dart/Flutter apps. |

## Resources

Key folders you might want to inspect:

| Path           | What’s inside                                               | How to use it                                                             |
|----------------|-------------------------------------------------------------|---------------------------------------------------------------------------|
| `agents_docs/` | Playbooks for specific languages, frameworks, or workflows. | Read the relevant `AGENTS.md` before touching that stack (Flutter, etc.). |
| `prompts/`     | Reusable slash-command prompts for Codex TUI.               | Trigger with `/prompts:<name>` to inject the template into your session.  |
