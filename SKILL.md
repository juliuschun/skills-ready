---
name: ready
description: Use this skill when the user asks "what was I working on", "show past sessions", "recent todos", "/ready", "resume", or wants to understand the project and see what they were working on.
version: 1.4.0
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

3. Session Timeline (interactive)
   ├── Haiku subagent reads last 10 sessions (5 turns each)
   ├── Show summary table with last message from each
   ├── Ask: Which session to continue?
   ├── Ask: How many events? (20/40/60)
   └── Display interleaved timeline (conversation + commands + edits)

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

### 3.1 Summarize Recent Sessions with Haiku Subagent

Use the **Task tool** with `subagent_type: "haiku"` and `model: "haiku"` to efficiently read and summarize the last 10 sessions. The subagent should:

1. Get the list of session IDs:
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"

# Get last 10 session IDs
cat "$PROJECT_DIR/sessions-index.json" 2>/dev/null | jq -r '
  .entries | sort_by(.modified) | reverse | .[:10][] |
  "\(.sessionId) | \(.summary // "Untitled") | \(.messageCount) msgs | \(.modified | split("T")[0])"
'
```

2. For each session, extract the **last 5 conversation turns** (user/assistant only):
```bash
SESSION_ID="<session-id>"
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "user" or .type == "assistant") |
  if .type == "user" then
    if (.message.content | type) == "string" then
      "👤 " + (.message.content | gsub("\n"; " ") | .[0:150])
    elif (.message.content | type) == "array" then
      (.message.content[] | select(.type == "text") | "👤 " + (.text | gsub("\n"; " ") | .[0:150]))
    else empty end
  elif .type == "assistant" then
    (.message.content[]? | select(.type == "text") | "🤖 " + (.text | gsub("\n"; " ") | .[0:200]))
  else empty end
' 2>/dev/null | grep -v "^$" | tail -10
```

3. Present a summary table with the **last message** from each session:

```
## Recent Sessions (Last 10)

| # | Session | Last Activity | Where You Left Off |
|---|---------|---------------|-------------------|
| 1 | Toss Payment Integration | 2h ago | 👤 "should we test or find bugs with subagents?" |
| 2 | Heavy Mode Refactor | 1d ago | 🤖 "All fixes applied. Ready for testing." |
| 3 | API Rate Limiting | 2d ago | 👤 "deploy to prod when ready" |
| 4 | UI Polish | 3d ago | 🤖 "Dark mode toggle is working." |
| 5 | Database Migration | 4d ago | 👤 "looks good, let's move on" |
...
```

**Haiku Subagent Prompt:**
```
Read the last 10 sessions for this project. For each session:
1. Extract the last 5 user/assistant conversation turns
2. Identify the final message (what was the user/claude last discussing?)
3. Summarize in one line what the session was about

Present as a table showing:
- Session number
- Session title
- Time since last activity
- The actual last message (truncated) so user knows where they left off

This helps the user quickly see all recent work and pick which to continue.
```

### 3.2 Ask Which Session

After showing the summary table, ask the user which session to explore:

```
Question: "Which session would you like to continue?"
Header: "Session"
Options (use actual last messages from the table):
  - "[1] Toss Payment" (description: "should we test or find bugs...")
  - "[2] Heavy Mode" (description: "All fixes applied. Ready for testing.")
  - "[3] API Rate Limiting" (description: "deploy to prod when ready")
  - "Skip" (description: proceed with synthesis only)
```

**Note:** The last message preview helps users instantly recognize where they left off without needing to load the full session.

### 3.3 Ask How Many Events

If user selected a session, use **AskUserQuestion**:
```
Question: "How many timeline events to retrieve?"
Header: "Events"
Options:
  - "20 events (Recommended)" (description: last 20 interactions)
  - "40 events" (description: more context)
  - "60 events" (description: extensive history)
```

### 3.4 Retrieve Session Timeline (Interleaved)

Display an **interleaved chronological timeline** showing conversation, CLI commands, and code changes in the order they actually happened:

```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"

# Get session ID by index (0-based)
SESSION_INDEX=0  # Change based on user selection
SESSION_ID=$(cat "$PROJECT_DIR/sessions-index.json" | jq -r ".entries | sort_by(.modified) | reverse | .[$SESSION_INDEX].sessionId")

# Number of events to show
EVENTS=20  # Change based on user selection

# Parse interleaved timeline (conversation + commands + code changes)
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "user" or .type == "assistant") |
  if .type == "user" then
    if (.message.content | type) == "string" then
      "👤 " + (.message.content | gsub("\n"; " ") | .[0:100])
    elif (.message.content | type) == "array" then
      (.message.content[] | select(.type == "text") | "👤 " + (.text | gsub("\n"; " ") | .[0:100]))
    else empty end
  elif .type == "assistant" then
    (.message.content[]? |
      if .type == "text" then
        "🤖 " + (.text | gsub("\n"; " ") | .[0:120])
      elif .type == "tool_use" then
        if .name == "Bash" then
          "⚡ $ " + (.input.command | gsub("\n"; " ") | .[0:80])
        elif .name == "Edit" then
          "📝 Edit: " + (.input.file_path | split("/") | .[-1])
        elif .name == "Write" then
          "📄 Write: " + (.input.file_path | split("/") | .[-1])
        else empty end
      else empty end)
  else empty end
' 2>/dev/null | grep -v "^$" | tail -$EVENTS
```

Present as:
```
## Session Timeline

🤖 The issue is that Toss redirects without the payToken...
📝 Edit: payment.complete.tsx
📝 Edit: payment.complete.tsx
👤 "Oops! An unexpected error occurred."
🤖 Let me check the logs...
⚡ $ pm2 logs deep-dive-dev --lines 30
🤖 I see! Toss returns orderNo not payToken. Let me fix...
📝 Edit: pricing.tsx
⚡ $ pnpm build
⚡ $ ./scripts/manage.sh restart webapp
🤖 Try again. The SSR issue is fixed.
```

**Legend:**
- 👤 User message
- 🤖 Claude response
- ⚡ CLI command executed
- 📝 File edited
- 📄 File created/written

This interleaved view shows the actual workflow narrative, making it easy to understand what happened and where you left off.

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

## Timeline Parser

The timeline shows an **interleaved chronological view** of the session, displaying events in the order they actually happened.

**Includes** (in chronological order):
- `👤` User messages (truncated to 100 chars)
- `🤖` Claude responses (truncated to 120 chars)
- `⚡` Bash commands executed (truncated to 80 chars)
- `📝` File edits (filename only)
- `📄` File writes/creates (filename only)

**Filters out** (noise):
- `tool_result` - Tool outputs/returns (we show the command, not the output)
- `thinking` - Extended thinking blocks
- System messages
- Read operations (just exploration, not changes)
- Exploratory tools (Glob, Grep, etc.)

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

### Session Timeline (Interleaved)
```bash
ENCODED_PATH=$(pwd | sed 's|/|-|g; s|_|-|g')
PROJECT_DIR="$HOME/.claude/projects/$ENCODED_PATH"
SESSION_ID="<session-id>"
cat "$PROJECT_DIR/$SESSION_ID.jsonl" | jq -r '
  select(.type == "user" or .type == "assistant") |
  if .type == "user" then
    if (.message.content | type) == "string" then
      "👤 " + (.message.content | gsub("\n"; " ") | .[0:100])
    elif (.message.content | type) == "array" then
      (.message.content[] | select(.type == "text") | "👤 " + (.text | gsub("\n"; " ") | .[0:100]))
    else empty end
  elif .type == "assistant" then
    (.message.content[]? |
      if .type == "text" then
        "🤖 " + (.text | gsub("\n"; " ") | .[0:120])
      elif .type == "tool_use" then
        if .name == "Bash" then
          "⚡ $ " + (.input.command | gsub("\n"; " ") | .[0:80])
        elif .name == "Edit" then
          "📝 Edit: " + (.input.file_path | split("/") | .[-1])
        elif .name == "Write" then
          "📄 Write: " + (.input.file_path | split("/") | .[-1])
        else empty end
      else empty end)
  else empty end
' | grep -v "^$" | tail -40
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
