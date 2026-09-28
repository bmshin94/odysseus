# Odysseus 전수조사 & 활용 전략 정리 (한국어)

> 이 문서는 Odysseus 저장소를 전수조사한 결과와, 활용·학습·수익화 방향을 한국어로 정리한 노트입니다.
> 작성일: 2026-09-28

---

## 0. 저장소 주소

| 구분 | 주소 |
|---|---|
| 공식(업스트림) | https://github.com/odysseus-dev/odysseus |
| 이 저장소(포크) | https://github.com/bmshin94/odysseus |
| 랜딩 페이지 | https://odysseus-dev.github.io/odysseus/ |
| 설치 가이드 | [website/setup.md](../website/setup.md) |
| 기여 가이드 | [CONTRIBUTING.md](../CONTRIBUTING.md) · [ROADMAP.md](../ROADMAP.md) |

라이선스: **AGPL-3.0-or-later**

---

## 1. 이게 뭐 하는 프로젝트인가

**Odysseus = 자체 호스팅(self-hosted) 올인원 AI 워크스페이스.**

채팅·에이전트·딥리서치·문서·이메일·노트·캘린더·로컬 모델 서빙을 하나의
FastAPI 애플리케이션 안에 통합한 프로젝트. 클라우드 종속 없이 내 컴퓨터/서버에서
내 데이터로 동작하는 것이 핵심 가치.

### 규모 실측

| 항목 | 수치 |
|---|---|
| Python 파일 | 1,157개 |
| Python 총 라인 | 약 247,751줄 |
| JS 파일 | 178개 (`static/js` 96 모듈) |
| 테스트 파일 | 814개 |
| 라우터 모듈 | 약 70개 (`routes/`) |
| 최대 파일 | `src/agent_loop.py` (6,453줄) |

### 커뮤니티 지표 (2026-09 기준)

- GitHub ⭐ 약 87,600 / 포크 993 / 오픈 이슈 약 1,100
- 출시 48시간 만에 30,000 스타, 3주 만에 77,000 스타 돌파
- 제작자: Felix Kjellberg (PewDiePie), 2026-05-31 공개

---

## 2. 폴더 구조 해부

| 경로 | 역할 |
|---|---|
| `app.py` | FastAPI 앱 진입점 (약 56KB) |
| `src/` | 핵심 로직. `agent_loop.py`, `llm_core.py`, `deep_research.py`, `rag_manager.py`, `memory.py`, `mcp_manager.py`, `task_scheduler.py` 등 |
| `src/agent_tools/` | 에이전트 실행 툴 (filesystem, web, coding, subprocess, document, admin, bg_job) |
| `src/tools/` | 기능 툴 (image, calendar, search, contacts, notes, vault, cookbook, research) |
| `src/model_capability_readers/` | provider별 모델 능력 파악 (openai, google, ollama, llamacpp, lmstudio, openrouter) |
| `routes/` | HTTP API 약 70개 모듈 (chat, email 6,152줄, cookbook 4,583줄, calendar, gallery, skills, shell, mcp, vault, webhook 등) |
| `services/` | 독립 서비스 계층 (search, research, tts, stt, shell, youtube, hwfit) |
| `mcp_servers/` | **내장 MCP 서버** — email / memory / rag / image_gen |
| `integrations/claude` | Claude Code용 **Skill 번들** (`SKILL.md` + `odysseus_api.py`) |
| `integrations/codex` | Codex용 **플러그인** (`.codex-plugin/plugin.json`) |
| `core/` | 기반 계층 — auth, database, session_manager, middleware, platform_compat |
| `static/` | 프론트엔드 (바닐라 JS 96모듈 + `style.css`) |
| `companion/` | LAN 클라이언트(휴대폰) 디스커버리·페어링 브리지 |
| `scripts/odysseus-*` | CLI 도구 22종 (mail, cookbook, memory, tasks, notes, research 등) |
| `specs/` | **코딩 에이전트용 서브시스템 명세 24종** |
| `swift/` | Apple MLX 이미지 생성 브리지 |
| `tests/` | 테스트 814개 (보안 회귀 테스트 1,546줄 포함) |

---

## 3. 핵심 기능 8가지

1. **Chat + Agents** — 로컬/API 모델 + 툴 + MCP + 파일 + 셸 + 스킬 + 메모리
2. **Cookbook** — 하드웨어 스캔 → 모델 추천 → 다운로드 → vLLM/llama.cpp/SGLang/Ollama 서빙 (킬러 기능)
3. **Deep Research** — 다단계 웹 리서치 + 출처 기반 리포트 생성
4. **Compare** — 블라인드 A/B 모델 비교 및 합성
5. **Documents** — AI 편집/제안 지원 에디터 (Markdown / HTML / CSV)
6. **Email** — IMAP/SMTP 인박스, 분류·태그·요약·답장 초안
7. **Notes / Tasks / Calendar** — 리마인더, 할 일, 예약 에이전트 작업, CalDAV 동기화
8. **Extras** — 갤러리/이미지 에디터, 테마, 업로드, 웹검색, 프리셋, 세션, 2FA

---

## 4. 설치 및 사용법

### Docker (권장)

```bash
git clone https://github.com/odysseus-dev/odysseus.git
cd odysseus
cp .env.example .env
docker compose up -d --build
```

- 접속: `http://localhost:7000`
- 최초 관리자 비밀번호: `docker compose logs odysseus`에 출력
- 함께 뜨는 컨테이너: `odysseus`, `chromadb`, `searxng`, `ntfy`

### 네이티브 (Python 3.11+)

```bash
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python setup.py
python -m uvicorn app:app --host 127.0.0.1 --port 7000
```

Cookbook 사용 시 `tmux` 필요.

### Apple Silicon

```bash
./start-macos.sh        # http://127.0.0.1:7860
./build-macos-app.sh    # .app 래퍼 빌드
```

macOS의 Docker는 Metal GPU에 접근할 수 없으므로 M시리즈에서는 네이티브 실행 권장.

### Windows

`launch-windows.ps1`, `build-windows-portable.ps1`, `update_windows.bat` 제공.

### GPU 오버레이

```env
COMPOSE_FILE=docker-compose.yml:docker/gpu.nvidia.yml
# 또는
COMPOSE_FILE=docker-compose.yml:docker/gpu.amd.yml
```

진단: `scripts/check-docker-gpu.sh`, `scripts/check-docker-amd-gpu.sh`

### 보안 기본값 (반드시 유지)

```env
AUTH_ENABLED=true
LOCALHOST_BYPASS=false
APP_BIND=127.0.0.1
```

셸 실행 툴을 포함하는 애플리케이션이므로 공인 인터넷에 직접 노출 금지.

---

## 5. 플러그인인가, 스킬인가, MCP인가

**셋 다 해당된다.** Odysseus 본체는 독립 실행형 웹 애플리케이션이고,
그 주변에 세 가지 확장 레이어가 붙어 있다.

| 레이어 | 위치 | 방향 |
|---|---|---|
| MCP 호스트 (외부 MCP 서버 사용) | `src/mcp_manager.py`, `routes/mcp_routes.py` | 밖 → 안 |
| MCP 서버 제공 (email/memory/rag/image_gen) | `mcp_servers/` | 안 → 밖 |
| Claude Code **Skill** 번들 | `integrations/claude/skills/odysseus/SKILL.md` | Claude → Odysseus |
| Codex **플러그인** | `integrations/codex/.codex-plugin/plugin.json` | Codex → Odysseus |

```
[Claude Code] --(Skill)-->  [Odysseus 본체]  --(MCP)--> [외부 MCP 서버]
[Codex]     --(Plugin)-->        ↑ ↓
                           [내장 MCP 서버 4개]
```

### Claude Code 연동 절차

1. Odysseus → Settings → Integrations → Add Claude Agent
2. 발급된 토큰 복사, 허용할 툴 토글 설정
3. 터미널에서:

```bash
export ODYSSEUS_URL=http://your-host:7000
export ODYSSEUS_API_TOKEN=ody_generated_token
mkdir -p ~/.claude
curl -fsSL -H "Authorization: Bearer $ODYSSEUS_API_TOKEN" \
  "$ODYSSEUS_URL/api/claude/plugin.zip" -o /tmp/odysseus-claude-skill.zip
python3 -m zipfile -e /tmp/odysseus-claude-skill.zip ~/.claude/
```

---

## 6. API 토큰 정책

### 외부 API 토큰 — 선택 사항

로컬 모델(Ollama / llama.cpp / vLLM / LM Studio / SGLang)만 사용하면 **외부 토큰 0개**로 동작.
웹검색은 내장 SearXNG, 임베딩은 `fastembed`(로컬 ONNX)로 처리되어 추가 비용이 없다.

지원 외부 provider: OpenAI, Anthropic, Google, Groq, xAI, OpenRouter, DeepSeek, Copilot, ChatGPT 구독 연동.
키는 `src/secret_storage.py` / `src/api_key_manager.py` 경로로 암호화 저장된다.

### 내부 토큰 — 에이전트 연동 시 필수

Claude Code / Codex 연동 시 Odysseus가 자체 발급하는 스코프 토큰(`ody_...`)이 필요하다.

허용 스코프 (`routes/api_token_routes.py`):

```
chat (기본)
todos:read / todos:write
documents:read / documents:write
email:read / email:draft / email:send
calendar:read / calendar:write
memory:read / memory:write
cookbook:read / cookbook:launch
```

모든 스코프는 서버 측에서 검증되며, 토글이 꺼져 있으면 `403`을 반환한다.

---

## 7. 왜 GitHub에서 유명한가

1. **제작자** — 유튜버 PewDiePie(Felix Kjellberg)가 직접 만든 오픈소스
2. **성장 속도** — 48시간 3만 스타 → 6주 8.3만 스타 → 현재 8.7만 (글로벌 랭크 #177), 해당 여름 최다 스타 신규 레포
3. **기능 범위** — 경쟁 셀프호스팅 프로젝트 대부분이 "채팅 UI"에 머무는 반면, 메일·캘린더·리서치·모델 서빙까지 포함
4. **Cookbook** — "내 하드웨어에 맞는 모델 추천 + 자동 서빙"은 희소한 기능
5. **완성도** — 테스트 814개, `THREAT_MODEL.md`, 스코프 기반 권한 모델
6. **솔직한 로드맵 톤** — "I don't know what I'm doing, help", "static/style.css basically Calypso's island atm" 등의 문구가 기여자 유입을 촉진

---

## 8. 로컬 에이전트 구축에 주는 가치

| 해결해야 할 문제 | Odysseus의 구현 | 참고 파일 |
|---|---|---|
| 에이전트 루프 설계 | 완성형 루프 6,453줄 | `src/agent_loop.py` |
| 툴 스키마 / 파싱 | 툴 약 60종 정의 + 파서 | `src/tool_schemas.py`, `src/tool_parsing.py` |
| 작은 모델의 컨텍스트 부족 | 예산 관리 + 압축 | `src/context_budget.py`, `src/context_compactor.py` |
| 툴 과다로 인한 혼선 | 시맨틱 툴 검색 | `src/tool_index.py` |
| 위험 툴 승인 | 승인 게이트 + 스코프 | `src/tool_approvals.py`, `src/tool_approval_scopes.py` |
| 프롬프트 인젝션 | 전용 방어 모듈 | `src/prompt_security.py`, `src/tool_security.py` |
| 메모리 / RAG | 벡터 + 키워드 폴백 | `src/memory_vector.py`, `src/rag_manager.py` |
| provider별 기능 차이 | capability 리더 | `src/model_capability_readers/` |
| SSRF / URL 안전 | 전용 가드 | `src/url_safety.py`, `src/outbound_fetch.py` |
| 백그라운드 작업 | 스케줄러 + 잡 | `src/task_scheduler.py`, `src/bg_jobs.py` |

`specs/` 폴더는 "humans and **coding agents**"를 대상으로 작성된 구현 진실 맵으로,
AI 친화적 저장소 설계의 좋은 참고 사례다.

### 추천 학습 순서

1. `specs/_readme.md` → `specs/agent-tools.md` → `specs/context-building.md`
2. `src/agent_loop.py` 정독
3. `src/tool_schemas.py` + `src/agent_tools/`
4. `src/mcp_manager.py` + `mcp_servers/`
5. `routes/api_token_routes.py` + `src/tool_security.py`

---

## 9. React / PHP 재구현 가능성

### 프론트엔드 → React: 가능하며 오히려 개선 방향

현재 프론트는 바닐라 JS 96모듈 + 대형 `style.css` 구조이고, 백엔드가 FastAPI REST/SSE로
분리되어 있어 프론트만 교체하는 접근이 가능하다. ROADMAP도 CSS 정리·모달 위치·모바일
폴리시를 리팩터 우선순위로 명시하고 있어 수요가 확인된 영역이다.

> 실전 팁: 전체 재작성 대신 `/chat` 또는 `/documents` 한 화면만 React로 시범 이식.

### 백엔드 → PHP: 가능하지만 권장하지 않음

| 기능 | PHP 적합도 |
|---|---|
| CRUD (문서/메일/노트) | 양호 (Laravel) |
| SSE 스트리밍 | 가능하나 번거로움 |
| 장시간 에이전트 루프 | 부적합 (워커 큐 별도 필요) |
| 로컬 모델 서빙 | 매우 부적합 (생태계가 Python) |
| 임베딩 / 벡터 / RAG | 부적합 (fastembed, chromadb = Python) |
| MCP SDK | 공식 SDK 없음 (Python/TS만) |

**권장 조합**

```
프론트엔드 : React / Next.js
백엔드     : FastAPI (Python) 유지
PHP        : 랜딩·결제·회원·관리자 등 AI 외 사업 레이어 (Laravel)
```

---

## 10. 수익화 아이디어

### 전제: AGPL-3.0 이해

- 가능: 상업적 사용, 판매, SaaS 운영, 수정
- 조건: 네트워크로 서비스 제공 시 **수정된 소스 공개 의무**
- 불가: 소스를 숨긴 폐쇄형 SaaS

→ 따라서 "코드"가 아니라 **서비스·시간·전문성·하드웨어**를 파는 모델이 적합하다.

### 티어 1 — 즉시 시작 가능 (초기비용 ≒ 0)

| # | 아이디어 | 내용 | 난이도 / 수익성 |
|---|---|---|---|
| 1 | 설치·구축 대행 | 원격으로 Docker+GPU+모델 세팅 완료. 회당 15~40만원 | 하 / 중 |
| 2 | 한국어 콘텐츠 선점 | 유튜브·블로그·전자책. 애드센스 + 제휴 + 전자책 판매 | 하 / 중 |
| 3 | 유료 강의 | "실전 코드로 배우는 AI 에이전트 아키텍처". 8~15만원 | 중 / 상 |
| 4 | 컨트리뷰터 트랙 | PR 머지 → 이력 강화 → 이직·프리랜스 단가 상승 | 중 / 간접 |

로드맵 최상단이 "Fresh install smoke tests on Linux, macOS, Windows"라는 점은
설치 난이도가 곧 시장이라는 뜻이며, 오픈 이슈 약 1,100개는 기여 기회가 많다는 신호다.

### 티어 2 — 사업화 단계

| # | 아이디어 | 내용 | 난이도 / 수익성 |
|---|---|---|---|
| 5 | 매니지드 호스팅 | 대행 운영 + 백업/업데이트/모니터링. 월 3~10만원 (AGPL 준수 필수) | 상 / 상 |
| 6 | **업종 특화 버전** | 병원·법무·학원·제조 등 민감 데이터 업종 대상 구축. 500만~2,000만원 | 상 / 최상 |
| 7 | React UI 리디자인 | 커뮤니티 인지도 → 용역 수주 발판 | 중 / 중 |
| 8 | 플러그인·스킬 마켓 | 국내 특화 MCP 서버(세무·부동산·커머스 등) | 중 / 중 |

6번이 가장 유망한 이유: 민감 데이터 업종은 클라우드 사용이 법·규정상 제약되는 경우가 많고,
Odysseus는 애초에 셀프호스팅 구조라 그 제약을 해소한다. 코드는 공개하더라도
도메인 노하우·구축·유지보수는 판매 가능하다.

### 티어 3 — 하드웨어 결합형

| # | 아이디어 | 내용 | 난이도 / 수익성 |
|---|---|---|---|
| 9 | "AI 미니PC" 완제품 | 미니PC/Mac mini + 세팅 완료 + 한글 매뉴얼. 원가 + 20~40만원 | 상 / 상 |
| 10 | 기업 온프레미스 SI | 구축비 + 연 유지보수(구축비의 15~20%). 프로젝트당 수천만원 | 최상 / 최상 |

### 실행 로드맵 제안

```
1개월차   : 설치 완주 + 한국어 콘텐츠 3건 제작
2~3개월차 : 코드 정독(agent_loop.py, specs/) + PR 1건 머지
4~6개월차 : React 프론트 개선 또는 한국어 i18n 공개
6~12개월차: 업종 특화 레퍼런스 1곳 구축 → 본격 수익 진입
```

집중 추천 조합: **2번(콘텐츠) + 6번(업종 특화)**.
콘텐츠로 신뢰를 쌓고, 그 신뢰로 고객을 확보하는 구조.

> 주의: AGPL 기반 상업 활용, 특히 SaaS 및 재배포는 반드시 법률 전문가의 검토를 받을 것.
> 본 문서는 기술 분석 노트이며 법률 자문이 아니다.

---

## 11. 참고 링크

- 저장소: https://github.com/odysseus-dev/odysseus
- 포크: https://github.com/bmshin94/odysseus
- 랜딩 페이지: https://odysseus-dev.github.io/odysseus/
- 스타 히스토리: https://www.star-history.com/odysseus-dev/odysseus/
- 소개 기사: https://www.mindstudio.ai/blog/what-is-odysseus-pewdiepie-open-source-ai-workspace
- 커뮤니티 정리: https://daily.dev/posts/odysseus-a-free-self-hosted-ai-workspace-by-pewdiepie-99zakai5u
