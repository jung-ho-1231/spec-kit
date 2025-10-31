# Spec Kit 핵심 작동 메커니즘

## 1. 전체 데이터 플로우

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 사용자가 specify init my-project 실행                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Python CLI (src/specify_cli/__init__.py)                 │
│    - AI 에이전트 감지 (claude, gemini, copilot 등)           │
│    - GitHub API 호출 (템플릿 다운로드)                        │
│    - 프로젝트 디렉토리 생성                                    │
│    - 템플릿 파일 복사                                         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. 프로젝트 구조 생성                                          │
│    .claude/commands/    (또는 .github/, .gemini/ 등)         │
│    .specify/                                                 │
│      ├── memory/constitution.md                             │
│      ├── scripts/                                            │
│      ├── specs/                                              │
│      └── templates/                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. 사용자가 AI 에이전트에서 /speckit.specify 실행             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. AI 에이전트가 commands/specify.md 읽기                     │
│    - Frontmatter에서 스크립트 경로 파싱                       │
│    - 프롬프트 본문을 AI 컨텍스트에 로드                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. AI가 지시사항에 따라 작업 수행                             │
│    Step 1: 브랜치 이름 생성 (short-name)                     │
│    Step 2: 기존 브랜치 확인 (remote + local + specs)         │
│    Step 3: 스크립트 실행                                      │
│            scripts/bash/create-new-feature.sh \             │
│              --json --number 5 --short-name "user-auth" \   │
│              "Add user authentication"                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. Bash 스크립트 실행                                         │
│    - Git 브랜치 생성 (001-user-auth)                         │
│    - specs/001-user-auth/ 디렉토리 생성                      │
│    - spec.md 파일 초기화 (템플릿 복사)                       │
│    - JSON 출력 (BRANCH_NAME, SPEC_FILE 경로)                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 8. AI가 JSON 파싱 후 계속 작업                                │
│    - templates/spec-template.md 읽기                        │
│    - 사용자 입력 분석                                         │
│    - 명세서 작성 (User Scenarios, Requirements 등)           │
│    - specs/001-user-auth/spec.md에 저장                     │
│    - checklists/requirements.md 생성 (품질 검증)            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 9. 사용자가 /speckit.plan 실행                                │
│    - 동일한 메커니즘으로 plan.md 생성                         │
│    - setup-plan.sh 실행                                      │
│    - data-model.md, contracts/, research.md 생성            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 10. 사용자가 /speckit.tasks 실행                              │
│     - tasks.md 생성 (실행 가능한 작업 분해)                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 11. 사용자가 /speckit.implement 실행                          │
│     - tasks.md 파싱                                          │
│     - 순차/병렬 실행                                          │
│     - 실제 코드 생성                                          │
└─────────────────────────────────────────────────────────────┘
```

## 2. 핵심 메커니즘 상세

### 메커니즘 A: 슬래시 커맨드 → 스크립트 연결

```yaml
# templates/commands/specify.md
---
description: Create or update the feature specification
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---
```

**작동 방식**:
1. AI 에이전트가 frontmatter 파싱
2. 현재 OS/환경에 맞는 스크립트 선택 (sh vs ps)
3. {ARGS}를 사용자 입력으로 치환
4. Bash 도구로 스크립트 실행
5. JSON 출력 파싱하여 후속 작업 진행

### 메커니즘 B: 자동 번호 부여

```bash
# scripts/bash/create-new-feature.sh에서

# 1. Remote 브랜치 체크
git ls-remote --heads origin | grep -E "refs/heads/[0-9]+-${SHORT_NAME}$"

# 2. Local 브랜치 체크  
git branch | grep -E "^[* ]*[0-9]+-${SHORT_NAME}$"

# 3. Specs 디렉토리 체크
ls .specify/specs/ | grep -E "^[0-9]+-${SHORT_NAME}$"

# 4. 세 곳에서 가장 큰 번호 찾기
MAX_NUM=$(최대값 계산)

# 5. 다음 번호 사용
NEXT_NUM=$((MAX_NUM + 1))
```

**이유**: Git 브랜치, 로컬 브랜치, 파일 시스템이 동기화되지 않을 수 있으므로 모든 소스 체크

### 메커니즘 C: AI 컨텍스트 관리

```markdown
# update-agent-context.sh가 생성하는 CLAUDE.md (또는 GEMINI.md 등)

## Active Feature
- Branch: 001-user-auth
- Spec: specs/001-user-auth/spec.md
- Status: Planning phase

## Available Commands
- /speckit.specify - Create specification
- /speckit.plan - Create implementation plan
- ...

## Current Context
[현재 작업 중인 기능의 정보]
```

**목적**: AI가 현재 작업 컨텍스트를 유지하도록 지원

## 3. 설계 의도 추론

### 왜 이런 구조인가?

**1. 왜 Python CLI + Bash 스크립트 분리?**
```
Python CLI (specify):
  - 장점: 크로스 플랫폼, GitHub API, 복잡한 로직
  - 역할: 프로젝트 초기화, 템플릿 다운로드
  
Bash 스크립트:
  - 장점: Git 조작, 파일 시스템 작업에 최적
  - 역할: 브랜치 생성, 파일 조작
  
→ 각 도구의 강점 활용
```

**2. 왜 슬래시 커맨드는 마크다운?**
```
마크다운 장점:
  - AI 에이전트가 이해하기 쉬움
  - 사람이 읽고 수정하기 쉬움
  - Frontmatter로 메타데이터 정의
  - 버전 관리 용이
  
→ AI-first 설계
```

**3. 왜 Git 브랜치 기반?**
```
브랜치별 기능 관리:
  - 001-user-auth
  - 002-payment-system
  - 003-admin-dashboard
  
장점:
  - 기능별 격리
  - 병렬 개발 가능
  - 롤백 용이
  - 리뷰 프로세스 자연스러움
  
→ 현대 개발 워크플로우와 자연스럽게 통합
```

**4. 왜 spec → plan → tasks → implement 순서?**
```
spec (WHAT & WHY):
  - 기술 스택 독립적
  - 비즈니스 가치 중심
  - 검증 가능한 요구사항
  
plan (HOW):
  - 기술 스택 결정
  - 아키텍처 설계
  - 구현 전략
  
tasks (EXECUTE):
  - 작업 분해
  - 의존성 관리
  - 순서 정의
  
implement (BUILD):
  - 실제 코드 생성
  - 테스트
  - 통합
  
→ AI가 각 단계에서 집중할 포인트 명확화
```

## 4. 확장 포인트

### 커스터마이징 가능한 부분

1. **새 슬래시 커맨드 추가**
   - `templates/commands/mycommand.md` 생성
   - Frontmatter에 스크립트 연결
   - `.claude/commands/` (또는 해당 에이전트 폴더)에 배포

2. **템플릿 수정**
   - `templates/*-template.md` 수정
   - 섹션 추가/제거/재구성

3. **스크립트 확장**
   - `scripts/bash/` 또는 `scripts/powershell/`
   - 새 스크립트 추가
   - common.sh 유틸리티 활용

4. **AI 에이전트 추가**
   - `src/specify_cli/__init__.py`의 AGENT_CONFIG에 추가
   - 해당 에이전트의 명령어 형식 지원

## 5. 학습 전략

### 빠른 시작 (1시간)
1. README.md 정독
2. specify init test-project 실행
3. /speckit.specify로 간단한 기능 생성
4. 생성된 파일들 확인

### 깊이 있는 이해 (1일)
1. src/specify_cli/__init__.py 전체 읽기
2. commands/specify.md + create-new-feature.sh 분석
3. 템플릿 구조 파악
4. 직접 수정 실험

### 마스터 (2-3일)
1. 모든 슬래시 커맨드 분석
2. 커스텀 워크플로우 구축
3. 유사 도구 개발 시도
4. 이슈/PR 기여
