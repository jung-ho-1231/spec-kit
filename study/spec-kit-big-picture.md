# Spec Kit - 전체 실행 흐름 (큰 그림)

README 순서대로 따라가면서 각 단계에서 실행되는 파일과 생성되는 산출물을 정리합니다.

---

## 📦 STEP 0: 프로젝트 초기화

### 명령어
```bash
specify init my-project --ai claude
```

### 실행되는 파일
- `src/specify_cli/__init__.py` (Python CLI 메인)
  - `init_command()` 함수 실행

### 무슨 일이 일어나는가?

1. **AI 에이전트 감지**
   - AGENT_CONFIG에서 claude, gemini, copilot 등 확인
   - 설치 여부 체크 (--ignore-agent-tools 없으면)

2. **GitHub에서 템플릿 다운로드**
   - GitHub API 호출 (기본: github/spec-kit repository)
   - 최신 템플릿 ZIP 다운로드

3. **프로젝트 구조 생성**
   ```
   my-project/
   ├── .claude/commands/          # AI 에이전트 슬래시 커맨드
   │   ├── constitution.md
   │   ├── specify.md
   │   ├── plan.md
   │   ├── tasks.md
   │   ├── implement.md
   │   ├── clarify.md
   │   ├── analyze.md
   │   └── checklist.md
   ├── .specify/
   │   ├── memory/
   │   │   └── constitution.md    # 템플릿 (빈 상태)
   │   ├── scripts/
   │   │   ├── bash/              # Linux/Mac 스크립트
   │   │   └── powershell/        # Windows 스크립트
   │   ├── specs/                 # 기능 명세 저장소 (비어있음)
   │   └── templates/
   │       ├── spec-template.md
   │       ├── plan-template.md
   │       ├── tasks-template.md
   │       └── checklist-template.md
   └── .git/                      # Git 초기화 (--no-git 없으면)
   ```

4. **Git 저장소 초기화** (옵션)
   - `git init`
   - 초기 커밋 생성

### 핵심 의미
> 개발 환경 셋업. AI 에이전트가 사용할 슬래시 커맨드와 템플릿 설치.

---

## 📜 STEP 1: 프로젝트 원칙 수립

### 명령어 (AI 에이전트에서)
```bash
/speckit.constitution Create principles focused on code quality, testing standards
```

### 실행되는 파일
1. `.claude/commands/constitution.md` (AI가 읽음)
   - AI에게 주어지는 프롬프트
   
2. AI가 직접 작업:
   - `.specify/memory/constitution.md` 읽기
   - 템플릿의 플레이스홀더 채우기
   - 버전 관리 (Semantic Versioning)

### 생성되는 산출물
```
.specify/memory/constitution.md (업데이트됨)
```

### 내용 구조
```markdown
# Project Constitution v1.0.0

## Project Identity
- [PROJECT_NAME] → "Taskify"
- [PROJECT_PURPOSE] → "Team productivity platform"

## Principles
1. [PRINCIPLE_1_NAME]: Code Quality
   - [PRINCIPLE_1_DETAILS]: All code must have 80% test coverage
   
2. [PRINCIPLE_2_NAME]: User Experience
   - [PRINCIPLE_2_DETAILS]: Sub-3-second page load

## Governance
- Ratification Date: 2025-01-15
- Last Amended: 2025-01-15
- Version: 1.0.0
```

### 핵심 의미
> 프로젝트의 "헌법". 모든 후속 단계에서 이 원칙을 따라 검증.

---

## 📋 STEP 2: 기능 명세 작성

### 명령어
```bash
/speckit.specify Build a task management app with Kanban boards, user assignments, and comments
```

### 실행되는 파일 (순서대로)

1. **AI가 commands/specify.md 읽음**
   - 프롬프트에서 지시사항 파악

2. **AI가 브랜치 번호 계산**
   ```bash
   # Remote, Local, Specs 세 곳 확인
   git fetch --all --prune
   git ls-remote --heads origin | grep -E 'refs/heads/[0-9]+-task-management$'
   git branch | grep -E '^[* ]*[0-9]+-task-management$'
   ls .specify/specs/ | grep -E '^[0-9]+-task-management$'
   
   # 최대값 + 1 사용 (예: 001)
   ```

3. **AI가 스크립트 실행**
   ```bash
   .specify/scripts/bash/create-new-feature.sh \
     --json \
     --number 1 \
     --short-name "task-management" \
     "Build a task management app..."
   ```

4. **create-new-feature.sh 실행 내용**
   - Git 브랜치 생성: `001-task-management`
   - 디렉토리 생성: `.specify/specs/001-task-management/`
   - 템플릿 복사: `spec-template.md` → `spec.md`
   - JSON 출력:
     ```json
     {
       "BRANCH_NAME": "001-task-management",
       "SPEC_FILE": "/path/to/.specify/specs/001-task-management/spec.md"
     }
     ```

5. **AI가 spec.md 작성**
   - 템플릿 읽기: `templates/spec-template.md`
   - 사용자 입력 분석
   - User Scenarios, Requirements, Success Criteria 작성
   - `specs/001-task-management/spec.md`에 저장

6. **AI가 체크리스트 생성**
   - `specs/001-task-management/checklists/requirements.md` 생성
   - 명세서 품질 검증 항목

### 생성되는 산출물
```
.specify/specs/001-task-management/
├── spec.md                          # 기능 명세서
└── checklists/
    └── requirements.md              # 품질 체크리스트
```

### spec.md 구조
```markdown
# Feature: Task Management

## User Scenarios
- As a project manager, I want to create tasks...
- As a developer, I want to move tasks between columns...

## Functional Requirements
1. Task CRUD operations
2. Kanban board with drag-and-drop
3. User assignment

## Success Criteria (기술 스택 독립적!)
- Users can create a task in under 10 seconds
- Drag-and-drop works smoothly (no lag)
- 95% of task updates complete in under 1 second

## Key Entities (데이터 중심)
- Task: { id, title, description, status, assignee }
- User: { id, name, role }
- Project: { id, name, tasks[] }
```

### 핵심 의미
> **WHAT & WHY**에 집중. 기술 스택은 언급 안 함. 비즈니스 가치 중심.

---

## 🏗️ STEP 3: 기술 구현 계획

### 명령어
```bash
/speckit.plan Use React, Node.js, PostgreSQL. REST API with real-time updates via WebSocket
```

### 실행되는 파일 (순서대로)

1. **AI가 commands/plan.md 읽음**

2. **AI가 스크립트 실행**
   ```bash
   .specify/scripts/bash/setup-plan.sh --json
   ```

3. **setup-plan.sh 실행 내용**
   - common.sh에서 경로 가져오기
   - `plan-template.md` 복사 → `plan.md`
   - JSON 출력:
     ```json
     {
       "FEATURE_SPEC": "/path/to/specs/001-task-management/spec.md",
       "IMPL_PLAN": "/path/to/specs/001-task-management/plan.md",
       "SPECS_DIR": "/path/to/specs/001-task-management",
       "BRANCH": "001-task-management"
     }
     ```

4. **AI가 Phase 0: Research 실행**
   - 기술 스택 조사 (React best practices, PostgreSQL patterns 등)
   - `research.md` 생성
   - 각 기술 선택에 대한 근거 문서화

5. **AI가 Phase 1: Design 실행**
   - `data-model.md` 생성 (엔티티, 관계)
   - `contracts/` 생성 (API 명세 - OpenAPI 등)
   - `quickstart.md` 생성 (개발 환경 설정)

6. **AI가 update-agent-context.sh 실행**
   ```bash
   .specify/scripts/bash/update-agent-context.sh claude
   ```
   - CLAUDE.md (또는 GEMINI.md 등) 업데이트
   - 현재 기능 컨텍스트 추가

### 생성되는 산출물
```
.specify/specs/001-task-management/
├── spec.md                     # (이미 존재)
├── plan.md                     # ⭐ 기술 구현 계획
├── research.md                 # 기술 조사 결과
├── data-model.md               # 데이터 모델
├── contracts/
│   ├── tasks-api.yaml          # OpenAPI 명세
│   └── websocket-events.md     # WebSocket 이벤트 정의
├── quickstart.md               # 개발 환경 설정
└── checklists/
    └── requirements.md         # (이미 존재)
```

### plan.md 구조
```markdown
# Implementation Plan: Task Management

## Technical Context
- Frontend: React 18, TypeScript, Vite
- Backend: Node.js 20, Express, Socket.io
- Database: PostgreSQL 15
- API: REST + WebSocket

## Constitution Check
✓ Code Quality: ESLint + Prettier configured
✓ Testing: Jest + React Testing Library
✓ Performance: <3s page load target

## Core Implementation
1. Project structure setup
2. Database schema (PostgreSQL)
3. REST API endpoints
4. WebSocket real-time updates
5. React components (Kanban board)

## File Structure
```
src/
├── backend/
│   ├── routes/tasks.js
│   ├── models/Task.js
│   └── sockets/taskUpdates.js
├── frontend/
│   ├── components/KanbanBoard.tsx
│   └── services/api.ts
```
```

### 핵심 의미
> **HOW**를 정의. 기술 스택, 아키텍처, 파일 구조. spec.md와 연결.

---

## ✅ STEP 4: 작업 분해

### 명령어
```bash
/speckit.tasks
```

### 실행되는 파일

1. **AI가 commands/tasks.md 읽음**

2. **AI가 스크립트 실행**
   ```bash
   .specify/scripts/bash/check-prerequisites.sh --json
   ```

3. **check-prerequisites.sh 실행 내용**
   - common.sh에서 경로 가져오기
   - `plan.md` 존재 확인 (필수)
   - 다른 문서들 확인 (선택):
     - research.md ✓
     - data-model.md ✓
     - contracts/ ✓
     - quickstart.md ✓
   - JSON 출력:
     ```json
     {
       "FEATURE_DIR": "/path/to/specs/001-task-management",
       "AVAILABLE_DOCS": ["research.md", "data-model.md", "contracts/", "quickstart.md"]
     }
     ```

4. **AI가 모든 문서 읽고 tasks.md 생성**
   - `plan.md`에서 기술 스택, 파일 구조 추출
   - `spec.md`에서 User Stories (P1, P2, P3) 추출
   - `data-model.md`에서 엔티티 추출
   - `contracts/`에서 API 엔드포인트 추출
   - User Story별로 작업 그룹화
   - 의존성 분석하여 순서 결정

### 생성되는 산출물
```
.specify/specs/001-task-management/
├── tasks.md                    # ⭐ 실행 가능한 작업 목록
├── ... (다른 파일들)
```

### tasks.md 구조
```markdown
# Implementation Tasks: Task Management

## Dependencies
- User Story 1 (P1) → Independent
- User Story 2 (P2) → Depends on US1 (Task model)
- User Story 3 (P3) → Independent

## Phase 1: Setup
- [ ] T001 Initialize Node.js project with TypeScript
- [ ] T002 [P] Setup PostgreSQL database
- [ ] T003 [P] Setup React frontend with Vite

## Phase 2: Foundational
- [ ] T004 Create database schema (tasks, users, projects tables)
- [ ] T005 [P] Setup Express server with middleware

## Phase 3: User Story 1 - Create and View Tasks
**Goal**: Users can create tasks and view them in a list
**Test**: Create 5 tasks, verify they appear in the list

- [ ] T006 [P] [US1] Create Task model in src/backend/models/Task.js
- [ ] T007 [US1] Implement POST /api/tasks endpoint in src/backend/routes/tasks.js
- [ ] T008 [US1] Implement GET /api/tasks endpoint in src/backend/routes/tasks.js
- [ ] T009 [P] [US1] Create TaskForm component in src/frontend/components/TaskForm.tsx
- [ ] T010 [US1] Create TaskList component in src/frontend/components/TaskList.tsx

**Checkpoint**: Can create and view tasks through UI

## Phase 4: User Story 2 - Kanban Drag and Drop
**Goal**: Users can drag tasks between columns
**Test**: Drag 3 tasks from "To Do" to "In Progress"

- [ ] T011 [P] [US2] Install react-beautiful-dnd library
- [ ] T012 [US2] Create KanbanBoard component in src/frontend/components/KanbanBoard.tsx
- [ ] T013 [US2] Implement PATCH /api/tasks/:id/status in src/backend/routes/tasks.js
- [ ] T014 [US2] Add WebSocket support in src/backend/sockets/taskUpdates.js

**Checkpoint**: Drag-and-drop works, real-time updates visible

## Phase 5: Polish
- [ ] T015 [P] Add error handling middleware
- [ ] T016 [P] Add loading states to UI
- [ ] T017 Add documentation in README.md
```

### 핵심 의미
> **실행 가능한 체크리스트**. 각 작업은 독립적으로 실행 가능. User Story별 그룹화.

---

## 🚀 STEP 5: 구현 실행

### 명령어
```bash
/speckit.implement
```

### 실행되는 파일

1. **AI가 commands/implement.md 읽음**

2. **AI가 스크립트 실행**
   ```bash
   .specify/scripts/bash/check-prerequisites.sh \
     --json --require-tasks --include-tasks
   ```

3. **check-prerequisites.sh 검증**
   - `plan.md` 존재 확인 ✓
   - `tasks.md` 존재 확인 ✓ (--require-tasks)
   - 모든 문서 리스트업

4. **AI가 체크리스트 상태 확인**
   - `checklists/` 디렉토리 스캔
   - 각 체크리스트의 완료 여부 확인
   - 미완료 항목이 있으면 사용자에게 확인 요청:
     ```
     | Checklist          | Total | Completed | Incomplete | Status |
     |--------------------|-------|-----------|------------|--------|
     | requirements.md    | 12    | 12        | 0          | ✓ PASS |
     | ux.md              | 8     | 5         | 3          | ✗ FAIL |
     
     Some checklists are incomplete. Proceed anyway? (yes/no)
     ```

5. **AI가 Ignore 파일 생성/검증**
   - Git 저장소 확인 → `.gitignore` 생성/업데이트
   - `plan.md`에서 기술 스택 확인:
     - Node.js → `node_modules/`, `.env`, `dist/`
     - React → `build/`, `.cache/`
     - PostgreSQL → `.env` (DB 크레덴셜)

6. **AI가 tasks.md 파싱**
   - Phase별 작업 추출
   - 의존성 분석
   - [P] 마커 → 병렬 실행 가능
   - User Story별 체크포인트 파악

7. **AI가 Phase별 실행**

   **Phase 1: Setup**
   ```bash
   # T001
   npm init -y
   npm install typescript @types/node --save-dev
   tsc --init
   
   # T002 [P] - 병렬 실행
   createdb taskify_dev
   
   # T003 [P] - 병렬 실행
   npm create vite@latest frontend -- --template react-ts
   ```

   **Phase 2: Foundational**
   ```sql
   -- T004: database schema
   CREATE TABLE tasks (
     id SERIAL PRIMARY KEY,
     title VARCHAR(255),
     description TEXT,
     status VARCHAR(50),
     assignee_id INT
   );
   ```

   **Phase 3: User Story 1**
   ```javascript
   // T006 [P]: Task model
   class Task {
     constructor(id, title, description, status) { ... }
   }
   
   // T007: POST endpoint
   app.post('/api/tasks', async (req, res) => { ... });
   
   // T008: GET endpoint
   app.get('/api/tasks', async (req, res) => { ... });
   ```

   **Phase 4, 5 계속...**

8. **AI가 각 작업 완료 후 tasks.md 업데이트**
   ```markdown
   - [X] T001 Initialize Node.js project ✓
   - [X] T002 [P] Setup PostgreSQL database ✓
   - [X] T003 [P] Setup React frontend ✓
   ```

### 생성되는 산출물
```
my-project/
├── .gitignore                  # ⭐ 자동 생성
├── package.json                # ⭐ 프로젝트 설정
├── tsconfig.json               # ⭐ TypeScript 설정
├── src/
│   ├── backend/                # ⭐ 백엔드 코드
│   │   ├── server.js
│   │   ├── routes/
│   │   │   └── tasks.js
│   │   ├── models/
│   │   │   └── Task.js
│   │   └── sockets/
│   │       └── taskUpdates.js
│   └── frontend/               # ⭐ 프론트엔드 코드
│       ├── components/
│       │   ├── TaskForm.tsx
│       │   ├── TaskList.tsx
│       │   └── KanbanBoard.tsx
│       └── services/
│           └── api.ts
├── .specify/
│   └── specs/001-task-management/
│       └── tasks.md            # ⭐ 업데이트됨 (체크 완료)
```

### 핵심 의미
> **실제 코드 생성**. tasks.md를 하나씩 실행하며 작업 완료 체크.

---

## 🎯 전체 흐름 요약

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 0: specify init                                             │
│ 역할: 프로젝트 초기화                                               │
│ 산출물: .claude/, .specify/, templates/                          │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: /speckit.constitution                                   │
│ 역할: 프로젝트 원칙 수립                                            │
│ 산출물: memory/constitution.md                                   │
│ 의미: 프로젝트의 "헌법"                                            │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: /speckit.specify                                        │
│ 역할: 기능 명세 작성 (WHAT & WHY)                                 │
│ 실행: create-new-feature.sh (브랜치 생성)                         │
│ 산출물: specs/001-xxx/spec.md, checklists/requirements.md       │
│ 의미: 비즈니스 요구사항, 기술 독립적                                │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: /speckit.plan                                           │
│ 역할: 기술 구현 계획 (HOW)                                         │
│ 실행: setup-plan.sh, update-agent-context.sh                    │
│ 산출물: plan.md, research.md, data-model.md, contracts/         │
│ 의미: 기술 스택, 아키텍처, API 설계                                │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: /speckit.tasks                                          │
│ 역할: 작업 분해 (EXECUTE)                                         │
│ 실행: check-prerequisites.sh                                    │
│ 산출물: tasks.md                                                 │
│ 의미: 실행 가능한 체크리스트, User Story별 그룹화                  │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STEP 5: /speckit.implement                                      │
│ 역할: 실제 구현 (BUILD)                                           │
│ 실행: check-prerequisites.sh (검증), tasks.md 파싱 및 실행        │
│ 산출물: 실제 코드 (src/), .gitignore, 설정 파일들                 │
│ 의미: AI가 tasks.md를 하나씩 실행하며 코드 생성                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 핵심 인사이트

### 1. 단계별 관심사 분리 (Separation of Concerns)

| 단계 | 질문 | 산출물 | 독립성 |
|------|------|--------|--------|
| Constitution | "프로젝트 원칙은?" | constitution.md | 모든 단계의 기준 |
| Specify | "무엇을, 왜?" | spec.md | 기술 독립적 |
| Plan | "어떻게?" | plan.md, data-model.md, contracts/ | spec.md에 의존 |
| Tasks | "어떤 순서로?" | tasks.md | plan.md에 의존 |
| Implement | "실행" | 실제 코드 | tasks.md에 의존 |

### 2. 스크립트의 역할

| 스크립트 | 역할 | 호출 단계 |
|----------|------|-----------|
| `create-new-feature.sh` | 브랜치 생성, 번호 부여 | /speckit.specify |
| `setup-plan.sh` | plan.md 초기화 | /speckit.plan |
| `update-agent-context.sh` | AI 컨텍스트 업데이트 | /speckit.plan |
| `check-prerequisites.sh` | 필수 파일 확인 | /speckit.tasks, /speckit.implement |
| `common.sh` | 공통 함수 (경로 계산) | 모든 스크립트에서 사용 |

### 3. AI와 스크립트의 협업

```
AI의 역할:
  - 슬래시 커맨드 읽기
  - 사용자 입력 분석
  - 스크립트 실행 결정
  - JSON 파싱
  - 문서 작성 (spec.md, plan.md, tasks.md)
  - 코드 생성

스크립트의 역할:
  - Git 조작 (브랜치 생성, 확인)
  - 파일 시스템 조작 (디렉토리, 템플릿 복사)
  - 환경 검증 (Git, 파일 존재 확인)
  - 구조화된 데이터 출력 (JSON)

→ AI는 "사고", 스크립트는 "실행"
```

### 4. 파일 흐름 (Templates → Specs)

```
templates/                         specs/001-xxx/
├── spec-template.md      →        ├── spec.md
├── plan-template.md      →        ├── plan.md
├── tasks-template.md     →        ├── tasks.md
└── checklist-template.md →        └── checklists/requirements.md
```

### 5. 왜 이런 순서인가?

**Spec → Plan 분리**
- Spec: "사진 앨범 정리 앱" (비기술적)
- Plan: "React + Node.js + S3" (기술적)
- 이유: 동일한 spec으로 여러 기술 스택 시도 가능

**Plan → Tasks 분리**
- Plan: "REST API 만들기" (전략)
- Tasks: "POST /api/tasks 엔드포인트 구현" (전술)
- 이유: AI가 한 번에 너무 많은 결정을 하지 않도록

**Tasks → Implement 분리**
- Tasks: "무엇을 할지" (체크리스트)
- Implement: "실제 실행" (코드 생성)
- 이유: 작업 계획과 실행을 분리하여 검토 가능

---

## 📊 파일 생성 타임라인

```
시간 →

specify init
├─ .claude/commands/*.md
├─ .specify/templates/*.md
└─ .specify/memory/constitution.md (템플릿)

/speckit.constitution
└─ .specify/memory/constitution.md (채워짐)

/speckit.specify
├─ specs/001-xxx/spec.md
└─ specs/001-xxx/checklists/requirements.md

/speckit.plan
├─ specs/001-xxx/plan.md
├─ specs/001-xxx/research.md
├─ specs/001-xxx/data-model.md
├─ specs/001-xxx/contracts/
└─ specs/001-xxx/quickstart.md

/speckit.tasks
└─ specs/001-xxx/tasks.md

/speckit.implement
├─ src/                          # 실제 코드
├─ .gitignore
├─ package.json
└─ 기타 설정 파일들
```

