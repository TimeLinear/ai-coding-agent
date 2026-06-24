# **🤖 AI Coding Agent (Windows CLI)**

**🚧 현재 상태 안내 (Project Status)**

본 프로젝트는 현재 **초기 부트스트래핑 및 뼈대 구축(Phase 1\) 단계**에 있습니다. 코어 모듈의 디렉토리 구조와 시스템 원칙(Constitution)이 정의되었으며, 각 파이프라인의 상세 로직을 구현해 나가는 중입니다.

**AI Coding Agent**는 Windows CLI 환경에서 구동되는 개인 맞춤형 자율 코딩 에이전트입니다. 제한된 LLM API 토큰 비용 내에서 최고의 정확도를 달성하며, **Git Worktree를 활용한 완벽한 샌드박스 격리**를 통해 사용자의 로컬 작업 환경을 안전하게 보호합니다.

## **✨ 핵심 기능 (5대 파이프라인)**

앞으로 구현될 본 에이전트의 핵심 파이프라인은 다음과 같습니다.

1. **지능형 컨텍스트 주입 (AST Repo Map)**  
   * tree-sitter를 활용해 전체 코드를 읽지 않고 클래스/함수 시그니처 중심의 요약된 지도를 LLM에 제공하여 토큰 비용을 최소화합니다.  
2. **Git Worktree 기반 안전한 격리**  
   * 코드를 수정하거나 테스트할 때 메인 브랜치가 오염되지 않도록 일회성 Worktree를 생성하여 안전한 도마(Sandbox) 위에서 작업합니다.  
3. **역순 패치 (Reverse Patching) 엔진**  
   * 파일 전체 재작성을 방지하고 줄 번호 기반의 Search/Replace를 수행합니다. 하단에서 상단으로 패치하여 줄 밀림 현상을 원천 봉쇄합니다.  
4. **다단계 자가 검증 루프**  
   * 비용이 발생하지 않는 로컬 환경에서 sloppylint(정적 검사), LSP(문맥 검사), 사용자 정의 테스트(런타임)를 통해 스스로 에러를 교정합니다. (최대 3회)  
5. **시각적 리뷰 및 통제권 보장**  
   * 모든 수정 사항은 반영 전 VSCode code \--diff (또는 CLI fallback)를 통해 사용자에게 검토되며, 승인 시에만 커밋 및 푸시됩니다.

## **🛠️ 기술 스택 및 아키텍처 원칙**

본 프로젝트는 시스템 안정성과 보안을 위해 엄격한 [**Constitution(원칙 가이드)**](http://docs.google.com/constitution.md) 에 따라 개발됩니다.

* **OS / 언어:** Windows / Python 3.11+ (UTF-8 모드 강제)  
* **패키지 관리:** uv  
* **주요 라이브러리:**  
  * GitPython: Worktree 제어 및 상태 롤백  
  * tree-sitter-builds: 의존성 분석 및 AST 파싱  
  * tenacity: 지수 백오프 기반 API 네트워크 에러 방어  
  * keyring: **절대적인 보안**을 위해 .env 대신 Windows 자격 증명 관리자를 통한 API 키 로드  
  * configargparse: 우선순위 기반 설정 병합

## **📂 프로젝트 구조**

각 모듈은 단일 책임 원칙(SRP)을 철저히 지키며, 낮은 결합도를 유지합니다.

ai-coding-agent/  
├── .env                       \# 안전한 비민감 설정만 포함 (TEST\_CMD 등)  
├── pyproject.toml             \# uv 기반 프로젝트 설정 및 의존성 명세  
├── src/  
│   └── agent/  
│       ├── main.py            \# 진입점 및 에러 롤백 오케스트레이터  
│       ├── context\_engine.py  \# \[Pipe 1\] AST 기반 Repo Map 주입  
│       ├── workspace\_manager.py \# \[Pipe 2\] Git Worktree 격리 및 GC   
│       ├── patch\_engine.py    \# \[Pipe 3\] 역순 패치 엔진  
│       ├── validation\_pipeline.py \# \[Pipe 4\] 다단계 에러 검사  
│       ├── deployment\_manager.py  \# \[Pipe 5\] 시각적 리뷰 및 반영  
│       └── utils/  
│           ├── llm\_client.py  \# API 통신 (Rate Limit 방어)  
│           └── exceptions.py  \# Custom Error (DirtyStateError 등)  
└── tests/                     \# 독립 모듈 테스트

## **🚀 시작하기 (Bootstrapping)**

현재 뼈대가 구축된 상태에서 프로젝트를 초기화하고 실행 환경을 구성하는 방법입니다. (Windows PowerShell 기준)

### **1\. 가상환경 초기화 및 활성화**

uv 패키지 관리자를 사용하여 빠르고 격리된 환경을 구성합니다.

\# 1\. uv를 활용한 프로젝트 및 가상환경 초기화  
uv venv

\# 2\. 가상환경 활성화  
.venv\\Scripts\\activate

\# 3\. 의존성 동기화 및 패키지 설치 (pyproject.toml 기반)  
uv sync

### **2\. 보안 및 환경 변수 설정**

**주의:** 이 프로젝트는 API 키를 평문 파일에 저장하지 않습니다.

* **.env 파일:** TEST\_CMD="uv run pytest"와 같은 실행 명령어 등 비민감 설정만 저장합니다.  
* **API 자격 증명:** 향후 제공될 스크립트를 통해 keyring (Windows Credential Manager)에 안전하게 등록하여 사용해야 합니다.

## **🗺️ 개발 로드맵 (Roadmap)**

* \[x\] **Phase 0:** 기술 스택 검증, 부트스트래핑 및 아키텍처(Constitution) 설계 완료  
* \[🏃‍♂️\] **Phase 1: 기반 인프라 구축** \- 예외 처리 클래스 정의 및 Git Worktree 샌드박스 구현 (파일 잠금 해제 완벽 구현 목표)  
* \[ \] **Phase 2: 인지 및 통신** \- AST Repo Map 파서 및 견고한 LLM 클라이언트(지수 백오프) 구현  
* \[ \] **Phase 3: 핵심 로직** \- 역순 패치 엔진(Reverse Patching) 및 다단계 검증 파이프라인 연동  
* \[ \] **Phase 4: 오케스트레이션** \- 메인 컨트롤러 및 사용자 리뷰/배포 흐름 완성

## **⚠️ 개발자 주의 사항 (OS 종속적 제약)**

본 프로젝트 기여 시 다음 사항을 반드시 준수해야 합니다.

1. **인코딩 충돌 방지:** 파일 I/O 및 subprocess 호출 시 시스템 로캘(CP949)에 의존하지 말고 반드시 encoding='utf-8' 파라미터를 명시하세요.  
2. **Git 백그라운드 잠금 방지:** Windows 환경에서 GitPython 사용 시 \[Error 32\]를 막기 위해 반드시 with 구문(Context Manager)을 사용하거나 명시적으로 .close()를 호출해야 합니다.
