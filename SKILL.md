---
name: ready
description: Use this skill when the user asks "what was I working on", "show past sessions", "recent todos", "/ready", "resume", or wants to understand the project and see what they were working on.
version: 1.2.0
---

# Ready

Complete "resume work" skill for Claude Code. Shows project context, git activity, session history, and related files for the current project folder.

## Execution Flow

```
1. Project Context
   ├── README.md summary
   ├── CLAUDE.md (if exists)
   └── Codebase structure (git ls-files)

2. Git Activity
   ├── Uncommitted changes (git status)
   ├── Recent commits (last 10)
   └── Files changed recently

3. Session History (interactive)
   ├── List sessions for this project
   ├── Ask: Which session to explore?
   ├── Ask: How many conversation turns? (10/20/30)
   ├── Display conversation (user/assistant messages)
   ├── Display key CLI commands (builds, commits, installs)
   └── Display code changes summary (files modified)

4. Related Files (interactive)
   ├── Extract files from session tool_use (Read/Edit/Write)
   ├── Rank by frequency and action type
   ├── Ask: Which files to read?
   └── Display selected files

5. Synthesis Report
   └── "Here's where you left off..."
```

---

## Step 1: Project Context

### Read Project Files
```bash
# Check for README and CLAUDE.md
ls -la README.md CLAUDE.md 2>/dev/null
```

Read and summarize:
- **README.md** - What is this project? Tech stack?
- **CLAUDE.md** - Any special instructions for Claude?

### Codebase Structure
```bash
# Quick overview of project structure
git ls-files 2>/dev/null | head -50

# Or if not a git repo
find . -type f -not -path '*/\.*' -not -path '*/node_modules/*' -not -path '*/dist/*' | head -50
```

Present a brief summary:
```
## Project: <name from README or folder>
**Tech Stack:** React, TypeScript, Vite (inferred from files)
**Structure:** src/, components/, utils/, tests/
```

---

## Step 2: Git Activity

### Uncommitted Changes
```bash
git status --short 2>/dev/null
```

### Recent Commits
```bash
git log --oneline --date=short --format="%h %ad %s" -10 2>/dev/null
```

### Files Changed Recently
```bash
git diff --stat HEAD~5 2>/dev/null | tail -15
```

### Recently Modified Files (last 24h)
```bash
find . -type f -not -path '*/\.*' -not -path '*/node_modules/*' -not -path '*/dist/*' -mtime -1 2>/dev/null | head -15
```

Present:
```
## Git Activity

**Uncommitted Changes:**
M  src/App.tsx
A  components/NewFeature.tsx

**Recent Commits:**
- abc123 2024-01-26 Add session history skill
- def456 2024-01-26 Fix conversation parser
```

---

## Step 3: Session History (Interactive)

### 3.1 List Sessions
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"

cat "$PROJECT_DIR/sessions-index.json" 2>/dev/null | jq -r '
  .entries | sort_by(.modified) | reverse | to_entries[] |
  "[\(.key + 1)] \(.value.summary // "Untitled") (\(.value.messageCount) msgs, \(.value.modified | split(".")[0] | sub("T"; " ")))"
'
```

Present the **full list** of all sessions with date and time:
```
## Sessions in This Project

[1] Claude Companion: Swarm Agent Timeline UI (47 msgs, 2024-01-26 18:45:32)
[2] Multi-agent orchestration skill (37 msgs, 2024-01-26 15:20:11)
[3] Swarm-Team Skill: 3-Layer Architecture (39 msgs, 2024-01-26 12:05:44)
[4] Repo dissection (8 msgs, 2024-01-26 09:30:00)
[5] Initial project setup (12 msgs, 2024-01-25 22:15:08)
[6] README drafting (5 msgs, 2024-01-25 14:00:22)
... (show all available sessions)
```

### 3.2 Ask Which Session

After showing the full list, ask the user which session number they want to explore.

Use **AskUserQuestion** tool with the actual session names from the list:
```
Question: "Which session would you like to explore?"
Header: "Session"
Options (use actual session summaries from the list above):
  - "[1] <first session summary>" (description: <message count> messages)
  - "[2] <second session summary>" (description: <message count> messages)
  - "[3] <third session summary>" (description: <message count> messages)
  - "Skip" (description: don't load conversation, just show summary)
```

**Note:** AskUserQuestion supports max 4 options. If there are more sessions, show the top 3 most recent + "Skip". The user can select "Other" to type a different session number from the full list displayed above.

### 3.3 Ask How Many Turns

If user selected a session, use **AskUserQuestion**:
```
Question: "How many conversation turns to retrieve?"
Header: "Turns"
Options:
  - "10 turns (Recommended)" (description: last 10 user + 10 assistant messages)
  - "20 turns" (description: more context)
  - "30 turns" (description: extensive history)
```

### 3.4 Retrieve Conversation
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"

# Get session ID by index (0-based)
SESSION_INDEX=0  # Change based on user selection
SESSION_ID=$(cat "$PROJECT_DIR/sessions-index.json" | jq -r ".entries | sort_by(.modified) | reverse | .[$SESSION_INDEX].sessionId")

# Number of turns
TURNS=10  # Change based on user selection
TAIL_LINES=$((TURNS * 2))

# Parse conversation (filters out tool calls, keeps only user/assistant text)
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "user" or .type == "assistant") |
  if .type == "user" then
    if (.message.content | type) == "string" then
      "👤 USER: " + (.message.content | gsub("\n"; " ") | .[0:200])
    elif (.message.content | type) == "array" then
      (.message.content[] | select(.type == "text") | "👤 USER: " + (.text | gsub("\n"; " ") | .[0:200]))
    else empty end
  elif .type == "assistant" then
    (.message.content[]? | select(.type == "text") | "🤖 CLAUDE: " + (.text | gsub("\n"; " ") | .[0:300]) + "...")
  else empty end
' 2>/dev/null | grep -v "^$" | tail -$TAIL_LINES
```

### 3.5 Extract Significant CLI Commands

Extract only meaningful CLI commands (builds, commits, installs, deployments). Skip exploratory commands like `ls`, `cat`, `grep`, repeated `git status`.

```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<from-step-3>"  # Use the session ID selected earlier

# Extract significant Bash commands only (filter out exploration/diagnostic commands)
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use") |
  select(.name == "Bash") |
  .input.command
' 2>/dev/null | grep -E '^(npm (run|install|build|test|start)|yarn |pnpm |git (add|commit|push|pull|merge|checkout|rebase)|docker |make |go (build|run|test)|cargo |python |pip |pytest|mv |cp |mkdir |rm |chmod |curl.*-X|wget )' | head -15 | while read cmd; do
  echo "$ $(echo "$cmd" | tr '\n' ' ' | cut -c1-120)"
done
```

Present as:
```
## Key Commands Executed

$ npm install axios
$ npm run build
$ git add -A && git commit -m "Add new feature"
$ docker compose up -d
```

**Note:** Only shows build/test/install/deploy commands, git operations, and file operations. Diagnostic commands are filtered out.

### 3.6 Extract Code Changes Summary

Show a compact summary of files edited/created (file paths with operation counts, not full diffs):

```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<from-step-3>"  # Use the session ID selected earlier

# Count edits and writes per file
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use") |
  select(.name == "Edit" or .name == "Write") |
  "\(.name) \(.input.file_path)"
' 2>/dev/null | grep -v "\.claude/" | sort | uniq -c | sort -rn | head -15
```

Present as:
```
## Code Changes Summary

| File | Operations |
|------|------------|
| src/components/App.tsx | 3 edits |
| src/components/NewFeature.tsx | 1 write |
| src/utils/api.ts | 2 edits |
| src/types/feature.ts | 1 write |
```

**Note:** Shows file-level summary. Use Step 4 (Related Files) to read the actual current file contents if needed.

---

## Step 4: Related Files (Interactive)

After showing the conversation, extract and offer to read files that were worked on.

### 4.1 Extract Files from Session
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<from-step-3>"  # Use the session ID selected earlier

# Extract files with action counts, exclude ~/.claude/ paths
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use") |
  select(.name == "Read" or .name == "Edit" or .name == "Write") |
  "\(.name) \(.input.file_path)"
' 2>/dev/null | grep -v "\.claude/" | sort | uniq -c | sort -rn | head -10
```

### 4.2 Present Files List

Show files ranked by activity:
```
## Files Worked On

| # | File | Activity |
|---|------|----------|
| 1 | src/components/AgentTimeline.tsx | 3 writes, 2 edits |
| 2 | src/App.tsx | 3 writes, 2 reads |
| 3 | src/types/agent.ts | 2 writes |
| 4 | src/data/mockAgents.ts | 2 writes |
```

### 4.3 Ask Which Files to Read

Use **AskUserQuestion** tool:
```
Question: "Which files would you like to read?"
Header: "Files"
multiSelect: true
Options:
  - "AgentTimeline.tsx (Recommended)" (description: most activity - 3 writes, 2 edits)
  - "App.tsx" (description: 3 writes, 2 reads)
  - "agent.ts" (description: 2 writes)
  - "Skip" (description: don't read any files)
```

### 4.4 Read Selected Files

For each selected file, use the **Read** tool to display its current contents.

Present as:
```markdown
## File: src/components/AgentTimeline.tsx

<file contents here>
```

---

## Step 5: Synthesis Report

After gathering all context, synthesize:

```markdown
## Summary: Where You Left Off

### Project
<Brief description from README>

### Recent Work
- <What files were changed based on git status/commits>
- <Any uncommitted work>

### Last Session: <session summary>
- <Key topics from conversation>
- <Any pending tasks from todos>

### Recommended Next Steps
1. <Based on uncommitted changes or in_progress todos>
2. <Based on conversation context>
```

---

## Data Sources

| Source | Location |
|--------|----------|
| Project files | `README.md`, `CLAUDE.md` |
| Codebase | `git ls-files` |
| Git activity | `git status`, `git log`, `git diff` |
| Sessions | `~/.claude/projects/<encoded-path>/sessions-index.json` |
| Transcripts | `~/.claude/projects/<encoded-path>/<session-id>.jsonl` |
| Todos | `~/.claude/todos/<session-id>-*.json` |
| Related files | Extracted from `tool_use` in session transcripts |
| CLI commands | Extracted from `Bash` tool_use in session transcripts |
| Code changes | Extracted from `Edit`/`Write` tool_use in session transcripts |

---

## Conversation & Activity Parser

**For conversation display** (filters out noise):
- `tool_result` - Tool outputs/returns
- `thinking` - Extended thinking
- System messages

**Conversation keeps**:
- `👤 USER:` - User prompts
- `🤖 CLAUDE:` - Claude's text responses

**CLI Commands extracts** (filtered to significant only):
- Build commands: `npm run`, `yarn`, `go build`, `cargo`, `make`
- Git operations: `git add`, `git commit`, `git push`
- Package installs: `npm install`, `pip install`
- Deployment: `docker`, `kubectl`

**Code Changes extracts** (file-level summary):
- Edit/Write operation counts per file
- Excludes `~/.claude/` paths

---

## Quick Reference Commands

### List Sessions
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
cat "$HOME/.claude/projects/$ENCODED_PATH/sessions-index.json" | jq -r '
  .entries | sort_by(.modified) | reverse | .[:10][] |
  "\(.summary) | \(.messageCount) msgs | \(.modified | split("T")[0])"
'
```

### Project Todos
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
for sid in $(cat "$PROJECT_DIR/sessions-index.json" | jq -r '.entries[].sessionId'); do
  TODO="$HOME/.claude/todos/${sid}-agent-${sid}.json"
  [ -s "$TODO" ] && [ $(wc -c < "$TODO") -gt 2 ] && echo "=== $sid ===" && cat "$TODO" | jq -r '.[] | "[\(.status)] \(.content)"'
done
```

### Key CLI Commands from Session
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<session-id>"
# Filter to significant commands only (builds, commits, installs)
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use" and .name == "Bash") |
  .input.command
' | grep -E '^(npm |yarn |git (add|commit|push)|docker |make |go |cargo )' | head -15
```

### Code Changes from Session
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<session-id>"
# Edits
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use" and .name == "Edit") |
  "EDIT: " + .input.file_path
'
# Writes
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "assistant") |
  .message.content[]? |
  select(.type == "tool_use" and .name == "Write") |
  "WRITE: " + .input.file_path
'
```

---

## Implementation Notes

### Project Path Encoding
- Claude encodes paths by replacing `/` AND `_` with `-`
- Example: `/Users/foo/01_bar` → `-Users-foo-01-bar`
- Project data: `~/.claude/projects/<encoded-path>/`

### Session Data
- `sessions-index.json`: Metadata (summary, timestamps, message count)
- `<session-id>.jsonl`: Full transcript
- Todo files: `<session-id>-agent-<session-id>.json`

### Non-Git Repos
- Git commands may fail - handle gracefully
- Fall back to `find` for file structure
- Skip git activity section if not a repo
