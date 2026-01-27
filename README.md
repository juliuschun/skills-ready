# /ready - Resume Work Skill for Claude Code

A Claude Code skill that helps you quickly resume work on any project. Shows project context, git activity, past conversation history, and related files.

## What it does

When you run `/ready`, Claude will:

1. **Project Context** - README summary, CLAUDE.md, codebase structure
2. **Git Activity** - Uncommitted changes, recent commits, modified files
3. **Session History** - List all Claude sessions for this project
4. **Interactive Exploration** - Choose a session, view conversation, read related files
5. **Synthesis** - "Here's where you left off..." with recommended next steps

## Installation

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/ready

# Copy the skill file
curl -o ~/.claude/skills/ready/SKILL.md https://raw.githubusercontent.com/<your-username>/claude-ready/main/SKILL.md
```

Or manually:
```bash
mkdir -p ~/.claude/skills/ready
cp SKILL.md ~/.claude/skills/ready/
```

## Usage

In Claude Code, type:

```
/ready
```

Or ask naturally:
- "What was I working on?"
- "Show me past sessions"
- "Resume"

## Features

### Project-Filtered Sessions
Only shows sessions from the current project folder.

### Conversation Parser
Extracts readable conversation from sessions:
- Filters out tool calls, tool results, thinking blocks
- Shows only user prompts and Claude's responses
- Choose: 10, 20, or 30 turns

### Related Files
After viewing a session, see which files were worked on:
- Extracts Read/Edit/Write operations from session
- Ranks by frequency
- Option to read selected files

### Works Without Git
Falls back gracefully for non-git projects.

## Example Output

```
## Project: my-app
Tech Stack: React, TypeScript, Vite

## Git Activity
Uncommitted: M src/App.tsx, A components/New.tsx
Recent: abc123 Add auth, def456 Setup project

## Sessions in This Project
[1] Implement auth flow (47 msgs, 2024-01-26)
[2] Setup project (23 msgs, 2024-01-25)

Which session to explore? → [1]
How many turns? → 10

## Last 10 Turns
👤 USER: Add login form...
🤖 CLAUDE: I'll create a login component...

## Files Worked On
[1] src/components/Login.tsx (5 writes)
[2] src/App.tsx (3 edits)

Which files to read? → [1]
```

## Requirements

- Claude Code CLI
- `jq` (for JSON parsing)

## License

MIT
