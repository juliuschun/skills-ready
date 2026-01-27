# /ready

Resume work skill for Claude Code. Quickly get back to where you left off.

Claude Code용 "이어서 작업하기" 스킬. 중단했던 작업을 빠르게 재개하세요.

---

## Installation / 설치

### Option 1: Ask Claude Code (Recommended / 추천)

Just type this in Claude Code:

> Install the /ready skill from https://github.com/juliuschun/skills-ready

Claude Code에서 이렇게 입력하세요:

> https://github.com/juliuschun/skills-ready 에서 /ready 스킬을 설치해줘

### Option 2: Manual / 수동 설치

```bash
mkdir -p ~/.claude/skills/ready && curl -o ~/.claude/skills/ready/SKILL.md https://raw.githubusercontent.com/juliuschun/skills-ready/main/SKILL.md
```

---

## Usage / 사용법

```
/ready
```

Or ask naturally / 자연어로 물어보세요:
- "What was I working on?" / "내가 뭐 하고 있었지?"
- "Resume" / "이어서 작업"
- "Show past sessions" / "지난 세션 보여줘"

---

## Features / 기능

| English | 한국어 |
|---------|--------|
| **Project Context** - README, CLAUDE.md, structure | **프로젝트 컨텍스트** - README, CLAUDE.md, 구조 |
| **Git Activity** - Uncommitted changes, commits | **Git 활동** - 미커밋 변경사항, 최근 커밋 |
| **Session History** - All sessions for this project | **세션 히스토리** - 이 프로젝트의 모든 세션 |
| **Conversation Replay** - View past chats (10/20/30 turns) | **대화 재생** - 과거 대화 보기 (10/20/30 턴) |
| **Related Files** - See and read files you worked on | **관련 파일** - 작업했던 파일 확인 및 읽기 |
| **Synthesis** - "Here's where you left off..." | **요약** - "여기까지 작업했습니다..." |

---

## How It Works / 작동 방식

```
1. Project Context       →  프로젝트 컨텍스트
   ├── README.md
   ├── CLAUDE.md
   └── Codebase structure

2. Git Activity          →  Git 활동
   ├── Uncommitted changes
   └── Recent commits

3. Session History       →  세션 히스토리 (인터랙티브)
   ├── List sessions
   ├── Pick one
   └── View conversation

4. Related Files         →  관련 파일 (인터랙티브)
   ├── Files from session
   └── Read selected

5. Synthesis             →  요약
   └── "Here's where you left off..."
```

---

## Example / 예시

```
## Project: my-app
Tech Stack: React, TypeScript, Vite

## Git Activity
Uncommitted: M src/App.tsx
Recent: abc123 Add auth flow

## Sessions
[1] Implement login (47 msgs)
[2] Setup project (23 msgs)

Which session? → 1
How many turns? → 10

👤 USER: Add a login form...
🤖 CLAUDE: I'll create a Login component...

## Files Worked On
[1] src/Login.tsx (5 writes)
[2] src/App.tsx (3 edits)

Read files? → 1
```

---

## Requirements / 요구사항

- Claude Code CLI
- `jq` (JSON parsing)

---

## License

MIT
