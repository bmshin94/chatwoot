# Chatwoot 레포지토리 전수조사 분석 정리

> 작성일: 2026-10-08
> 작성: Claude Code (카리나 페르소나) 세션 대화 정리
> 대상 레포: **https://github.com/bmshin94/chatwoot** (포크본)
> 원본(upstream): **https://github.com/chatwoot/chatwoot**
> 분석 기준 브랜치: `claude/modest-einstein-skv1xo` (base: `develop`) / 버전 `4.18.0`

---

## 목차

1. [이게 뭐야? — 정체 파악](#1-이게-뭐야--정체-파악)
2. [쉽게 다시 설명 — 비유편](#2-쉽게-다시-설명--비유편)
3. [질문 7개 답변](#3-질문-7개-답변)
4. [수익화 아이디어 상세](#4-수익화-아이디어-상세)
5. [부록 — 참고 링크 & 체크리스트](#5-부록--참고-링크--체크리스트)

---

## 1. 이게 뭐야? — 정체 파악

### 1.1 한 줄 정의

**Chatwoot** = 오픈소스 고객상담(고객 지원) 플랫폼.
Zendesk / Intercom / Salesforce Service Cloud / 채널톡 같은 상용 SaaS를 **셀프호스팅으로 대체**하는 제품.

- 레포 주소: https://github.com/bmshin94/chatwoot
- 원본 레포: https://github.com/chatwoot/chatwoot
- 버전: `4.18.0`
- 라이선스: **MIT** (단, `enterprise/` 디렉터리는 **별도 상용 라이선스**)
- 브랜칭 모델: git-flow, base 브랜치는 `develop`

### 1.2 규모 (실측치)

| 항목 | 수치 |
|---|---|
| Ruby 파일 | 1,380개 (약 87,700줄) |
| Vue 컴포넌트 | 1,187개 |
| JS 파일 | 1,118개 (Vue+JS 합계 약 291,000줄) |
| DB 마이그레이션 | 180개 |
| RSpec 스펙 파일 | 909개 |
| 코어 모델 | 57개 (+ enterprise 50개 이상) |
| 환경변수(.env.example) | 59개 |

> 개인 토이 프로젝트가 아니라, 수년간 다수 개발자가 만든 **상업용 SaaS 제품의 전체 소스**.

### 1.3 기술 스택

```
백엔드    : Ruby 3.4.4 + Rails 7.2.3.1
프론트엔드 : Vue 3 (Composition API, <script setup>) + Vite + Tailwind CSS
DB        : PostgreSQL 16 (pgvector 이미지 — AI 임베딩용)
캐시/큐    : Redis + Sidekiq (+ sidekiq-cron)
실시간     : ActionCable (WebSocket)
검색       : Searchkick + OpenSearch
인증       : Devise + devise_token_auth + devise-two-factor (+ SAML: EE)
권한       : Pundit
결제       : Stripe
어드민     : Administrate
배포       : Docker / Docker Compose / Heroku / DigitalOcean / Kubernetes(Helm)
모니터링   : Sentry, Datadog, New Relic, Elastic APM, Scout, Langfuse(LLM 추적)
API 문서   : OpenAPI 3.1 (swagger/ 디렉터리)
```

### 1.4 폴더 구조 전수조사

#### `app/` — 코어 (MIT, 무료)

| 경로 | 역할 |
|---|---|
| `app/models/` | 57개 핵심 테이블: `account`(멀티테넌트), `conversation`, `message`, `contact`, `inbox`, `agent_bot`, `automation_rule`, `campaign`, `portal`/`article`(헬프센터), `webhook`, `working_hour` 등 |
| `app/models/channel/` | **12개 채널 어댑터**: `web_widget`, `email`, `facebook_page`, `instagram`, `whatsapp`, `telegram`, `line`, `sms`, `twilio_sms`, `twitter_profile`, `tiktok`, `api` |
| `app/controllers/api/v1/` | REST API 전체 (accounts, conversations, contacts, inboxes, campaigns, reports, search 등) |
| `app/controllers/platform/` | **Platform API** — 계정·유저·봇을 프로그래밍으로 생성 (화이트라벨 재판매의 핵심) |
| `app/controllers/public/` | 헬프센터 포털 및 공개 API |
| `app/controllers/webhooks/` | 페이스북/인스타/왓츠앱/텔레그램/트윌리오 웹훅 수신 |
| `app/services/` | 60개 이상 비즈니스 로직 (자동배정, 필터, 리포트, CSAT, IMAP, Shopify, CRM, MFA 등) |
| `app/jobs/` | Sidekiq 비동기 작업 |
| `app/javascript/dashboard/` | 상담원용 메인 대시보드 SPA (Vue 3) |
| `app/javascript/dashboard/components-next/` | 신규 컴포넌트 (기존 `components/`는 단계적 폐기 중) |
| `app/javascript/widget/` | 웹사이트 임베드 채팅 위젯 |
| `app/javascript/sdk/` | `<script>` 한 줄 설치 SDK (사이즈 제한 40KB) |
| `app/javascript/portal/` | 헬프센터 공개 페이지 |
| `app/javascript/design-system/` | 자체 디자인 시스템 |
| `app/dashboards/` | Administrate 기반 슈퍼 어드민 |

#### `enterprise/` — 유료 오버레이 (상용 라이선스)

Rails의 `prepend_mod_with` / `include_mod_with` 패턴으로 코어를 덮어쓰는 구조.

| 기능 | 설명 |
|---|---|
| **Captain** | AI 에이전트. Assistant, Document(RAG), Scenario, Copilot, FAQ 자동 생성, Custom Tool(function calling) |
| **Voice** | Twilio 기반 전화 상담 + 통화 녹음 + STT |
| **SLA** | 응답시간 약정 관리 및 위반 알림 |
| **Custom Roles** | 세밀한 권한 관리 |
| **SAML SSO** | 기업용 싱글사인온 |
| **Audit Log** | 전 영역 변경 추적 |
| **Companies** | 고객사 단위 관리 |
| **Agent Capacity Policy** | 상담원 업무량 밸런싱 |

#### AI / LLM 관련 (핵심 자산)

```ruby
# lib/llm_constants.rb
DEFAULT_MODEL = 'gpt-4.1'
DEFAULT_EMBEDDING_MODEL = 'text-embedding-3-small'
PDF_PROCESSING_MODEL = 'gpt-4.1-mini'
OPENAI_API_ENDPOINT = 'https://api.openai.com'
PROVIDER_PREFIXES = {
  'openai'    => %w[gpt- o1 o3 o4 text-embedding- whisper- tts-],
  'anthropic' => %w[claude-],
  'google'    => %w[gemini-],
  'mistral'   => %w[mistral- codestral-],
  'deepseek'  => %w[deepseek-]
}
```

| 에이전트 구성요소 | 구현 위치 |
|---|---|
| 멀티 LLM 라우팅 | `lib/llm/feature_router.rb`, `lib/llm/models.rb`, `lib/llm_constants.rb` |
| RAG (벡터 검색) | `enterprise/app/models/article_embedding.rb` + pgvector + `Captain::Document` |
| Tool / Function Calling | `enterprise/app/services/captain/tools/custom_http_tool.rb`, `tool_registry_service.rb` |
| 웹 크롤링 수집 | `firecrawl_service.rb`, `simple_page_crawl_service.rb`, `html_page_parser.rb` |
| 세션/스레드 관리 | `Captain::AgentSession`, `CopilotThread`, `CopilotMessage` |
| LLM 관측(Observability) | `lib/integrations/llm_instrumentation*.rb` + Langfuse |
| 시나리오/워크플로우 | `Captain::Scenario` |
| 사람에게 에스컬레이션 | `v1_action_classifier.rb`, `v1_false_promise_handler.rb` |
| 환각/지시 검증 | `assistant_migration/instruction_auditor.rb` |
| 품질 평가 | `Captain::MessageReport`, `conversation_outcome_tracker.rb` |
| FAQ 자동 생성 | `captain/llm/conversation_faq_job.rb`, `faq_suggestion_approval_service.rb` |
| Copilot (상담원 보조) | `captain/copilot/reply_suggestion_job.rb`, `response_job.rb` |

**배울 만한 디테일 3가지**

1. 툴 개수/이름 제약 관리 — `Captain::CustomTool`
   ```ruby
   MAX_PER_ACCOUNT = 15   # 컨텍스트 폭발 방지
   MAX_SLUG_LENGTH = 64   # OpenAI 함수명 64자 제한
   ```
2. SSRF 방어 — `Concerns::SafeEndpointValidatable` + `ssrf_filter` gem
   (사용자 등록 URL로 AI가 요청할 때 내부망 공격 차단)
3. LLM 비용/품질 추적 — `ChatwootApp.otel_enabled?` + Langfuse 연동

#### 기타

| 경로 | 역할 |
|---|---|
| `swagger/` | OpenAPI 3.1 스펙 전체 (클라이언트 자동 생성 가능) |
| `lib/` | 공통 라이브러리 (LLM 라우터, 통합, 시더, 웹훅, 필터) |
| `spec/` | RSpec 909개 |
| `db/migrate/` | 180개 마이그레이션 (DB 설계 레퍼런스) |
| `docker/` | Dockerfile + 엔트리포인트 |
| `config/` | 라우트, i18n(다국어), Sidekiq 설정 |
| `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` | AI 코딩 에이전트용 가이드 문서 |

### 1.5 포크본에서 추가된 부분

```
648733d  Update CLAUDE.md
854d494  Add files via upload           → CLAUDE.md, GEMINI.md
46abe15  Delete CLAUDE.md
c661734  Merge pull request #1 (feat/claude-guide)
8df9809  docs: appended CLAUDE.md persona guide   (+158줄)
```

→ **애플리케이션 코드 변경 없음.** AI 에이전트 가이드 문서(`CLAUDE.md`, `GEMINI.md`)와
"카리나 페르소나" 섹션만 추가됨.

### 1.6 어떨 때 쓰는가

1. 옴니채널 고객상담 통합 센터가 필요할 때
2. Zendesk/Intercom 상담원 과금(월 $20~100/명)이 부담될 때
3. 고객 데이터를 외부 SaaS에 둘 수 없을 때 (금융/의료/공공 — 데이터 주권)
4. AI 상담봇(FAQ 자동응답 + RAG)을 만들고 싶을 때
5. 화이트라벨로 재판매하고 싶을 때 (Platform API)

### 1.7 내게 어떤 도움이 되는가

- **학습 교재**: Rails 7 + Vue 3 프로덕션 코드, 멀티테넌시, 권한 설계, 비동기, WebSocket, OpenSearch
- **DB/테스트 레퍼런스**: 마이그레이션 180개, 스펙 909개
- **SaaS 수익화 설계 패턴**: OSS/EE 분리(`prepend_mod_with`)
- **AI 에이전트 레퍼런스**: Captain 전체 (RAG, 툴, 관측, 에스컬레이션)
- **즉시 사용 가능한 제품**: Docker로 당일 고객센터 구축
- **합법적 수익화 자산**: MIT (단 `enterprise/` 제외)

---

## 2. 쉽게 다시 설명 — 비유편

### 2.1 핵심 비유

상용 고객상담 SaaS를 쓰는 건 **사무실을 월세로 임대**하는 것.
- 상담원 1명당 월 3~10만원, 10명이면 월 30~100만원
- 고객 데이터는 그 회사 서버에 있음

Chatwoot은 **사무실을 직접 지을 수 있는 설계도 + 자재 전부를 무료로 받은 것**.
- 내 서버에 설치 → 서버비만 (월 5만원 수준)
- 상담원 무제한
- 고객 데이터 100% 내 소유
- 브랜딩 자유 변경 가능 (MIT)

### 2.2 기능별 비유

**(1) 옴니채널 수신함 = 모든 우편물이 한 통으로**
```
홈페이지 채팅 ─┐
이메일        ─┤
왓츠앱        ─┼──→ [하나의 받은편지함] ──→ 상담원 1명이 전부 처리
인스타 DM     ─┤
페북 메신저   ─┤
텔레그램      ─┤
SMS          ─┘
```
원래는 앱 7개를 번갈아 켜야 하는 작업이 한 화면에서 끝남.

**(2) Captain(AI) = 우리 회사 문서만 읽는 신입 상담원**
```
고객 질문 → pgvector로 사내 FAQ/문서 벡터 검색 → 근거 기반 자동 답변
         → 어려운 질문이면 사람 상담원에게 에스컬레이션
```
사내 문서에 근거해 답하므로 환각(헛소리)이 줄어듦.

**(3) 헬프센터 = 셀프 안내 데스크** — FAQ를 공개해 문의 자체를 감소
**(4) 자동화/캠페인 = 자동 비서** — 자동 배정, 영업시간 외 자동응답, 온보딩 메시지 발송
**(5) 리포트 = 성적표** — 응답시간, 해결건수, 상담원별 성과, CSAT

### 2.3 폴더 구조를 집에 비유

```
chatwoot/
├── app/            집 본체 (무료, MIT)
│   ├── models/         벽돌·골조 (데이터 구조)
│   ├── controllers/    현관문 (API 입출구)
│   ├── services/       보일러·배관 (비즈니스 로직)
│   ├── jobs/           로봇청소기 (백그라운드 작업)
│   └── javascript/     인테리어 (화면 UI)
├── enterprise/     프리미엄 별관 (상용 라이선스 / Captain AI 등)
├── swagger/        설명서 (API 문서)
├── spec/           품질검사 (테스트 909개)
├── docker/         이사 패키지 (한 번에 설치)
└── CLAUDE.md       AI 에이전트 사용설명서
```

### 2.4 가장 중요한 주의사항 (라이선스)

```
app/         → MIT          → 상업 이용 / 수정 / 재판매 가능
enterprise/  → 상용 라이선스 → 유효한 구독 없이 "프로덕션 사용 금지"
                              (개발·테스트 목적 복사/수정은 허용)
```

`enterprise/LICENSE` 원문:
> "This software ... may only be used in production, if you (and any entity that you represent)
> have agreed to, and are in compliance with, the Chatwoot Subscription Terms of Service ...
> Notwithstanding the foregoing, you may copy and modify the Software for development and
> testing purposes, without requiring a subscription."

코드에 비활성화 스위치가 존재:
```ruby
# lib/chatwoot_app.rb
def self.enterprise?
  return if ENV.fetch('DISABLE_ENTERPRISE', false)
  @enterprise ||= root.join('enterprise').exist?
end
```
→ 상업적 이용 시 **`enterprise/` 디렉터리 삭제 또는 `DISABLE_ENTERPRISE=true`** 필수.

---

## 3. 질문 7개 답변

### Q1. 설치 및 사용법

#### 방법 A — Docker (권장)

```bash
git clone https://github.com/bmshin94/chatwoot.git
cd chatwoot

cp .env.example .env
# .env 최소 설정:
#   SECRET_KEY_BASE=$(openssl rand -hex 64)
#   FRONTEND_URL=http://localhost:3000
#   REDIS_PASSWORD=<임의 비밀번호>

docker compose run --rm rails bundle exec rails db:chatwoot_prepare
docker compose up

# 대시보드: http://localhost:3000
# 메일 확인(Mailhog): http://localhost:8025
```

> `docker-compose.yaml`의 Postgres 이미지는 **`pgvector/pgvector:pg16`**.
> AI 벡터검색(Captain)을 쓰려면 이 이미지(또는 pgvector 확장)가 반드시 필요.

#### 방법 B — 로컬 직접 설치 (개발용)

사전 준비: Ruby 3.4.4(rbenv), Node 24.13.0(nvm), pnpm, PostgreSQL+pgvector, Redis, overmind

```bash
rbenv install $(cat .ruby-version)
eval "$(rbenv init -)"        # CLAUDE.md 명시 필수 단계

make setup                     # bundle install + pnpm install
cp .env.example .env
make db                        # rails db:chatwoot_prepare
bundle exec rails db:seed      # 테스트 데이터

make run                       # overmind start -f Procfile.dev  (= pnpm dev)
```

#### 방법 C — 원클릭 배포
- Heroku 버튼 (README)
- DigitalOcean Marketplace 1-Click Kubernetes
- Helm 차트 (ArtifactHub)

#### 첫 사용 흐름
```
1. http://localhost:3000 → 첫 계정(슈퍼관리자) 생성
2. Settings > Inboxes > Add Inbox > Website
3. 발급된 <script> 스니펫을 내 홈페이지 <body> 끝에 삽입
4. Settings > Agents → 상담원 초대
5. Settings > Canned Responses → 자주 쓰는 답변
6. Settings > Automation → 자동 배정 규칙
7. /super_admin → 인스턴스 전역 설정 (LLM 키 등)
```

#### 개발 명령어 (CLAUDE.md 기준)
```bash
pnpm eslint / pnpm eslint:fix              # JS/Vue 린트
bundle exec rubocop -a                     # Ruby 린트 + 자동수정
pnpm test                                  # JS 테스트 (vitest)
bundle exec rspec spec/경로/파일_spec.rb    # Ruby 테스트
bundle exec rspec spec/파일_spec.rb:42      # 특정 라인
make console                               # Rails 콘솔
make debug                                 # overmind connect backend
pnpm story:dev                             # Histoire (컴포넌트 스토리)
```

---

### Q2. 플러그인? 스킬? MCP?

**결론: 셋 다 아니다. 완전한 독립 웹 애플리케이션(풀스택 SaaS 제품).**

`grep -ril "mcp"` 결과 — MCP 관련 코드 **0건**.

| 분류 | 해당 | 설명 |
|---|---|---|
| 플러그인 | X | 다른 앱에 끼우는 것이 아님. 자체가 메인 앱 (Rails 서버 + DB + Redis 필요) |
| Claude 스킬 | X | `.claude/skills/` 없음, `SKILL.md` 없음 |
| MCP 서버 | X | MCP 코드/설정 전무 |
| **독립 웹앱** | **O** | Rails 7 모놀리식 + Vue 3 SPA |

**헷갈린 이유**: 루트에 `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.windsurf/`, `.vscode/`가 있어서.
이들은 **AI 코딩 에이전트가 읽는 "컨텍스트/규칙 문서"**이며 플러그인·스킬이 아니다.

**Chatwoot 자체의 확장 포인트**

| 방식 | 설명 |
|---|---|
| Agent Bot | API 토큰으로 외부 봇을 수신함에 연결 (웹훅 수신 → 답장) |
| Webhook | 이벤트 발생 시 지정 URL로 POST |
| Dashboard App | 상담 화면 사이드바에 내 웹앱을 iframe으로 삽입 |
| Captain Custom Tool | AI가 호출할 HTTP API 등록 (function calling — MCP와 가장 유사) |
| Integrations | Slack, Dialogflow, Shopify, Linear, Google Translate 등 |
| Enterprise 오버레이 | `prepend_mod_with`로 코어 클래스 override |

> Chatwoot REST API가 완성돼 있으므로(`swagger/`), **Chatwoot MCP 서버를 직접 만드는 것은 가능**하다.
> (→ 4장 수익화 아이디어 #6)

---

### Q3. API 토큰을 사용해야 되는가

`swagger/index.yml`에 정의된 **토큰 3종류** (모두 헤더 `api_access_token` 사용):

| 토큰 | 발급 경로 | 권한 |
|---|---|---|
| `userApiKey` | 프로필 페이지 / rails 콘솔 | 해당 유저 권한 범위의 API |
| `agentBotApiKey` | 시스템 관리자 / rails 콘솔 | 봇 전용, 제한된 API |
| `platformAppApiKey` | 슈퍼관리자가 Platform App 생성 후 | 계정·유저·봇 프로비저닝 (최상위) |

#### 상황별 필요 여부

| 하려는 일 | 토큰 | 종류 |
|---|---|---|
| UI로만 상담 | 불필요 | — |
| 홈페이지 채팅 위젯 설치 | 불필요 | website_token(공개용, 자동 발급) |
| 외부 시스템에서 대화/연락처 조회·생성 | 필요 | `userApiKey` |
| 챗봇 구축 | 필요 | `agentBotApiKey` |
| 계정 대량 자동 생성 (화이트라벨) | 필요 | `platformAppApiKey` |
| **AI(Captain) 사용** | 필요 | **LLM 프로바이더 API 키 별도 (과금 발생)** |

#### AI 사용 시 (중요)
Chatwoot 토큰과 **별개로** LLM API 키가 필요하다.
Super Admin > Settings의 InstallationConfig(예: `CAPTAIN_OPEN_AI_API_KEY`)로 설정.
지원 프로바이더: OpenAI / Anthropic / Google / Mistral / DeepSeek.
→ **토큰 사용량만큼 비용이 발생**하므로 비용 상한 설계가 필요.

#### 사용 예시
```bash
# 대화 목록
curl -X GET 'http://localhost:3000/api/v1/accounts/1/conversations' \
  -H 'api_access_token: <TOKEN>'

# 메시지 전송
curl -X POST 'http://localhost:3000/api/v1/accounts/1/conversations/5/messages' \
  -H 'api_access_token: <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"content":"안녕하세요!","message_type":"outgoing"}'
```

#### 그 외 외부 연동 키 (.env.example 주요 항목)
```
FB_APP_ID / FB_APP_SECRET / FB_VERIFY_TOKEN   페이스북·인스타
IG_VERIFY_TOKEN                               인스타그램
TWITTER_CONSUMER_KEY / SECRET                 트위터
SLACK_CLIENT_ID / SECRET / SIGNING_SECRET     슬랙
GOOGLE_OAUTH_CLIENT_ID / SECRET               구글 로그인
AWS_ACCESS_KEY_ID / SECRET / AWS_REGION       S3 스토리지
STRIPE_SECRET_KEY / STRIPE_WEBHOOK_SECRET     결제
SMTP_USERNAME / SMTP_PASSWORD                 메일 발송
AZURE_APP_ID / AZURE_APP_SECRET               MS 365 메일
```

---

### Q4. AI 에이전트 구축에 도움이 되는가

**매우 도움이 된다. 이 레포의 최고 가치 중 하나.**

Captain은 튜토리얼 수준이 아닌 **실제 상용 서비스에서 운영되는 AI 에이전트 구현체**다.
구성요소별 구현 위치는 [1.4절 AI/LLM 표](#ai--llm-관련-핵심-자산) 참고.

#### 특히 배울 포인트
1. **툴 설계 제약** — 툴 개수 15개 제한(컨텍스트 폭발 방지), 함수명 64자 제한 대응
2. **SSRF 방어** — 사용자 등록 URL로 AI가 요청할 때 내부망 공격 차단 (`ssrf_filter`)
3. **LLM 관측** — Langfuse로 토큰/비용/지연 추적 (AI 비용 통제의 핵심)
4. **에스컬레이션 설계** — AI가 못 하는 질문을 사람에게 넘기는 분류기
5. **지시 감사(instruction_auditor)** — 환각/잘못된 약속 방지

#### 바로 만들 수 있는 것
```
- 내 서비스용 AI 고객상담 봇 (Captain 활용)
- Chatwoot MCP 서버 (Claude가 상담을 직접 처리)
- Agent Bot API 기반 완전 커스텀 에이전트
- Dashboard App으로 상담원용 AI 패널 삽입
- Captain Custom Tool에 사내 API 등록 (주문조회·배송추적 등)
- Captain 아키텍처를 참고해 다른 도메인 에이전트 구축
```

#### 주의
- Captain은 `enterprise/`에 있어 **프로덕션 상업 이용은 라이선스 필요**
- 단 "개발·테스트 목적 복사·수정"은 라이선스상 허용 → **학습·참고는 문제 없음**
- 상업 이용 시: Chatwoot 구독 구매 또는 **동일 패턴으로 직접 구현**(아이디어 자체는 저작권 대상 아님)

---

### Q5. 수익화 아이디어

→ **4장에서 상세히 다룸.**
핵심: `Platform API`로 계정을 프로그래밍 생성할 수 있다는 점이 **멀티테넌트 SaaS 재판매를 설계상 가능하게** 한다.

---

### Q6. React나 PHP로 만들 수 있는가

**결론: 전체 포팅은 비현실적. 그러나 "붙이는 것"은 매우 쉽다.**

#### 규모 체크
```
Ruby   : 1,380 파일 / 87,700줄
Vue+JS : 2,305 파일 / 291,000줄
합계   : 약 38만 줄 + 마이그레이션 180개 + 테스트 909개
→ 포팅 시 숙련 개발자 5명 × 2~3년 규모 (비추천)
```

#### 전략 A — 백엔드는 Chatwoot 그대로, 프론트만 React/PHP (권장)
```
[React 대시보드] 또는 [PHP 관리자]
        │  REST API (api_access_token)
        ▼
  [Chatwoot Rails 백엔드]  ← 그대로 사용
```
```js
const res = await fetch(`${CW}/api/v1/accounts/${accountId}/conversations`, {
  headers: { api_access_token: token },
});
const { data } = await res.json();
```
- `swagger/`에 OpenAPI 3.1 스펙 전체가 있어 **TypeScript 클라이언트 자동 생성 가능**
- 실시간은 ActionCable WebSocket 구독
- 공수: 2~4주

#### 전략 B — Dashboard App으로 React 앱 삽입 (가장 쉬움)
```
Settings > Dashboard Apps 에 내 React 앱 URL 등록
→ 상담 화면 사이드바에 iframe으로 표시
→ postMessage로 현재 대화/고객 컨텍스트 전달받음
```
- 공수: 3~5일 (가성비 최고)

#### 전략 C — PHP로 Agent Bot
```php
<?php
$payload = json_decode(file_get_contents('php://input'), true);

if ($payload['event'] === 'message_created'
    && $payload['message_type'] === 'incoming') {

    $reply = myAiLogic($payload['content']);

    $ch = curl_init("$CW/api/v1/accounts/{$acc}/conversations/{$conv}/messages");
    curl_setopt_array($ch, [
      CURLOPT_POST => true,
      CURLOPT_HTTPHEADER => [
        "api_access_token: $botToken",
        "Content-Type: application/json",
      ],
      CURLOPT_POSTFIELDS => json_encode([
        'content' => $reply,
        'message_type' => 'outgoing',
      ]),
    ]);
    curl_exec($ch);
}
```
- 공수: 1~2주

#### 클론을 만들고 싶다면 (MVP 범위로 축소)
```
포함: 웹 채팅 위젯 1채널 / 상담원 1:N 대화 / 실시간 / 간단 FAQ + AI 자동응답
제외: 7개 채널 전부, 리포트/SLA/캠페인
→ 1인 3개월 가능

추천 스택
  React : Next.js + Prisma + PostgreSQL + Socket.io + Vercel
  PHP   : Laravel 11 + Filament + Reverb(실시간) + Inertia + React
```

**권고**: 클론보다 **전략 A/B로 Chatwoot 위에 얹는 것**이 압도적으로 효율적.
38만 줄을 무료로 활용하고 차별화 포인트만 직접 개발.

---

### Q7. 유튜브 강의 영상 제작이 가능한가

**가능하며, 소재로서 매우 유리하다.**

#### 유리한 이유
| 이유 | 설명 |
|---|---|
| 한국어 콘텐츠 희박 | Chatwoot 한국어 강의 거의 없음 (블루오션) |
| "비용 절감" 훅 | "월 100만원 Zendesk 무료 대체" → 높은 클릭률 |
| AI 트렌드 직결 | Captain / RAG / 에이전트 = 최상위 인기 키워드 |
| 타겟 명확 | 스타트업 대표, 1인 사업가, 백엔드 개발자, 외주 개발자 |
| 실무급 코드 | Rails/Vue 프로덕션 코드 = 소재 무한 |
| 라이선스 안전 | MIT이므로 코드 시연·해설 자유 |

#### 추천 커리큘럼

**시리즈 1 — 무료로 고객센터 만들기 (입문)**
```
EP1 Zendesk 월 100만원 → 0원? Chatwoot 소개            (5분)
EP2 Docker로 10분 설치하기                              (15분)
EP3 내 홈페이지에 채팅창 달기 (스크립트 한 줄)            (10분)
EP4 카톡·인스타·왓츠앱 연결하기                           (20분)
EP5 상담원 초대 & 자동 배정                               (12분)
EP6 헬프센터(FAQ) 만들기                                 (15분)
EP7 리포트로 성과 분석                                   (12분)
EP8 실서버(VPS) 배포 + 도메인 + SSL                       (25분)
```

**시리즈 2 — AI 상담봇 만들기 (조회수 기대 최상)**
```
EP1 Captain AI 에이전트 구조 해부                        (15분)
EP2 OpenAI 키 연결 + 첫 AI 응답                          (18분)
EP3 RAG 원리 — pgvector로 내 문서 학습                    (25분)
EP4 Custom Tool로 주문조회 API 붙이기(function calling)   (30분)
EP5 Langfuse로 AI 비용·품질 추적                          (20분)
EP6 AI → 사람 에스컬레이션 설계                            (18분)
EP7 Chatwoot MCP 서버 만들어 Claude와 연결                 (35분)
```

**시리즈 3 — 실무 코드 읽기 (개발자 타겟)**
```
EP1 38만 줄 모놀리식 구조 읽는 법                         (20분)
EP2 멀티테넌시 설계 (account_id 전략)                     (25분)
EP3 12개 채널을 하나로 — 어댑터 패턴                       (30분)
EP4 Sidekiq 비동기 아키텍처                               (25분)
EP5 ActionCable 실시간 채팅 구현 분석                      (28분)
EP6 OSS/EE 분리 패턴(prepend_mod_with) = SaaS 수익화 설계  (30분)
EP7 Pundit 권한 설계                                     (20분)
EP8 909개 테스트에서 배우는 RSpec                          (25분)
EP9 180개 마이그레이션으로 배우는 DB 설계                   (30분)
```

**시리즈 4 — React 커스텀 대시보드**
```
EP1 Swagger에서 TypeScript 클라이언트 자동 생성            (20분)
EP2 Next.js로 나만의 상담 대시보드                         (35분)
EP3 ActionCable WebSocket React 연결                     (25분)
EP4 Dashboard App으로 React 앱 삽입                       (20분)
```

#### 법적 주의사항
| 항목 | 가이드 |
|---|---|
| 코드 설명/시연 | MIT이므로 자유. 출처 표기 |
| 스크린샷/화면녹화 | 가능 |
| `enterprise/` 폴더 | 교육 목적 코드 해설은 가능. **라이선스 우회 가이드는 금지** |
| 로고/브랜드 | Chatwoot 로고를 내 강의 브랜드로 사용 금지 |
| 유료 강의 판매 | 가능 |
| 광고 수익 | 가능 |
| 필수 고지 | "Chatwoot은 Chatwoot Inc의 오픈소스 프로젝트이며, enterprise 디렉터리는 별도 상용 라이선스입니다" |

#### 예상 성과 (추정)
```
입문 시리즈 : 3,000 ~ 15,000 뷰/편
AI 시리즈   : 5,000 ~ 30,000 뷰/편
코드 해설   : 1,000 ~  5,000 뷰/편 (좁지만 수익화 전환율 높음)
```

#### 제작 팁
```
1. EP1은 결과 먼저 (완성 화면을 3초 안에)
2. 썸네일에 숫자 대비 ("월 100만원 → 0원")
3. 에러 상황도 그대로 촬영 (신뢰도 상승, 반복 질문 감소)
4. GitHub에 강의용 레포 + .env 템플릿 제공 (구독 유도)
5. 각 편 끝에 다음편 예고
6. 설치편은 Docker로 (환경차 문제 최소)
7. 유튜브 → 유료 강의 → 컨설팅 → 구축 외주 퍼널 설계
```

---

## 4. 수익화 아이디어 상세

### TIER S — 최우선 추천

#### S-1. 화이트라벨 SaaS 재판매

핵심 근거: **Platform API로 계정을 프로그래밍 생성 가능** → 멀티테넌트 재판매가 설계상 가능.

```
고객사 A ─┐
고객사 B ─┼→ 하나의 Chatwoot 인스턴스 (account_id로 테넌트 격리)
고객사 C ─┘

프로비저닝:
  POST /platform/api/v1/accounts
  POST /platform/api/v1/accounts/{id}/account_users
```

| 항목 | 내용 |
|---|---|
| 가격 모델 | 월 29,000원(상담원 3명) / 79,000원(10명) / 199,000원(무제한) |
| 원가 | VPS 월 5~15만원 + 도메인 → 고객 20곳 시 월 매출 150만원, 마진 90%+ |
| 개발 기간 | 2~3개월 (브랜딩 교체 + 가입·결제 + 테넌트 프로비저닝) |
| 타겟 | 스몰 비즈니스, 쇼핑몰, 학원, 병원, 부동산 |
| 필수 조건 | `enterprise/` 삭제 또는 `DISABLE_ENTERPRISE=true` |
| 난이도 | 중 |

차별화 아이디어
- **카카오톡 채널 연동 추가** (Chatwoot에 없음 → 한국 시장 킬러 기능)
- 네이버 톡톡, 당근마켓 채팅 연동
- 한국형 리포트 (주간 보고서 PDF 자동 발송)
- 토스페이먼츠 / 아임포트 결제 연동

#### S-2. 설치·구축 대행 + 유지보수 (가장 빠른 현금화)

```
기본 설치 패키지      : 100 ~   300만원 (1주)     서버 세팅 + SSL + 도메인 + 채널 연동 + 교육
커스터마이징 패키지   : 300 ~ 1,000만원 (3~4주)   브랜딩 + 카톡 연동 + 기존 CRM 연동
AI 상담봇 구축        : 500 ~ 2,000만원 (4~8주)   RAG + 문서 학습 + 튜닝
월 유지보수 (리커링)  :  30 ~   100만원/월        모니터링 + 업데이트 + 장애 대응
```

| 항목 | 내용 |
|---|---|
| 시작까지 | 즉시 가능 |
| 1년차 현실 매출 | 고객 5곳 × 설치 200만 + 월 50만 × 12 ≈ 4,000만원 |
| 영업 채널 | 크몽/위시켓/프리모아, 유튜브 유입, 링크드인, 스타트업 커뮤니티 |
| 난이도 | 낮음 (최우선 추천) |

세일즈 피치
> "Zendesk를 10명이 쓰면 월 90만원, 1년 1,080만원입니다.
> 저희가 300만원에 구축해드리면 4개월에 본전이고, 이후는 서버비 5만원만 내시면 됩니다.
> 게다가 고객 데이터는 100% 고객사 소유입니다."

#### S-3. 한국 시장 특화 애드온 판매

| 애드온 | 가격 | 수요 |
|---|---|---|
| 카카오톡 채널 연동 | 50~200만원/건 또는 월 5만원 | 매우 높음 |
| 네이버 톡톡 연동 | 50~150만원 | 높음 |
| 카페24/고도몰/메이크샵 연동 | 30~100만원 | 높음 |
| 알림톡/친구톡 발송 | 월 과금 | 높음 |
| 토스/아임포트 결제조회 툴 | 30~80만원 | 보통 |
| 한국형 리포트 팩 | 20~50만원 | 보통 |

구현 방식 — 기존 채널 어댑터 패턴을 그대로 따름
```
app/models/channel/kakao.rb              채널 모델 추가
app/controllers/webhooks/kakao_controller.rb  웹훅 수신
app/services/kakao/                      메시지 전송 서비스
```
→ 코어에 **12개 채널 어댑터 예시가 이미 있어** 참고 가능.
→ 커뮤니티에 PR을 보내면 포트폴리오 + 인지도 확보 효과.

### TIER A

#### A-4. 교육 콘텐츠 (유튜브 → 유료 강의 퍼널)
```
[무료 유튜브] → [유료 강의] → [1:1 컨설팅] → [구축 외주]
 구독자 확보     29~99만원     시간당 10~30만원   300~2,000만원
```
| 수익원 | 예상 |
|---|---|
| 유튜브 광고 | 월 10~100만원 (구독 1만 기준) |
| 인프런/클래스101 | 편당 29~99만원 × 수강생 100명 = 3,000~9,000만원 |
| 자체 플랫폼 | 수수료 0% |
| 유료 멤버십(디스코드) | 월 1~3만원 × 200명 = 월 200~600만원 |
| 전자책/노션 템플릿 | 3~5만원 × 500부 |

> 유튜브는 광고 수익보다 **리드 생성 채널**로서의 가치가 크다. 구축 외주 1건이 1년 광고 수익보다 클 수 있음.

#### A-5. AI 고객상담 봇 SaaS
```
구조: Chatwoot(상담 인프라) + 자체 RAG 엔진 + 멀티 LLM
      (Captain 아키텍처를 참고해 직접 구현 → EE 라이선스 회피 + 차별화)

가격: 월 9.9만원(대화 1,000건) / 29.9만원(5,000건) / 맞춤형
원가: LLM 토큰비 (대화당 약 30~100원) → 마진 70~85%
```
차별화: 한국어 특화 임베딩(KoE5, bge-m3), 환각 방지 강화, 답변 신뢰도 점수, 비용 상한 설정
개발 기간 3~6개월 / 난이도 높음 / 포텐셜 매우 높음

#### A-6. Chatwoot MCP 서버 제작 (희소성 최상)

Chatwoot에 MCP 코드가 **0건** → 선점 가능.
```
Claude / Cursor / ChatGPT
        │ MCP 프로토콜
        ▼
  [Chatwoot MCP 서버]  ← 직접 제작
        │ REST API (swagger 스펙 활용)
        ▼
   Chatwoot 인스턴스

활용 예:
  "어제 들어온 미처리 문의 요약해줘"
  "불만 고객 찾아서 사과 메시지 초안 작성해줘"
  "이번 주 CSAT 낮은 대화 분석해줘"
```
| 수익화 | 방법 |
|---|---|
| 오픈소스 + 유료 지원 | GitHub Sponsors, 기업 지원 계약 |
| SaaS 호스팅 버전 | 월 1~3만원 |
| 브랜딩 | "MCP 서버 제작자" 포지셔닝 |

개발 기간 2~4주 (API 스펙 완비로 빠름) / 난이도 중 / 희소성 최상

### TIER B

#### B-7. 산업별 특화 패키지 (버티컬 SaaS)
```
병원용   : 예약 연동 + 진료과 라우팅 + 개인정보 강화
학원용   : 학부모 상담 + 출결 알림 + 성적 문의
쇼핑몰용 : 주문/배송 조회 봇 + 반품 자동화
부동산용 : 매물 문의 + 방문 예약
```
→ 특화 시 가격을 3배 수준으로 책정 가능.

#### B-8. 매니지드 호스팅
```
월 5~20만원 × 50곳 = 월 250~1,000만원
단, 24/7 대응 부담 / Chatwoot Cloud와 가격 경쟁 필요
```

#### B-9. 템플릿·리소스 판매 (패시브 인컴)
```
업종별 FAQ 템플릿 팩            : 3~10만원
커스텀 테마/디자인 팩            : 10~30만원
자동화 규칙 레시피북             : 5만원
원클릭 배포 스크립트(Terraform)  : 10~50만원
```

#### B-10. 기술 컨설팅 / 코드 리뷰
```
시간당 10~30만원 / 아키텍처 감수 건당 100~500만원
→ 유튜브 + 오픈소스 기여로 권위 확보 후 자동 유입
```

### 추천 로드맵

```
Month 1-2   로컬+VPS 설치 숙달
            유튜브 입문 시리즈 EP1~4 업로드 (리드 생성)
            크몽/위시켓에 "Chatwoot 구축" 서비스 등록
            목표: 첫 구축 외주 1건 (200만원)

Month 3-4   구축 외주 2~3건 추가 (현금 확보)
            유튜브 AI 시리즈 시작
            MCP 서버 개발 → 오픈소스 공개 (브랜딩)
            목표: 월 매출 300만원 + 구독자 1,000명

Month 5-8   카카오톡 채널 연동 애드온 개발 (독점 무기)
            월 유지보수 계약 5곳 확보 (리커링 월 250만원)
            유료 강의 출시
            목표: 월 매출 600만원, 리커링 비중 40%

Month 9-12  화이트라벨 SaaS 런칭
            버티컬 1개 선택 집중
            목표: 월 매출 1,000만원+, 리커링 60%
```

### 수익화 비교표

| # | 아이디어 | 난이도 | 투자 | 수익화까지 | 연 매출 포텐셜 | 추천도 |
|---|---|---|---|---|---|---|
| S-2 | 구축 대행 + 유지보수 | 낮음 | 거의 0 | 즉시 | 4,000만~1억 | ★★★ |
| S-3 | 카톡 연동 애드온 | 중 | 1~2개월 | 2개월 | 3,000만~1억 | ★★★ |
| S-1 | 화이트라벨 SaaS | 중 | 2~3개월 | 4개월 | 1억~5억 | ★★★ |
| A-6 | MCP 서버 | 중 | 2~4주 | 2개월 | 간접(브랜딩) | ★★ |
| A-4 | 교육 콘텐츠 | 낮음 | 2주 | 3개월 | 3,000만~1억 | ★★ |
| A-5 | AI 봇 SaaS | 높음 | 3~6개월 | 8개월 | 1억~10억 | ★★ |
| B-7 | 버티컬 특화 | 중 | 2~3개월 | 5개월 | 5,000만~2억 | ★★ |
| B-8 | 매니지드 호스팅 | 낮음 | 1개월 | 3개월 | 3,000만~1억 | ★ |
| B-9 | 템플릿 판매 | 매우 낮음 | 2주 | 1개월 | 500~3,000만 | ★ |
| B-10 | 컨설팅 | 낮음 | 0 | 6개월 | 2,000만~1억 | ★ |

### 결론
```
1순위  S-2 구축 대행   — 즉시 시작, 현금 회수 빠름
2순위  A-4 유튜브      — 리드 생성 엔진
3순위  S-3 카톡 연동   — 한국 시장 독점 무기
최종   S-1 화이트라벨  — 리커링 매출 전환
```

### 수익화 전 법률 체크리스트

```
[ ] enterprise/ 디렉터리 제거 또는 DISABLE_ENTERPRISE=true 확인
[ ] MIT 라이선스 고지문 유지 (LICENSE 파일 포함 + 저작권 표기)
[ ] "Chatwoot" 상표를 제품명으로 사용하지 않음 (자체 브랜드 사용)
[ ] 고객에게 오픈소스 기반임을 투명하게 고지
[ ] 개인정보처리방침 + 처리위탁 계약서 구비 (국내법)
[ ] 재판매 시 책임 범위(SLA, 장애 보상) 계약서 명시
[ ] LLM API 키는 고객사 명의 또는 비용 전가 구조로 설계 (토큰비 리스크 차단)
```

---

## 5. 부록 — 참고 링크 & 체크리스트

### 5.1 GitHub / 공식 링크

| 구분 | 주소 |
|---|---|
| **이 포크 레포** | **https://github.com/bmshin94/chatwoot** |
| 원본(upstream) 레포 | https://github.com/chatwoot/chatwoot |
| 기여자 목록 | https://github.com/chatwoot/chatwoot/graphs/contributors |
| 공식 홈페이지 | https://www.chatwoot.com |
| 공식 문서 / 헬프센터 | https://www.chatwoot.com/help-center |
| 환경변수 문서 | https://www.chatwoot.com/docs/environment-variables |
| 배포 옵션 | https://chatwoot.com/deploy |
| Captain(AI) 문서 | https://chwt.app/captain-docs |
| 번역(Crowdin) | https://translate.chatwoot.com |
| 커뮤니티 Discord | https://discord.gg/cJXdrwS |
| Docker Hub | https://hub.docker.com/r/chatwoot/chatwoot |
| Helm 차트 | https://artifacthub.io/packages/helm/chatwoot/chatwoot |
| 상태 페이지 | https://status.chatwoot.com |
| 엔터프라이즈 개발 가이드 | https://chatwoot.help/hc/handbook/articles/developing-enterprise-edition-features-38 |

### 5.2 레포 내 핵심 파일 바로가기

```
README.md                         제품 개요 및 기능 목록
LICENSE                           MIT + enterprise/ 예외 조항
enterprise/LICENSE                엔터프라이즈 상용 라이선스 원문
CLAUDE.md / AGENTS.md / GEMINI.md AI 에이전트 가이드
.env.example                      환경변수 59개
docker-compose.yaml               로컬 개발 스택 (pgvector/pg16 포함)
Makefile                          setup / db / run / console / debug
Procfile.dev                      overmind 개발 프로세스
swagger/index.yml                 OpenAPI 3.1 + 토큰 3종 정의
lib/chatwoot_app.rb               enterprise?/cloud?/otel_enabled? 플래그
lib/llm_constants.rb              LLM 모델 및 프로바이더 프리픽스
lib/llm/feature_router.rb         기능별 LLM 라우팅
app/models/channel/               12개 채널 어댑터
app/controllers/platform/         Platform API (계정 프로비저닝)
enterprise/app/models/captain/    Captain AI 도메인 모델
enterprise/app/services/captain/  Captain 서비스 + tools
```

### 5.3 빠른 시작 체크리스트

```
[ ] git clone https://github.com/bmshin94/chatwoot.git
[ ] cp .env.example .env
[ ] SECRET_KEY_BASE 생성 (openssl rand -hex 64)
[ ] FRONTEND_URL / REDIS_PASSWORD 설정
[ ] docker compose run --rm rails bundle exec rails db:chatwoot_prepare
[ ] docker compose up
[ ] http://localhost:3000 접속 → 슈퍼관리자 계정 생성
[ ] Settings > Inboxes > Website 수신함 생성
[ ] 발급 스크립트를 내 사이트에 삽입
[ ] (AI 사용 시) Super Admin에서 LLM API 키 설정
[ ] (상업 이용 시) enterprise/ 제거 또는 DISABLE_ENTERPRISE=true
```

### 5.4 결론 요약

```
이것은        : 오픈소스 옴니채널 고객상담 플랫폼 (독립 웹앱)
플러그인/스킬/MCP : 전부 아님 (MCP 코드 0건)
토큰          : 외부 연동 시 필요 (3종) / AI 사용 시 LLM 키 별도 과금
AI 에이전트   : 프로덕션급 레퍼런스로 매우 유용 (Captain)
React/PHP     : 전체 포팅 비현실적, API 연동/Dashboard App으로 붙이는 것이 정답
유튜브        : 제작 가치 높음 (한국어 콘텐츠 희박 + AI 트렌드)
수익화        : 구축 대행 → 유튜브 → 카톡 애드온 → 화이트라벨 SaaS 순서 권장
최대 주의사항 : enterprise/ 디렉터리는 상용 라이선스 (프로덕션 무단 사용 금지)
```
