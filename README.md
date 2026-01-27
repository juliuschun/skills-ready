# /ready

Resume work skill for Claude Code. Quickly get back to where you left off.

## Why?

When you start a new Claude Code session, Claude has no memory of previous conversations. This makes it hard to continue unfinished tasks or pick up work that was never documented.

Even within a session, **autocompact** can silently discard context when conversations get long. You lose decisions, debugging history, and the "why" behind your code - often without realizing it until you need that information.

Past conversation history contains valuable context:
- **Undone tasks** - What you planned but didn't finish
- **Decisions made** - Why you chose a certain approach
- **Files touched** - What was being worked on
- **Debugging context** - Errors encountered and solutions discussed

This skill retrieves that history so Claude can understand where you left off and continue seamlessly.

> **Tip:** Consider disabling autocompact for better context retention:
> ```
> claude config set --global autoCompact false
> ```
> Full context helps Claude make better decisions and maintain consistency throughout your session.

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

## 왜 필요한가요?

새 Claude Code 세션을 시작하면 Claude는 이전 대화를 기억하지 못합니다. 그래서 미완료 작업을 이어가거나 문서화되지 않은 작업을 다시 시작하기 어렵습니다.

세션 중에도 **autocompact** 기능이 대화가 길어지면 컨텍스트를 자동으로 삭제합니다. 결정 사항, 디버깅 히스토리, 코드의 "왜"가 사라지고 - 그 정보가 필요할 때까지 알아차리지 못하는 경우가 많습니다.

과거 대화 기록에는 소중한 컨텍스트가 담겨 있습니다:
- **미완료 작업** - 계획했지만 끝내지 못한 것들
- **결정 사항** - 특정 접근 방식을 선택한 이유
- **작업한 파일** - 어떤 파일을 수정했는지
- **디버깅 컨텍스트** - 발생한 오류와 논의된 해결책

이 스킬은 그 기록을 가져와서 Claude가 어디서 멈췄는지 이해하고 자연스럽게 이어서 작업할 수 있게 합니다.

> **팁:** 더 나은 컨텍스트 유지를 위해 autocompact를 끄는 것을 권장합니다:
> ```
> claude config set --global autoCompact false
> ```
> 전체 컨텍스트가 있어야 Claude가 더 나은 결정을 내리고 세션 전체에서 일관성을 유지할 수 있습니다.

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
