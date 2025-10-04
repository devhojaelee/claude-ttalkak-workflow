# Claude Workflow Automation

프로젝트별 Claude Code 워크플로우 자동화 시스템. Linear 이슈 관리와 Git 브랜치 전략을 자동화하여 개발 생산성을 극대화합니다.

## 📋 개요

이 시스템은 **Parent-Sub Issue 패턴**을 사용하여 복잡한 기능 개발을 체계적으로 관리합니다:

1. **Parent Issue**: 큰 기능 단위 (예: "이메일 인증 시스템")
2. **Sub Issues**: 작은 작업 단위 (예: "이메일 전송", "인증 코드 검증", "UI 구현")

각 Sub Issue는 독립적인 브랜치에서 자동으로 구현되며, 완료 후 Parent 브랜치에 통합됩니다.

## ✨ 주요 기능

- **자동 브랜치 생성**: Linear 이슈 정보를 기반으로 Parent/Sub 브랜치 자동 생성
- **병렬 작업**: Tmux를 활용한 여러 Sub Issue 동시 처리
- **Lock 기반 충돌 방지**: Linear 레이블을 통한 파일 충돌 사전 차단
- **자동 통합**: Sub 브랜치들을 Parent 브랜치에 자동 merge
- **자동 PR 생성**: Linear 정보를 기반으로 GitHub PR 자동 생성
- **프레임워크 독립적**: Flask, Next.js, Spring Boot 등 모든 프로젝트에 적용 가능

## 🚀 설치 방법

### 1. Git Submodule로 추가

기존 프로젝트에 이 workflow 시스템을 추가:

```bash
cd your-project
git submodule add https://github.com/devhojaelee/claude-ttalkak-workflow .claude-core
git submodule update --init --recursive
```

### 2. 설정 파일 생성

프로젝트 루트에 `.claude-config` 파일 생성:

```bash
# 프레임워크별 템플릿 선택
cp .claude-core/templates/.claude-config.flask .claude-config    # Flask
# or
cp .claude-core/templates/.claude-config.nextjs .claude-config   # Next.js
# or
cp .claude-core/templates/.claude-config.spring .claude-config   # Spring Boot
# or
cp .claude-core/templates/.claude-config.example .claude-config  # Custom
```

### 3. 설정 파일 편집

`.claude-config` 파일을 프로젝트에 맞게 수정:

```bash
# Linear 정보
LINEAR_TEAM="YourTeamName"           # Linear 팀 이름
LINEAR_PROJECT="YourProjectName"     # Linear 프로젝트 이름

# 프로젝트 정보
PROJECT_TYPE="flask"                 # 프로젝트 타입
LANGUAGE="python"                    # 프로그래밍 언어

# 검증 명령어
TEST_COMMAND="python -m pytest"      # 테스트 실행 명령어
BUILD_COMMAND=""                     # 빌드 명령어 (선택)
LINT_COMMAND="flake8 ."             # Lint 명령어 (선택)

# 환경 파일
ENV_FILE=".env.prod"                # LINEAR_API_KEY가 포함된 환경 파일
```

### 4. 환경 변수 설정

환경 파일(`.env`, `.env.prod` 등)에 `LINEAR_API_KEY` 추가:

```bash
LINEAR_API_KEY=lin_api_xxxxxxxxxxxxxxxxxxxxxx
```

## 📖 사용 방법

### 1단계: Parent Issue에 Sub Issue 생성

Linear에서 Parent Issue를 생성하고, 하위에 여러 Sub Issue 추가:

```
Parent Issue: 100P-123 "이메일 인증 시스템"
├─ Sub Issue: 100P-124 "이메일 전송 기능"
├─ Sub Issue: 100P-125 "인증 코드 검증"
└─ Sub Issue: 100P-126 "UI 구현"
```

### 2단계: 자동 구현 시작

```bash
# Parent Issue ID를 인자로 스크립트 실행
./.claude-core/scripts/2-ttalkak.sh 100P-123

# 또는 특정 base 브랜치 지정
./.claude-core/scripts/2-ttalkak.sh 100P-123 develop
```

실행 결과:
- Parent 브랜치 생성: `feature/100P-123-email-verification`
- Sub 브랜치들 자동 생성 및 Claude Code로 구현:
  - `feature/100P-124-email-send`
  - `feature/100P-125-code-verify`
  - `feature/100P-126-ui-implement`
- Tmux 세션에서 우선순위 순으로 자동 실행

### 3단계: Tmux 세션 확인

```bash
# Tmux 세션 접속
tmux attach -t Claude-Worker-100P-123

# 윈도우 이동
# Ctrl+b, n (다음 윈도우)
# Ctrl+b, p (이전 윈도우)
# Ctrl+b, w (윈도우 목록)

# 세션 종료
# Ctrl+b, d (detach)
```

각 Sub Issue는:
1. Linear에서 Lock 레이블 확인 (파일 충돌 방지)
2. 구현 계획 수립
3. 코드 구현 및 자체 리뷰
4. 테스트 작성 및 실행
5. Git commit & push
6. Linear 상태를 "In Review"로 변경

### 4단계: Sub 브랜치 통합

모든 Sub Issue가 완료되면 Parent 브랜치에 통합:

```bash
./.claude-core/scripts/3-integrate.sh 100P-123
```

실행 결과:
- Parent 브랜치로 checkout
- main 브랜치에서 rebase (최신 상태 반영)
- Sub 브랜치들을 우선순위 순으로 merge
- 충돌 발생 시 Claude Code와 대화형으로 해결

### 5단계: 통합 테스트 및 Push

```bash
# 프로젝트별 테스트 실행
python -m pytest                      # Flask
npm test                              # Next.js
./mvnw test                           # Spring Boot

# 테스트 통과 시 push
git push origin feature/100P-123-email-verification
```

### 6단계: PR 자동 생성

```bash
./.claude-core/scripts/4-create-pr.sh 100P-123
```

실행 결과:
- Linear에서 Parent/Sub Issue 정보 조회
- PR body 자동 생성 (Summary, Features, Sub Issues, Test Plan)
- GitHub PR 자동 생성
- PR URL 출력

## 🔧 고급 기능

### Lock 기반 충돌 방지

동일한 파일을 수정하는 Sub Issue들은 Lock 레이블로 순서 보장:

```
Sub Issue 1: "app.py 수정" → Lock: app.py 획득 → In Progress
Sub Issue 2: "app.py 수정" → Lock: app.py 대기 → BLOCKED (30초 후 재시도)
```

Sub Issue 1이 완료되면 Lock 해제 → Sub Issue 2 자동 시작

### 프로젝트별 검증 커스터마이징

`.claude-config`의 `CLAUDE_WORKFLOW_CUSTOM` 변수로 추가 검증 단계 정의:

```bash
CLAUDE_WORKFLOW_CUSTOM="
- 타입 체크 실행 (tsc --noEmit)
- E2E 테스트 실행 (playwright test)
- 성능 벤치마크 확인
"
```

### Base 브랜치 변경

main 외의 브랜치에서 시작:

```bash
./.claude-core/scripts/2-ttalkak.sh 100P-123 develop
```

Parent/Sub 브랜치 모두 `develop` 브랜치에서 분기됩니다.

## 🏗️ 디렉토리 구조

```
.claude-core/               # Git submodule
├── scripts/
│   ├── 2-ttalkak.sh       # Sub Issue 자동 구현
│   ├── 3-integrate.sh     # Parent 브랜치 통합
│   └── 4-create-pr.sh     # PR 자동 생성
├── templates/
│   ├── .claude-config.example    # 일반 템플릿
│   ├── .claude-config.flask      # Flask 템플릿
│   ├── .claude-config.nextjs     # Next.js 템플릿
│   └── .claude-config.spring     # Spring Boot 템플릿
└── README.md

your-project/               # 실제 프로젝트
├── .claude-core/          # Git submodule (심볼릭 링크)
├── .claude-config         # 프로젝트별 설정
├── .env.prod              # LINEAR_API_KEY 포함
└── [프로젝트 파일들]
```

## 🎯 워크플로우 단계별 설명

### Phase 1: Context & Lock Management
- Linear Issue 정보 로드
- 수정할 파일 분석 및 Lock 레이블 생성
- 동시 작업 확인 및 Lock 획득/대기

### Phase 2: Implementation Planning
- 코드베이스 분석
- 영향받는 파일 목록 작성
- 구현 계획 수립

### Phase 3: Solve Issue
- 코드 구현
- 자체 코드 리뷰
- 테스트 작성

### Phase 4: Git Commit & Push + Validation
- 변경사항 Git commit
- 프로젝트별 검증 (테스트, 빌드, Lint)
- Git push (필수)

### Phase 5: Issue Management & Lock Release
- Linear 상태 "In Review"로 변경
- 구현 내용 comment 작성
- Lock 레이블 해제

## 🌐 지원 프레임워크

템플릿이 제공되는 프레임워크:

- **Flask** (Python): Web application
- **Next.js** (TypeScript): React framework
- **Spring Boot** (Java): Enterprise application

기타 프레임워크도 `.claude-config.example`을 복사하여 쉽게 추가 가능:

```bash
# Django 프로젝트 예시
PROJECT_TYPE="django"
LANGUAGE="python"
TEST_COMMAND="python manage.py test"
BUILD_COMMAND=""
LINT_COMMAND="flake8 ."
```

## 📦 필수 요구사항

### 시스템 도구
- **jq**: JSON 파싱 (`brew install jq`)
- **tmux**: 병렬 작업 (`brew install tmux`)
- **gh**: GitHub CLI (`brew install gh`)
- **git**: 버전 관리
- **curl**: API 호출

### 프로젝트 설정
- Linear 계정 및 API 키
- GitHub 레포지토리
- `.claude-config` 설정 파일
- Claude Code CLI 설치

## 🔍 문제 해결

### "LINEAR_API_KEY가 설정되지 않았습니다"
→ 환경 파일(`.env.prod` 등)에 `LINEAR_API_KEY` 추가

### "jq가 설치되지 않았습니다"
→ `brew install jq` 실행

### "Parent 브랜치로 전환 실패"
→ `2-ttalkak.sh`를 먼저 실행하여 Parent 브랜치 생성

### Tmux 세션이 보이지 않음
→ `tmux ls`로 세션 목록 확인

### Lock 레이블로 계속 BLOCKED
→ Linear에서 해당 Lock 레이블을 가진 "In Progress" 이슈 확인

## 📝 예시: Flask 프로젝트

```bash
# 1. Submodule 추가
git submodule add https://github.com/devhojaelee/claude-ttalkak-workflow .claude-core

# 2. Flask 템플릿 복사
cp .claude-core/templates/.claude-config.flask .claude-config

# 3. 설정 편집
vim .claude-config
# LINEAR_TEAM="100products"
# LINEAR_PROJECT="Coffeechat"

# 4. 환경 변수 설정
echo "LINEAR_API_KEY=lin_api_xxx" >> .env.prod

# 5. 워크플로우 실행
./.claude-core/scripts/2-ttalkak.sh 100P-123
tmux attach -t Claude-Worker-100P-123

# 6. 통합 및 테스트
./.claude-core/scripts/3-integrate.sh 100P-123
python -m pytest
git push origin feature/100P-123-email-verification

# 7. PR 생성
./.claude-core/scripts/4-create-pr.sh 100P-123
```

## 🤝 기여

이슈 제출, 기능 제안, PR은 언제나 환영합니다!

## 📄 라이선스

MIT License

## 🙏 감사의 말

이 워크플로우 시스템은 실제 프로덕션 환경에서 검증되었으며, Flask 기반 커피챗 예약 시스템 개발 과정에서 탄생했습니다.
