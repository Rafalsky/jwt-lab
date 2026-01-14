# TODO

## Milestone 0 — repo i tooling
- [ ] Create repo + basic directories (`services/`, `infra/`, `docs/`)
- [ ] Docker Compose: issuer + api + web + redis (healthchecks)
- [ ] Common config TypeScript (tsconfig), ESLint, prettier

## Milestone 1 — JWT fundamentals (offline verification)
- [ ] Issuer generates kay pairs (dev) i signs JWT (RS256)
- [ ] API verified JWT public key (offline)
- [ ] Endpoints `GET /public`, `GET /me` (requires JWT)
- [ ] Claims: `sub`, `iss`, `aud`, `iat`, `exp`, `scope`
- [ ] Walidations: `iss`, `aud`, clock skew tolerance

## Milestone 2 — Access vs refresh
- [ ] Issuer: `POST /login` returns:
  - [ ] access token (JWT, short)
  - [ ] refresh token (opaque, long) stored on Redis
- [ ] Issuer: `POST /refresh` rotates refresh token i returns new access token
- [ ] API: endpoints required scope (`admin:write`, `orders:read`)

## Milestone 3 — Opaque + introspection
- [ ] Issuer: `POST /introspect` (RFC 7662-like)
- [ ] API: switcher based by ENV:
  - [ ] `AUTH_MODE=jwt` (offline)
  - [ ] `AUTH_MODE=introspect` (online truth)
- [ ] Cache introspection (np. 30s) i deauthorization logic

## Milestone 4 — Revocation i „brak natychmiastowości” JWT
- [ ] `POST /revoke` revokes refresh token (server-side)
- [ ] Demo: after revoke:
  - [ ] access token works till `exp`
  - [ ] refresh is not working anymore
- [ ] Endpoint admin revoke (in issuer) + test scenario

## Milestone 5 — Cookie vs Bearer + XSS/CSRF demos
- [ ] Web flow A: HttpOnly cookie (refresh inside cookie)
- [ ] Web flow B: Authorization header (access inside memory; describe why not localStorage)
- [ ] CSRF demo endpoint (`POST /transfer`):
  - [ ] variation without security (demonstrate susceptibility)
  - [ ] variation with security (SameSite/CSRF token/Origin check)
- [ ] XSS demo (secure, local):
  - [ ] a page with a controlled XSS sink showing the theft of a localStorage token (only in the lab)
  - [ ] comparation: HttpOnly cookie can't be read

## Milestone 6 — Tests + documentation
- [ ] Integration tests (np. Vitest + supertest) for:
  - [ ] JWT verify
  - [ ] refresh rotation
  - [ ] introspection
  - [ ] revocation flow
- [ ] `/docs/`:
  - [ ] sequence diagrams (login/refresh/introspect)
  - [ ] checklists: when JWT vs session vs opaque