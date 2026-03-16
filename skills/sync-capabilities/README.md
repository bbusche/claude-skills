# sync-capabilities

A Claude skill that bridges the gap between Claude Code (CLI) and Claude Desktop/CoWork by generating a shared capabilities manifest.

## What It Does

When triggered, this skill:

1. **Inventories MCP servers** — runs `claude mcp list` to discover all connected remote integrations
2. **Inventories installed skills** — scans `~/.claude/skills/` for all skill definitions
3. **Inventories CLI tools** — checks for key tools (notebooklm, yt-dlp, gh, node, python3, playwright, etc.)
4. **Writes a manifest** — generates a structured markdown file at `~/Development/claude-capabilities.md`

## The Problem

Claude Code and Claude Desktop/CoWork are completely separate environments:

- **Claude Code** has shell access, MCP servers, CLI tools, and custom skills
- **Claude Desktop/CoWork** has its own connectors and skills but no visibility into Claude Code

When you ask CoWork to do something that requires Claude Code (e.g., create a NotebookLM notebook, push code to GitHub), it has no idea those capabilities exist. This skill solves that by maintaining a manifest that CoWork can reference.

## Triggers

- "Sync capabilities"
- "Update capabilities manifest"
- "Refresh what CoWork knows about Claude Code"

## File Structure

```
sync-capabilities/
├── README.md     ← this file (GitHub display)
├── SKILL.md      ← skill instructions loaded by Claude
└── LICENSE.txt   ← MIT license
```

## How CoWork Discovers the Manifest

Place a `CLAUDE.md` in a parent directory (e.g., `~/Development/CLAUDE.md`) that tells CoWork to read the manifest:

```markdown
# Claude Code Integration

Read `~/Development/claude-capabilities.md` whenever the user asks about available
tools, integrations, or capabilities — or when a task may require CLI access.
```

CoWork will see this `CLAUDE.md` whenever scoped to any folder under that directory.

## Output

The generated manifest (`claude-capabilities.md`) includes:

- **MCP Servers** — name, endpoint, connection status
- **Installed Skills** — name, trigger, description
- **CLI Tools** — name, path, version, purpose
- **Capability Summary** — what Claude Code can do that CoWork cannot
- **Handoff Guide** — when CoWork should suggest switching to Claude Code

## When to Run

Run "sync capabilities" after:

- Adding or removing an MCP server
- Installing or uninstalling a Claude Code skill
- Installing or updating a CLI tool (notebooklm, yt-dlp, etc.)
- Any time CoWork seems unaware of Claude Code's current capabilities

## Installation

Copy the `sync-capabilities/` folder into your Claude skills directory (typically `~/.claude/skills/` or the path configured in your Claude settings), then restart Claude.

## Author

Built using the [Claude Skill framework](https://github.com/anthropics/skills).
