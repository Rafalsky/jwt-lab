# jwt-lab

Proof of Concept repo for understanding token-based authentication and authorization in practice:
JWT (JWS), sessions, opaque tokens, refresh tokens, and token introspection — including browser threats (XSS/CSRF) and delivery strategies (HttpOnly cookies vs Authorization header).

## Goals

This repository is a hands-on lab to:
- Understand what JWT is (header/payload/signature) and what it verifies (authenticity + integrity, not confidentiality).
- Implement offline verification (JWT) vs online verification (opaque + introspection).
- Compare session-based auth vs token-based auth.
- Demonstrate access vs refresh token patterns and why refresh tokens are a control point.
- Show real browser threat models: XSS vs CSRF, and how storage/transport choices change risk.
- Keep the code small, explicit, and educational.

## Non-goals

- Production-ready IAM system.
- Full OAuth2/OIDC implementation.
- UI polish.

## Architecture

Services (Docker):
- **issuer**: authentication server that issues tokens (JWT access tokens, opaque refresh tokens) and exposes an introspection endpoint.
- **api**: resource server that protects endpoints using:
  - JWT verification (offline)
  - optional introspection (online truth)
- **web**: minimal web client to demonstrate:
  - HttpOnly cookie flow (server-set tokens)
  - Authorization header flow (bearer token) for comparison
- **redis**: token store for opaque tokens / refresh sessions (enables revocation)

## Key concepts demonstrated

### JWT (JWS)
- Header and payload are Base64URL-encoded (readable, not encrypted).
- Signature binds header + payload and allows verification by trusted servers.
- JWT enables stateless verification but cannot be revoked instantly (only `exp` limits lifetime).

### Access vs Refresh
- **Access token**: short-lived JWT, used on API requests.
- **Refresh token**: opaque, long-lived, stored server-side (revocable), used only to mint new access tokens.

### Opaque tokens
- Random identifiers with no client-readable meaning.
- Require online lookup/introspection to map to identity/scope.
- Enable immediate revocation.

### Token introspection
- API asks issuer: “is this token active right now and what does it mean?”
- Enables online truth at a performance cost.

### Cookies vs Authorization header
- HttpOnly cookies reduce token theft via XSS but require CSRF protection.
- Bearer tokens in JS storage increase risk under XSS; sending Authorization headers reduces classical CSRF.

## Threat model (what we show)

- **XSS** demo: why localStorage is dangerous for bearer tokens (portable credential theft).
- **CSRF** demo: why cookies require SameSite/CSRF tokens/Origin checks for state-changing requests.
- Logging pitfalls: why tokens must not be logged.

## How to run

```bash
docker compose -f infra/docker-compose.yml up --build
```

Then open:
- Web: http://localhost:3000
- Issuer: http://localhost:4000
- API: http://localhost:5000
- Redis: localhost:6379

## Planned endpoints (high level)

### Issuer:
- POST /login -> sets refresh token cookie + returns access token (depending on flow)
- POST /refresh -> rotates refresh token, returns new access token
- POST /revoke -> revokes refresh token (logout / admin revoke)
- POST /introspect -> returns { active, sub, scope, exp }

### API:

- GET /public -> no auth
- GET /me -> auth required
- POST /admin/action -> scope required (admin:write)
- POST /transfer -> CSRF protected cookie-based mutation endpoint
- (optional) GET /edge-check -> demonstrates pure offline JWT verification behavior

### Web:

- Buttons/flows to:
- login via cookie flow
- login via bearer flow
- call endpoints
- trigger revoke
- trigger introspection vs offline verify comparisons

## Project structure (planned)

- /services/issuer
- /services/api
- /services/web
- /infra/docker-compose.yml
- /docs/notes.md (learning notes and diagrams)

## Implementation notes

- JWT: RS256 (issuer signs with private key, API verifies with public key).
- Keys served via a local JWKS endpoint (cached by API).
- Refresh token is opaque and stored in Redis with metadata (userId, scopes, expiry, revoked flag).
- Access token lifetime short (e.g. 1–5 minutes) to demonstrate revocation latency.
- CSRF defense: SameSite + CSRF token (double submit) and/or Origin checking.