# 조직역량개발비 관리 시스템 — 바이브코딩 프롬프트

> 이 파일을 Cursor/AI 에이전트에 붙여넣어 단계별 구현을 진행하세요.

## 컨텍스트

- **프로젝트**: Harness Engineering 기반 조직역량개발비 통합 플랫폼
- **스택**: Next.js 15 + NestJS + PostgreSQL + Prisma
- **설계 문서**: `docs/` 폴더 (01~10)
- **DB**: `prisma/schema.prisma`

## Phase 1 — 인프라 (완료 스캐폴드)

```
npm install
cp .env.example .env
docker compose up -d postgres redis
npx prisma migrate dev --name init
npm run db:seed
npm run dev
```

## Phase 2 — 완료 ✅

- budget-requests PATCH/DELETE + 첨부파일 (로컬/S3)
- exports/excel, exports/csv
- notifications + Email Adapter
- Frontend: budget-requests/new, approvals/pending
- OrgScopeGuard + e2e 테스트
- Refresh token rotation + HttpOnly cookie
- shadcn/ui 컴포넌트 마이그레이션

## Phase 2 — 원본 지시 (참고)

```
다음 작업을 순서대로 구현하라:

1. budget-requests PATCH/DELETE 및 첨부파일 multipart 업로드 (S3/MinIO)
2. exports/excel, exports/csv 엔드포인트 (exceljs)
3. notifications 모듈 + 이메일 Adapter 인터페이스
4. Frontend: budget-requests/new 폼 + AI classify debounce 연동
5. Frontend: approvals/pending + ApprovalTimeline API 연동
6. OrgScopeGuard 완성 및 integration 테스트
7. Refresh token rotation + HttpOnly cookie
8. shadcn/ui 설치 및 컴포넌트 마이그레이션

규칙:
- docs/05-rbac.md 권한 매트릭스 준수
- 모든 mutation에 audit_logs 기록
- packages/shared Zod 스키마와 DTO 동기화
- Clean Architecture: application/domain 분리
```

## Phase 3 — AI 고도화

```
1. apps/api/src/modules/ai/prompts/ 에 LLM 프롬프트 추가
2. OpenAI Adapter (AI_PROVIDER=openai)
3. ai_anomaly_snapshots 테이블 + Cron 배치
4. 관리자 대시보드 AI 패널 실시간 API 연동
```

## Phase 4 — 엔터프라이즈 확장

```
1. SSO (OIDC) integrations/sso
2. Slack webhook notifications
3. SAP ERP Adapter stub
4. OCR 영수증 → budget_request 자동 채움
5. K8s manifests (k8s/overlays)
```

## 핵심 API (참고)

- POST `/api/v1/budget-requests`
- POST `/api/v1/approvals/:id`
- GET `/api/v1/dashboard/summary`
- POST `/api/v1/ai/classify-suggest`
- GET `/api/v1/audit-logs`

## 테스트 계정 (seed)

| 역할 | 이메일 | 비밀번호 |
|------|--------|----------|
| 담당자 | user@company.com | Password1! |
| 조직장 | leader@company.com | Password1! |
| 관리자 | admin@company.com | Password1! |

## 코드 품질 체크리스트

- [ ] TypeScript strict, any 금지
- [ ] RBAC Guard 모든 protected route
- [ ] Input validation (DTO + Zod)
- [ ] Audit interceptor mutation 적용
- [ ] Soft delete middleware
- [ ] OpenAPI 데코레이터 완비
