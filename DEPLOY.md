# GitHub 배포 가이드

## 1. 저장소 준비

```bash
git init
git add .
git commit -m "chore: initial deploy-ready setup"
git branch -M main
git remote add origin https://github.com/YOUR_USER/YOUR_REPO.git
git push -u origin main
```

로컬에서 lock 파일 생성 후 커밋 (CI `npm ci`용):

```bash
npm install
git add package-lock.json
git commit -m "chore: add package-lock.json"
```

## 2. GitHub Actions (자동)

| Workflow | 트리거 | 역할 |
|----------|--------|------|
| `ci.yml` | PR / push | 빌드·린트·Docker 검증 |
| `deploy.yml` | `main` push, `v*` 태그 | GHCR 이미지 푸시 |

### GHCR 이미지

- `ghcr.io/<owner>/<repo>/ocd-api:latest`
- `ghcr.io/<owner>/<repo>/ocd-web:latest`

패키지 공개: GitHub → Packages → Package settings → Change visibility

### Repository Variables (권장)

Settings → Secrets and variables → Actions → Variables

| 이름 | 예시 |
|------|------|
| `NEXT_PUBLIC_API_URL` | `https://api.your-domain.com/api/v1` |

### Secrets (서버 배포 시)

| Secret | 설명 |
|--------|------|
| `JWT_ACCESS_SECRET` | 32자 이상 랜덤 |
| `JWT_REFRESH_SECRET` | 32자 이상 랜덤 |
| `POSTGRES_PASSWORD` | DB 비밀번호 |

## 3. VPS / 클라우드 Docker 배포

서버에 `.env.production` 생성:

```bash
cp .env.production.example .env.production
# 값 수정 후
```

`docker-compose.prod.yml` 이미지 경로 수정:

```yaml
API_IMAGE=ghcr.io/YOUR_USER/YOUR_REPO/ocd-api:latest
WEB_IMAGE=ghcr.io/YOUR_USER/YOUR_REPO/ocd-web:latest
```

배포:

```bash
docker login ghcr.io -u YOUR_GITHUB_USER
docker compose -f docker-compose.prod.yml --env-file .env.production pull
docker compose -f docker-compose.prod.yml --env-file .env.production up -d
```

최초 시드 (선택):

```bash
docker compose -f docker-compose.prod.yml exec api npx tsx prisma/seed.ts
```

## 4. CORS / URL 체크리스트

- [ ] `CORS_ORIGIN` = Web 공개 URL (예: `https://app.example.com`)
- [ ] `NEXT_PUBLIC_API_URL` = API 공개 URL + `/api/v1` (Web **빌드 시** 주입)
- [ ] `API_PUBLIC_URL` = API 공개 URL (첨부파일 링크용)
- [ ] HTTPS 리버스 프록시 (nginx/Caddy) 권장

## 5. nginx 리버스 프록시 예시

```nginx
server {
  listen 443 ssl http2;
  server_name app.example.com;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}

server {
  listen 443 ssl http2;
  server_name api.example.com;

  location / {
    proxy_pass http://127.0.0.1:4000;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

## 6. 태그 릴리스

```bash
git tag v1.0.0
git push origin v1.0.0
```

→ `deploy.yml`이 semver 태그로 이미지 추가 publish

## 7. 트러블슈팅

| 증상 | 해결 |
|------|------|
| Web API 연결 실패 | `NEXT_PUBLIC_API_URL` 빌드 args 재확인 후 Web 이미지 재빌드 |
| DB 연결 실패 | `DATABASE_URL` 호스트 `postgres` (compose 내부) |
| 마이그레이션 실패 | API 로그 확인, `prisma migrate deploy`는 entrypoint에서 자동 실행 |
| 401 로그인 | `db:seed` 실행 또는 비밀번호 확인 |
