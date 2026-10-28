# Deployment, operations and security checks

## Scope and baseline

Guidance/checklists for backend `2f34fef` on 2026-10-28; this does not claim deployment, backup,
restore or acceptance was performed. Detailed local commands and configuration are in `backend/README.md`,
`backend/docker-compose.yml` and `backend/e2e/README.md`. See [backend test results](../04-testing/BACKEND_VERIFICATION.md).

## Local setup and configuration

1. Prepare Java 21, Maven and Docker Compose; copy `backend/.env.example` to `.env` as needed.
   Never commit `.env`, JWT secrets, passwords, reset codes, VNPAY or AI keys.
2. From `backend`, run `docker compose up -d --build`; inspect `docker compose ps` and
   `docker compose logs backend`, then check `GET http://localhost:8080/actuator/health`.
3. Compose overrides `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` and `REDIS_URL` for the backend container:
   it uses Compose Postgres/Redis, **not a managed DB from `.env`**. Host `mvn spring-boot:run` uses
   host configuration; Compose Redis is not published to the host.
4. `docker compose down` retains the volume; `docker compose down -v` **deletes data**, so use it
   only with authorization and understanding of its scope.

## Release / Render checklist

- [ ] Record artifact commit, environment, operator and rollback plan; check the matching tests/CI.
- [ ] Confirm the database, backup and restore permissions; do not assume backup/restore succeeded.
- [ ] Configure `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`, `APP_JWT_SECRET` via a secret manager; set
  CORS to trusted origins and configure SMTP, VNPAY, AI and Redis as needed. Do not use local credentials in production.
- [ ] Review new Flyway migrations and compatibility with the old application; do not edit applied
  migrations. Migrations run at backend startup: inspect logs/health before serving traffic.
- [ ] After release, check health, public catalog GET, an authorized test-account login and a read-only
  authenticated GET. Do not put tokens or sensitive data in evidence.
- [ ] Do not run the user/order-creating E2E collection on Render. Stop release if smoke checks fail.

Rollback: confirm the old application is compatible with the current schema before restoring the old
artifact. Do not automatically reverse migrations or restore a production database; data impact,
approval and a recovery plan are required. Record the incident, timing and outcome; test backup/restore
in a safe environment before treating it as available.

## Security and incident checklist

| Check | Existing evidence / outstanding work |
|---|---|
| Invalid/expired tokens, refresh rotation/logout | `AuthIntegrationTest`, `AuthTokenEdgeCaseTest`; H2 tests do not replace deployed-environment checks. |
| Role and ownership for orders/sellers/AI | `OrderApiTest`, `SellerProductApiTest`, `ShipmentApiTest`, `AiApiTest`; identify and record the exact test for each requirement before acceptance. |
| VNPAY signatures and repeat callback | `VnpayGatewayTest`, `PaymentApiTest`; no real sandbox transaction was verified in this run. |
| CORS, secrets, logs | `CorsConfigTest` tests configuration; review real origins, secrets and logs before release. |
| Redis / AI provider failure | `CartCacheApiTest`, `AiApiTest` contain fallback tests; monitor logs and verify the actual environment. Redis is excluded from health checks. |

See [security flow](../features/auth/SECURITY_FLOW.md). On provider/DB failure, inspect health, sanitized
logs and configuration; never disable authentication as incident remediation. Payment reconciliation
or refunds require the approved process, not ad hoc data edits.
