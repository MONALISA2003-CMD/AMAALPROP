# AMAALPROP — Recommended GitHub Repository Blueprint

**Version:** 1.0 · **Prepared:** 13 November 2026 · **Status:** Implementation proposal, not a deployed repository

**Read with:** `phases.md`, `architecture.md`, `tech-stack.md`, the AMAALPROP PRD v1.0 and ERD v1.0.

## 1. Recommendation

Use **one GitHub monorepo** with one authoritative, modular Rust backend. Deploy independently:

| Workload | Directory | Target | Stack |
|---|---|---|---|
| Landlord/staff web and tenant portal | `apps/web/` | Vercel | Next.js, React, TypeScript |
| Authoritative public API | `apps/core-api/` | Render web service | Rust, Axum, Tokio, SQLx |
| Outbox and scheduled processing | `apps/worker/` | Render background worker | Rust, shared core crates |
| Internal AI/document intelligence (later) | `apps/ai-service/` | Render private/internal service | Python, FastAPI |
| Transactional system of record | `database/` | Neon PostgreSQL | Versioned SQL migrations, RLS |
| Private object storage | storage adapter in `crates/integrations/` | Provider TBD; Cloudflare R2 is a candidate | S3-compatible private buckets |
| DNS, security and selected CDN | `infrastructure/cloudflare/` | Cloudflare | DNS, TLS, WAF, rate limiting |
| CI/CD and source of truth | `.github/` | GitHub | Actions, CODEOWNERS, reviews |

**Decision to approve:** `architecture.md` recommends NestJS as one *candidate*, but the dedicated `tech-stack (1).md` recommends **Rust/Axum**. This blueprint adopts the latter as a coherent working baseline, pending an explicit ADR. Do not build both NestJS and Rust business backends.

**Important architecture refinement:** The source stack document suggests domain modules inside `apps/core-api/src/modules/` *and* shared domain crates. That risks duplicated business logic. Here, **all authoritative business modules live once in `crates/core/`**; the API and worker are thin executable adapters that import them. This preserves the modular-monolith requirement.

## 2. Suggested repository tree

```text
amaalprop/
├── .github/
│   ├── workflows/
│   │   ├── ci-web.yml
│   │   ├── ci-rust.yml
│   │   ├── ci-ai.yml
│   │   ├── ci-database.yml
│   │   ├── security.yml
│   │   ├── deploy-staging.yml
│   │   └── release.yml
│   ├── CODEOWNERS
│   ├── dependabot.yml
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
│
├── apps/
│   ├── web/                              # Vercel project root
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── (public)/              # Public informational pages
│   │   │   │   ├── (auth)/                # Sign-in, invitations, recovery
│   │   │   │   ├── (workspace)/           # Landlord and staff workspace
│   │   │   │   └── (tenant)/              # Scoped tenant portal
│   │   │   ├── features/
│   │   │   │   ├── identity-access/
│   │   │   │   ├── organisations/
│   │   │   │   ├── property/
│   │   │   │   ├── tenants/
│   │   │   │   ├── leases/
│   │   │   │   ├── billing/
│   │   │   │   ├── payments/
│   │   │   │   ├── accounting/
│   │   │   │   ├── maintenance/
│   │   │   │   ├── inspections-clearance/
│   │   │   │   ├── approvals/
│   │   │   │   ├── documents/
│   │   │   │   ├── communications/
│   │   │   │   └── reporting/
│   │   │   ├── components/               # Application-wide primitives
│   │   │   ├── lib/                      # API client, auth, formatters
│   │   │   └── styles/
│   │   ├── tests/
│   │   ├── next.config.ts
│   │   ├── package.json
│   │   └── .env.example
│   │
│   ├── core-api/                         # Render HTTP entrypoint, not domain owner
│   │   ├── src/
│   │   │   ├── main.rs
│   │   │   ├── bootstrap.rs
│   │   │   ├── http/
│   │   │   │   ├── routes/                # /api/v1/* route wiring by domain
│   │   │   │   ├── handlers/              # DTO mapping into use cases
│   │   │   │   ├── middleware/            # Authentication, limits, tracing
│   │   │   │   └── errors/
│   │   │   └── openapi.rs                 # Generated/published API contract
│   │   ├── tests/
│   │   ├── Cargo.toml
│   │   └── Dockerfile
│   │
│   ├── worker/                           # Render background worker entrypoint
│   │   ├── src/
│   │   │   ├── main.rs
│   │   │   ├── outbox.rs
│   │   │   ├── schedules/
│   │   │   └── consumers/                 # Idempotent event consumers
│   │   ├── tests/
│   │   ├── Cargo.toml
│   │   └── Dockerfile
│   │
│   └── ai-service/                       # Optional until AI phase
│       ├── app/
│       │   ├── main.py
│       │   ├── api/                      # Authenticated internal endpoints
│       │   ├── security/                 # Verified org/actor context
│       │   ├── document_intelligence/
│       │   ├── retrieval/                # Permission-filtered RAG
│       │   ├── workflows/                # Human review/orchestration
│       │   ├── model_gateway/
│       │   └── schemas/
│       ├── tests/
│       ├── pyproject.toml
│       ├── uv.lock
│       └── Dockerfile
│
├── crates/                               # One root Rust Cargo workspace
│   ├── core/                             # Authoritative modular monolith
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── shared/                   # Shared domain primitives only
│   │       └── modules/
│   │           ├── platform/
│   │           ├── identity_access/
│   │           ├── property/
│   │           ├── tenant/
│   │           ├── lease/
│   │           ├── billing/
│   │           ├── payments/
│   │           ├── accounting/
│   │           ├── maintenance/
│   │           ├── inspections_clearance/
│   │           ├── approvals/
│   │           ├── documents/
│   │           ├── communications/
│   │           ├── reporting/
│   │           └── audit_compliance/
│   ├── persistence/                     # SQLx repository adapters per module
│   │   └── src/
│   │       ├── tenant_context.rs         # Transaction-local RLS context
│   │       ├── transaction.rs
│   │       └── repositories/
│   ├── security/                         # Identity, membership, RBAC + scope
│   ├── events/                           # Event schemas, outbox, idempotency
│   ├── integrations/                     # Storage, mail, SMS, AI API adapters
│   ├── contracts/                        # Shared API/domain transport types
│   └── telemetry/                        # Logging and tracing setup
│
├── packages/                             # pnpm workspace
│   ├── api-contracts/                    # Committed OpenAPI snapshots
│   ├── api-client/                       # Generated TS types/client
│   ├── ui/                               # Shared accessible UI, when useful
│   └── config/                           # Shared lint/TS/test configuration
│
├── database/
│   ├── migrations/                       # ONLY authoritative applied SQL history
│   ├── seeds/                            # Non-sensitive reference/role seed data
│   ├── policies/                         # RLS documentation/examples; apply via migrations
│   ├── tests/
│   │   ├── rls/
│   │   ├── constraints/
│   │   ├── accounting/
│   │   └── concurrency/
│   └── README.md
│
├── infrastructure/
│   ├── docker/
│   │   ├── compose.yaml                 # Local Postgres, object store, mail sink
│   │   ├── compose.test.yaml
│   │   └── postgres-init/
│   ├── render/
│   │   └── render.yaml                  # API, worker, optional AI service
│   ├── vercel/
│   │   └── README.md                    # Root directory and env settings
│   ├── neon/
│   │   └── README.md                    # Branching, roles, pooling, migrations
│   ├── cloudflare/
│   │   ├── README.md
│   │   ├── dns.md
│   │   ├── tls-origin.md
│   │   └── cache-waf-rules.md
│   └── storage/
│       └── README.md                    # Private buckets, retention, signed URLs
│
├── tests/
│   ├── e2e/                             # Playwright: landlord, staff, tenant
│   ├── contract/                        # OpenAPI and client compatibility
│   ├── security/                        # Cross-tenant and privilege tests
│   ├── performance/
│   └── fixtures/                         # Synthetic: no real personal data
│
├── docs/
│   ├── sources/
│   │   ├── architecture.md              # Keep supplied baseline unmodified
│   │   ├── tech-stack.md
│   │   ├── product-requirements.md
│   │   └── erd.md
│   ├── api/
│   ├── adr/                             # Architecture decisions + rationale
│   ├── workflows/                       # State machines and transition tables
│   ├── data-model/                      # ERD -> reviewed physical schema
│   ├── security/                        # RBAC matrix, RLS threat model
│   ├── accounting/                      # Posting matrices, reconciliation policy
│   ├── operations/                      # Deployment, backup/restore, incidents
│   └── testing/                         # Acceptance + negative case catalogue
│
├── scripts/
│   ├── dev.sh
│   ├── test.sh
│   ├── db-migrate.sh
│   ├── db-reset-local.sh
│   ├── openapi-export.sh
│   └── generate-api-client.sh
│
├── .editorconfig
├── .gitignore
├── .dockerignore
├── .env.example                       # Non-secret placeholders only
├── .pre-commit-config.yaml            # Optional
├── Cargo.toml                         # Rust workspace members
├── Cargo.lock
├── rust-toolchain.toml
├── package.json                       # Workspace scripts
├── pnpm-workspace.yaml
├── pnpm-lock.yaml
├── compose.yaml                       # Convenience wrapper/entrypoint
├── phases.md                          # Implementation roadmap, gates, checklist
├── README.md
└── LICENSE                           # Choose explicitly; omit until selected
```

### Inside each Rust business module

Use an internal layout like the following; **do not duplicate these layers inside the API executable**:

```text
crates/core/src/modules/payments/
├── mod.rs                  # Public module facade
├── domain.rs               # Invariants and state transitions
├── commands.rs             # Application use-case inputs
├── service.rs              # Authorised payment workflow orchestration
├── ports.rs                # Repository/provider interfaces
├── events.rs               # Versioned business events
└── tests.rs                # Unit and property-based tests

crates/persistence/src/repositories/payments.rs  # SQLx implementation
apps/core-api/src/http/routes/payments.rs       # HTTP adapter only
apps/worker/src/consumers/payment_events.rs     # Async adapter only
```

Modules communicate through **public application interfaces** and events; they must never write to other modules' tables behind that module's business rules. Accounting is the **only** owner of journal posting. `crates/core` is a Rust library inside the modular monolith, **not** a separately deployed microservice.

## 3. Design and security rules from the PRD/ERD

1. **Identity:** global `user_accounts.normalised_email` is unique; account, membership, tenant party, tenant person, lease participant and occupant are different concepts. Preserve **one active organisation membership per account** until formally changed. Require a controlled transfer workflow.
2. **Owner:** one designated active landlord-owner account per organisation; staff authority is not ownership.
3. **Scope:** role/permission, property/portfolio/building assignments, reporting relationship, and approval authority are separate checks.
4. **Multi-tenancy:** derive organisation from verified membership; every protected SQL transaction sets organisation context **transaction-locally**; deny on missing context; enforce RLS and composite organisation FKs; runtime database role must not bypass RLS. Repeat on worker, exports and AI retrieval.
5. **Property:** optional portfolio/building/level. Standalone units must work without fake buildings/floors. Keep custom labels unchanged.
6. **Leasing:** one lease supports multiple units and tenant parties, with separate actual occupants; prevent overlapping/over-limit leases under concurrent activation.
7. **Payments:** tenants pay externally; screenshots and references are submissions, **never receipt confirmation**. Staff verifies independently, then posts/allocates and issues receipts. Direct bank/mobile-money API is *not* a prerequisite.
8. **Accounting:** use `NUMERIC`/`rust_decimal`, fixed currency rules, transactional balanced journals, immutable posted records, reversals, period locks, deposit liabilities, idempotency and reconciliation.
9. **Files:** private object store, authorised short-lived downloads, metadata and access audit. Never store uploads on ephemeral Render disks.
10. **AI:** private authenticated core→AI calls, human review for extraction and consequential decisions; no free-form model-generated production SQL, no autonomous journal posting or payment approval.
11. **Architecture:** REST `/api/v1`, OpenAPI-derived clients, transactional outbox and worker, separate staging/prod, Docker Compose locally, GitHub Actions CI, versioned SQL migrations.
12. **Caching:** never edge-cache tenant/financial/personal data. Cloudflare is not a substitute for backend and database security.

## 4. Unresolved model/architecture decisions to resolve in Phase 0

The following are **gaps/contradictions in the supplied documents**, not decisions already made. Record an ADR and/or product decision before affected physical migrations:

| ID | Topic | Required resolution |
|---|---|---|
| D-01 | Core implementation language | Architecture baseline proposes NestJS; specialist stack proposes Rust/Axum. Approve one. This blueprint assumes Rust. |
| D-02 | Email rule | Enforce global uniqueness and one organisation membership; decide whether tenant portal identities are also subject to the restriction; define transfer and email-change workflows. |
| D-03 | Physical property hierarchy | PRD says levels belong to buildings; ERD permits nullable `building_id` for levels. Decide and implement FK/unique rules. |
| D-04 | Missing logical tables | The PRD/architecture require records beyond the ERD's named tables: occupancy history/occupants, lease amendments, billing schedules and policy, approvals, clearance cases, refunds, receipts, suppliers/AP, bank reconciliation, outbox, idempotency and notification attempts. Write a schema reconciliation matrix. |
| D-05 | Payment instruction override | PRD lists organisation→portfolio→property→building→lease; architecture also mentions unit scope. Approve precedence and whether unit override exists. |
| D-06 | Financial policy | Confirm accounting basis, taxes/withholding, joint tenant liability, cash receipts, multi-currency FX treatment, refund authority and period reopening. |
| D-07 | Approval policy | Define separation-of-duties thresholds, delegation, and tenant exit final approval. |
| D-08 | Authentication and files | Choose managed identity vs self-managed sessions, storage provider/region, document retention, malware scanning and recovery. |
| D-09 | Uganda/privacy/legal | Approve data regions, cross-border processing, retention, subprocessors, backups and local compliance advice before production customer data. |
| D-10 | Service configuration | Confirm Render/Neon proximity and pooling/RLS behaviour; Cloudflare proxy/origin compatibility; staging restrictions; RTO/RPO. |
| D-11 | Product subscription billing | Keep AMAALPROP SaaS subscriptions separate from landlord property accounting; decide whether to defer entitlements/billing. |

## 5. First commit vs future directories

**Create immediately:** `.github/`, `apps/web/`, `apps/core-api/`, `apps/worker/`, `crates/`, `database/`, `infrastructure/docker/`, `infrastructure/{render,vercel,neon,cloudflare}/`, `docs/sources/`, `docs/adr/`, `tests/`, root workspace files and `phases.md`.

**Scaffold only (do not spend weeks building it):** `apps/ai-service/`, all domain-module folders with TODO interfaces, and the provider adapters. Start functionality per the phases instead of adding speculative services or empty complexity.

**Do not create yet:** C++ runtime, microservices by domain, Kafka, Kubernetes, Redis cluster, a second authoritative payments backend, native mobile app or a separate vector database. Add only when measured demand and approved decisions require them.

## 6. Workspace/build conventions

- One root Rust workspace includes `apps/core-api`, `apps/worker` and all `crates/*`; one checked-in `Cargo.lock`. `cargo fmt`, `clippy`, `test`, migration integration tests and dependency checks run from the root.
- One `pnpm` workspace for `apps/web` and `packages/*`; checked-in `pnpm-lock.yaml` and pinned Node/PNPM version. Do not couple Rust compilation to frontend package scripts.
- `apps/ai-service/` is independently managed with `uv.lock`, Python tests and its own Dockerfile. Phase 0 can include a skeleton, but no AI dependency in financial operations.
- **One migration source:** `database/migrations/`. `db-migrate.sh` invokes the designated SQLx migration runner with this explicit path. All policy changes deployed through migrations; manual policies in `database/policies/` are explanatory/tests only.
- OpenAPI is generated from authoritative Rust endpoints or maintained with a single declared source of truth, committed to `packages/api-contracts/`, and checked for drift. Generate TypeScript API client from that contract in CI.
- Build Render Rust images with **repository root Docker context**, allowing shared `crates/` dependencies. The Dockerfiles live under `apps/core-api/` and `apps/worker/`, but Docker build context must include root `Cargo.toml` and `crates/`.
- Vercel project root: `apps/web/`; configure monorepo build/install compatibility. Render runs API, worker and optional AI as separate services. Keep sensitive env vars per environment, never in source or client-side `NEXT_PUBLIC_*` values.
- Use isolated local Postgres and local object/email fixtures via Docker Compose; real managed Neon/Render/Vercel credentials belong only to approved development/staging/production environments.

## 7. Suggested GitHub operating model

- Protect `main`; require PR, ownership review and CI checks.
- Workflow checks: Rust format/clippy/tests, TypeScript lint/typecheck/unit/UI/E2E, Python lint/type/pytest, OpenAPI drift, SQL migrations/constraints, RLS cross-tenant negative tests, accounting concurrency/idempotency, dependency/secret/container scans.
- Promote **staging before production** with explicit migration review. Migration must not run automatically at every container start.
- Tag releases; record deployment commit and database migration version; backup and restore tests precede production financial use.
- Commit ADRs when a decision changes. Keep baseline source documents unchanged under `docs/sources/`; append reconciliation notes in separate files.

## 8. Local setup when creating the repository

1. Create a **private** GitHub repository `amaalprop` and enable branch protection and app installation/repository permissions.
2. Copy the four supplied files unmodified into `docs/sources/` (rename only the copied *filenames*, not the contents).
3. Create root Cargo and pnpm workspaces, initial Next.js app, Axum API/worker shells, local compose setup, and health endpoints.
4. Add a restricted local DB role, migration runner, the first organisation RLS tests, and synthetic test fixtures.
5. Connect Vercel to `apps/web` and Render to root-context Rust Docker builds; allocate separate Neon staging and production environments. Do not attach production secrets before environment/region/legal checks.
6. Add Cloudflare DNS/TLS/WAF only after provider-domain compatibility is tested. Choose private object storage, email and SMS providers with appropriate privacy and recovery terms.
7. Follow the order and mandatory exit gates in **`phases.md`**.

> The connectors may authenticate to the providers without automatically having installation access to a new repository or permissions to create production resources. Verify repository-level installation, resource-level roles and deployment authorisations during Phase 0. No resources were created as part of this blueprint.
