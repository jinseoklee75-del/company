# 조직역량개발비 관리 시스템 (OCD Budget Platform)

기업용 조직역량개발비 통합 운영 플랫폼 — **Harness Engineering** 기반, 보안 중심, AI 협업형 엔터프라이즈 SaaS.

## 핵심 목적

| 영역 | 설명 |
|------|------|
| 사용내역 | 조직별 역량개발비 등록·수정·첨부 |
| 결재 | 작성 → 조직장 승인 → 관리자 집계 |
| 모니터링 | 실시간 예산·승인 대기·소진율 |
| 분석 | 조직/분류별 대시보드, AI 이상탐지·예측 |
| 보안 | RBAC, Audit Trail, 암호화, MFA 확장 |

## 기술 스택

- **Frontend**: Next.js 15, React, TypeScript, TailwindCSS, shadcn/ui, Recharts
- **Backend**: NestJS, PostgreSQL, Prisma ORM
- **인증**: JWT + Refresh Token, bcrypt, RBAC
- **인프라**: Docker, Kubernetes-ready, GitHub Actions CI/CD

## 프로젝트 구조

```
조직역량개발비/
├── apps/
│   ├── web/          # Next.js (App Router)
│   └── api/          # NestJS REST API
├── packages/
│   └── shared/       # 공유 타입, 상수, Zod 스키마
├── prisma/           # DB 스키마 (단일 소스)
├── docs/             # 설계 산출물
├── docker/           # Dockerfile, nginx
└── .github/workflows/
```

## 빠른 시작 (로컬 개발)

```bash
npm install
cp .env.example .env
npm run db:migrate
npm run db:seed
npm run dev
```

## GitHub 배포

상세: **[DEPLOY.md](DEPLOY.md)**

1. `main` 브랜치 push → **GHCR**에 `ocd-api`, `ocd-web` 이미지 자동 빌드
2. 서버에서 `docker-compose.prod.yml` + `.env.production` 으로 실행

```bash
cp .env.production.example .env.production
# GHCR 이미지 경로·도메인·JWT 시크릿 설정
npm run docker:prod
```

### Phase 2 기능 (적용 완료)

- 사용내역 PATCH/DELETE, 첨부파일 업로드 (로컬 `uploads/` 또는 S3/MinIO)
- Excel/CSV보내기 (`GET /exports/excel`, `/exports/csv`)
- 알림 모듈 + 이메일 Adapter (콘솔 로그, SMTP 확장 가능)
- 사용내역 등록 UI + AI 분류 debounce
- 조직장 승인 대기 UI + 결재 타임라인 API
- OrgScopeGuard + e2e 테스트 (`npm run test:e2e -w @ocd/api`)
- JWT Refresh Token rotation (HttpOnly Cookie)
- shadcn/ui 스타일 컴포넌트 (Button, Input, Card, Badge 등)

- API: http://localhost:4000
- API Docs (Swagger): http://localhost:4000/api/docs
- Web: http://localhost:3000

## 설계 문서

| 문서 | 경로 |
|------|------|
| 시스템 아키텍처 | [docs/01-architecture.md](docs/01-architecture.md) |
| ERD | [docs/02-erd.md](docs/02-erd.md) |
| API 명세 | [docs/03-api-spec.md](docs/03-api-spec.md) |
| 화면 설계 | [docs/04-screen-design.md](docs/04-screen-design.md) |
| 권한 구조 | [docs/05-rbac.md](docs/05-rbac.md) |
| 보안 설계 | [docs/06-security.md](docs/06-security.md) |
| AI 분석 구조 | [docs/07-ai-structure.md](docs/07-ai-structure.md) |
| 테스트 전략 | [docs/08-test-strategy.md](docs/08-test-strategy.md) |
| 배포 전략 | [docs/09-deployment.md](docs/09-deployment.md) |

## 사용자 역할

1. **USER** — 조직 담당자 (등록, 수정요청, 본인 조직 조회)
2. **ORG_LEADER** — 조직장 (승인/반려, 예산·추이 조회)
3. **BUDGET_ADMIN** — 역량개발비 관리자 (통합 조회, 대시보드, Excel, AI 분석)
4. **SYSTEM_ADMIN** — 시스템 관리자 (사용자·조직·권한·감사로그)

## Harness Engineering 원칙

- 모듈 단위 분리, 이벤트 중심 설계
- OpenAPI 표준화 → AI/프롬프트 확장 친화
- Audit Trail + Observability 전 구간
- 확장 포인트: SSO, ERP, Slack/Teams, Copilot, OCR

## 라이선스

Proprietary — 내부 사용
