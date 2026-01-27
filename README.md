# /ready

Resume work skill for Claude Code. Quickly get back to where you left off.

## Installation

### Option 1: Ask Claude Code (Recommended)

Just type this in Claude Code:

> Install the /ready skill from https://github.com/juliuschun/skills-ready

### Option 2: Manual

```bash
mkdir -p ~/.claude/skills/ready && curl -o ~/.claude/skills/ready/SKILL.md https://raw.githubusercontent.com/juliuschun/skills-ready/main/SKILL.md
```

## Usage

```
/ready
```

Or ask naturally:
- "What was I working on?"
- "Resume"
- "Show past sessions"

## Features

| Feature | Description |
|---------|-------------|
| **Project Context** | README, CLAUDE.md, codebase structure |
| **Git Activity** | Uncommitted changes, recent commits |
| **Session History** | All Claude sessions for this project |
| **Conversation Replay** | View past chats (10/20/30 turns) |
| **Related Files** | See and read files you worked on |
| **Synthesis** | "Here's where you left off..." |

## How It Works

```
1. Project Context
   ├── README.md
   ├── CLAUDE.md
   └── Codebase structure

2. Git Activity
   ├── Uncommitted changes
   └── Recent commits

3. Session History (interactive)
   ├── List sessions
   ├── Pick one
   └── View conversation

4. Related Files (interactive)
   ├── Files from session
   └── Read selected

5. Synthesis
   └── "Here's where you left off..."
```

## Example

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

## Requirements

- Claude Code CLI
- `jq` (JSON parsing)

## License

MIT

---

# /ready (한국어)

Claude Code용 "이어서 작업하기" 스킬. 중단했던 작업을 빠르게 재개하세요.

## 설치

### 방법 1: Claude Code에서 설치 (추천)

Claude Code에서 이렇게 입력하세요:

> https://github.com/juliuschun/skills-ready 에서 /ready 스킬을 설치해줘

### 방법 2: 수동 설치

```bash
mkdir -p ~/.claude/skills/ready && curl -o ~/.claude/skills/ready/SKILL.md https://raw.githubusercontent.com/juliuschun/skills-ready/main/SKILL.md
```

## 사용법

```
/ready
```

또는 자연어로 물어보세요:
- "내가 뭐 하고 있었지?"
- "이어서 작업"
- "지난 세션 보여줘"

## 기능

| 기능 | 설명 |
|------|------|
| **프로젝트 컨텍스트** | README, CLAUDE.md, 코드베이스 구조 |
| **Git 활동** | 미커밋 변경사항, 최근 커밋 |
| **세션 히스토리** | 이 프로젝트의 모든 Claude 세션 |
| **대화 재생** | 과거 대화 보기 (10/20/30 턴) |
| **관련 파일** | 작업했던 파일 확인 및 읽기 |
| **요약** | "여기까지 작업했습니다..." |

## 작동 방식

```
1. 프로젝트 컨텍스트
   ├── README.md
   ├── CLAUDE.md
   └── 코드베이스 구조

2. Git 활동
   ├── 미커밋 변경사항
   └── 최근 커밋

3. 세션 히스토리 (인터랙티브)
   ├── 세션 목록
   ├── 선택
   └── 대화 보기

4. 관련 파일 (인터랙티브)
   ├── 세션에서 작업한 파일
   └── 선택한 파일 읽기

5. 요약
   └── "여기까지 작업했습니다..."
```

## 예시

```
## 프로젝트: my-app
기술 스택: React, TypeScript, Vite

## Git 활동
미커밋: M src/App.tsx
최근: abc123 인증 플로우 추가

## 세션
[1] 로그인 구현 (47 메시지)
[2] 프로젝트 설정 (23 메시지)

어떤 세션? → 1
몇 턴? → 10

👤 USER: 로그인 폼 추가해줘...
🤖 CLAUDE: Login 컴포넌트를 만들겠습니다...

## 작업한 파일
[1] src/Login.tsx (5회 쓰기)
[2] src/App.tsx (3회 수정)

파일 읽기? → 1
```

## 요구사항

- Claude Code CLI
- `jq` (JSON 파싱)

## 라이선스

MIT
