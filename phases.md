# AMAALPROP — Implementation Phases and Acceptance Gates

**Plan version:** 1.0  
**Prepared:** 13 November 2026  
**Status:** Proposed executable engineering roadmap  
**Sources:** AMAALPROP Product Requirements Document v1.0, AMAALPROP ERD v1.0, `architecture.md` v1.0, and `tech-stack.md` v1.0  
**Companion:** `repo-structure.md`  
**Delivery model:** Web first, modular monolith, incremental vertical slices, no live financial posting before financial controls pass.

## A. Read this first

This plan describes **work packages and exit gates, not calendar promises**. Size/sequence can be refined after team capacity, architecture decisions and initial estimates are known. Each phase has explicit prerequisites, deliverables and tests. Track each checkbox in GitHub issues/PRs and link it to the implemented commit.

### Recommended stack (working baseline, not an already approved ADR)

- **Web:** Next.js + TypeScript, Vercel, Tailwind/shadcn/Radix, TanStack Query/Table, Zod, Vitest and Playwright.
- **Authoritative backend:** Rust + Axum + Tokio + SQLx in one modular business core, API deployed to Render.
- **Worker:** Rust process on Render using the same business core and a PostgreSQL transactional outbox.
- **AI:** Separate Python + FastAPI internal service on Render, **optional and not on the critical path** for the operational/financial MVP.
- **Database:** Neon PostgreSQL + versioned migrations, composite organisation FKs and PostgreSQL RLS.
- **Private files:** S3-compatible storage (provider/region pending, Cloudflare R2 possible).
- **Infrastructure:** GitHub, Docker/Compose, Cloudflare DNS/TLS/WAF, provider-native observability plus tracing/error tracking.
- **API:** Authenticated and versioned REST `/api/v1`, checked-in OpenAPI contract and generated TypeScript client.

The general architecture document proposed NestJS while the dedicated technology-stack document proposed Rust. Record an ADR approving one implementation; **this plan assumes Rust**. Do not start two competing authoritative backend implementations.

### Always-on engineering rules (apply in every phase)

- [ ] Every business record, write path, file, report, job and AI retrieval is tested for **organisation isolation and scope permissions**.
- [ ] Database transactions set the authorised organisation context with `SET LOCAL` or equivalent, deny on missing context, use a restricted runtime role and enforce RLS plus organisation-safe foreign keys.
- [ ] No browser-supplied `organisation_id` is trusted; global account != organisation membership != real-estate tenant.
- [ ] Implement and test permission, scope, workflow-state and separation-of-duty checks at the backend, not merely by hiding UI controls.
- [ ] Protect sensitive person, account, document and finance data; prevent public caching of private responses and use private file storage.
- [ ] All significant transitions are versioned, attributable and audited; events/outbox and side effects use stable IDs and idempotency.
- [ ] Use `NUMERIC`/decimal rather than floating point for money. Preserve posted journals; use reversals, not edits/deletes.
- [ ] Changes have tests (positive and negative), reviewed migrations, OpenAPI updates and an observable error path.
- [ ] Prefer small vertical slices and keep a usable web interface alongside API/database implementation.
- [ ] Use synthetic fixture data; never copy customer data into local development or CI.

### Phase map

| Phase | Focus | Depends on | Delivery status at exit |
|---|---|---|---|
| 0 | Decisions, monorepo, environments, CI | Source docs | Reproducible empty system + approved critical ADRs |
| 1 | Trust boundary, accounts, organisation, access | 0 | Secure organisation onboarding and isolation |
| 2 | Portfolio/property/building/level/unit | 1 | Fully usable property inventory and scopes |
| 3 | Tenants, portal, leases, approvals | 1–2 | Multiple units/parties and safe lease activation |
| 4 | Accounting engine | 1–3 | Balanced immutable ledger, posting/reversal controls |
| 5 | Billing, verified payments, receipts, reconciliation | 3–4 | Complete controlled rent-to-ledger flow |
| 6 | Maintenance, expenses, inspection, clearance | 3–5 | Auditable repairs and tenant-exit settlement |
| 7 | Communications, dashboards, reporting | 1–6 | Complete operational MVP and reliable reporting |
| 8 (optional) | AI and document intelligence | 1, 3, 7 | Secure assistive AI with human review |
| 9 | Production readiness and controlled release | 0–7; 8 is optional | Approved live release, restoration and incident readiness |
| 10 (later) | Expansion and optimisation | Validated production needs | Advanced integrations/mobile/scale |

**MVP launch scope:** Phases **0–7 and 9**. Do **not** delay launch for experimental AI; do **not** shorten accounting or tenant-isolation gates. A demonstration or pilot may run earlier using synthetic/non-production records and with financial posting disabled.

---

## Phase 0 — Product/architecture reconciliation + engineering foundation

**Goal:** Resolve the architectural and schema choices that would otherwise cause expensive rework, and establish a reproducible repository and delivery pipeline.

**Prerequisite:** The four provided source documents are version-controlled without silent rewriting.

### Product and design decisions

- [ ] Create `docs/sources/` and copy the supplied PRD, ERD, system architecture and specialist tech-stack document unchanged.
- [ ] Write `ADR-001` modular monolith; `ADR-002` Rust vs NestJS backend; `ADR-003` web-first and OpenAPI; `ADR-004` Neon/RLS; `ADR-005` outbox/worker; `ADR-006` AI separation; `ADR-007` Cloudflare/Render/Vercel topology.
- [ ] Confirm global normalised email uniqueness, **one active organisation membership per login**, tenant portal treatment, employee/tenant transfers, owner transfer and account recovery.
- [ ] Decide levels-without-building semantics and payment-instruction override precedence (PRD and architecture describe different allowed scope sets).
- [ ] Create a **PRD → ERD → physical-schema reconciliation matrix** covering missing/inconsistent occupants and occupancy history, lease variations, billing schedules, approvals, clearance cases, receipts, refunds, supplier/AP, bank statement reconciliation, notifications, outbox, idempotency and storage records.
- [ ] Agree a permission matrix and state-transition tables for identity, invitation, lease, invoice, payment verification, journal posting, maintenance, approvals and clearance.
- [ ] Define accounting decisions requiring professional review: tax/withholding, accounting basis, co-tenant liability, cash receipts, deposits, refunds, currency rounding/conversion and closed-period reopening.
- [ ] Choose auth provider/session strategy, private object-storage provider and region, email/SMS vendors, log/error tracking and key management.
- [ ] Document Uganda privacy/data-processing, residency/cross-border transfer, retention and incident-response review; **block production personal data** until approved, without blocking isolated local development.

### Repo and infrastructure

- [ ] Create private GitHub `amaalprop` monorepo following `repo-structure.md`, set branch protection, CODEOWNERS, PR templates, secret scanning and environment protection.
- [ ] Verify/install GitHub App repository access; a successful account login alone is not sufficient for repo-specific operations.
- [ ] Create root Rust Cargo workspace; `apps/core-api`, `apps/worker`, `crates/core`, `crates/persistence`, `crates/security`, `crates/events`, `crates/integrations`, `crates/contracts` and `crates/telemetry`.
- [ ] Create `apps/web` and pnpm workspace, and a minimal `apps/ai-service` skeleton (AI deployment can wait).
- [ ] Add Dockerfiles with root monorepo build context for Rust; local Compose for PostgreSQL, object storage and email sink. Keep local/dev/staging/prod separate.
- [ ] Create `database/migrations/` as the single migration history; set up SQLx migration runner, SQL tests, schema lint and a restricted database role.
- [ ] Start GitHub Actions checks: Rust fmt/clippy/test, TS lint/type/test, Python lint/test when applicable, migration/RLS tests, OpenAPI diff, dependency/container/secret scans.
- [ ] Set up structured logs, request IDs, metrics, traces, exception monitoring and basic health/readiness endpoints.
- [ ] Define environment/secret contracts; connect Vercel web, Render API + worker and a **non-production** Neon environment. Test actual Render→Neon region latency and transaction-pool/RLS behaviour.
- [ ] Prepare Cloudflare DNS/TLS/WAF configuration and test custom-domain/origin compatibility before enabling proxying. Avoid caching private application data.
- [ ] Write database backup/restore and object-storage recovery procedures; set provisional RTO/RPO for approval.
- [ ] Publish a minimal versioned `/api/v1/health`/readiness contract and generated TypeScript client pipeline.

### Exit gate P0

- [ ] A fresh developer checkout can boot local stack from documented commands; CI builds all required executables and runs tests.
- [ ] Staging web can call the API health/readiness route without exposing secrets; authenticated endpoints are introduced in Phase 1.
- [ ] CI can run a migration/rollback test on an isolated database and an initial RLS-deny test.
- [ ] No critical architecture or identity cardinality ambiguity remains in code design; unresolved legal/accounting questions have owners and explicit future blockers.

---

## Phase 1 — Trust boundary: organisation, identity and authorisation

**Goal:** One organisation has one designated owner, explicitly approved members and safe scoped access; no other organisation's records are visible.

**Prerequisite:** P0 decisions for identity, sessions and physical tenant-isolation strategy.

### Database and core modules

- [ ] Migrate `property_organisations`, `organisation_owners`, `user_accounts`, `organisation_memberships`, invitation/verification records and session/auth metadata.
- [ ] Add roles, permission catalogue, role permissions, member-role assignments, reporting relationships, scope assignments and approval authority definitions.
- [ ] Enforce unique `normalised_email` globally, membership cardinality, one active owner per organisation and explicit owner transfer history.
- [ ] Add audit/security event storage, outbox/event envelope and idempotency keys; audit owner grants, access changes and invitations.
- [ ] Add transaction-local organisation context, RLS `USING`/`WITH CHECK`, restricted runtime database role and composite same-organisation FKs.
- [ ] Build authenticated member lookup, default-deny RBAC, scope resolution and versioned permissions; supervisor relationships do not automatically widen scope.
- [ ] Implement secure invitation tokens (short-lived, single-use), contact verification, explicit approval, suspensions/revocation and controlled transfers.
- [ ] Establish private-file metadata and server-authorised access primitives (actual business file usage expands later).

### API and web

- [ ] Organisation registration/configuration and designated-owner onboarding.
- [ ] Login, verification, recovery and secure session handling (managed provider or reviewed self-managed implementation).
- [ ] Staff invitation, approval, role/scope assignment, and organisation settings screens.
- [ ] Cross-organisation and cross-scope error contracts; security-sensitive actions have audited actor attribution.
- [ ] Minimal notification adapter (e.g. invitations) with delivery attempt tracking; full communication automation is Phase 7.

### Exit gate P1

- [ ] Two synthetic organisations demonstrate no cross-organisation reads/writes through API, SQL, files and worker jobs.
- [ ] Duplicate email/membership and second owner are rejected; transfer preserves history.
- [ ] Staff lacking scope cannot use guessed IDs or manipulate URLs to bypass access.
- [ ] Login by itself does **not** grant organisation data access; invitation approval is enforced.

---

## Phase 2 — Property inventory and organisational operating scopes

**Goal:** Model real property structures without artificial buildings/floors, while supporting custom labels and scoped assignments.

**Prerequisite:** P1 organisation model and authorisation.

### Database and core

- [ ] Implement portfolios, properties, buildings, property levels, unit types, rentable units, features and feature assignments.
- [ ] Implement versioned unit occupancy policies (`max_tenant_parties`, `max_persons`, shared vs single modes), even before leases exist.
- [ ] Enforce same-organisation and same-property building/level/unit integrity, property code uniqueness and configurable unit-label uniqueness policy.
- [ ] Support residential, commercial and mixed-use classification, property-level currency/country/timezone fields and flexible addresses.
- [ ] Provide effective employee assignment resolution at organisation/portfolio/property/building and supported other scopes.

### API and web

- [ ] CRUD/list/search/pagination and basic exports for portfolios, properties, buildings, levels, units and occupancy policies.
- [ ] Property/unit selection and mobile-friendly landlord/staff inventory views.
- [ ] Availability state and event history without claiming a unit is occupied merely because a draft lease exists.

### Exit gate P2

- [ ] Register a standalone house **without** fake building/level, a multi-building mall with custom levels and custom unit identifiers such as `MN20`, `PA1`, `PN400`.
- [ ] Property A cannot link a unit to Property B's level/building; other organisations cannot access either.
- [ ] Effective permissions respect property/building assignment, even where reporting relationships differ.

---

## Phase 3 — Tenant parties, portals, leases and lease approvals

**Goal:** Correctly represent who signs, who occupies, and which units they rent; activate leases without concurrency violations.

**Prerequisite:** P1 authorisation and P2 property/occupancy policies.

### Database and core

- [ ] Implement tenant parties (individual/company), tenant persons, representatives, contacts, protected IDs and authorised occupants **as distinct entities/relationships**.
- [ ] Link tenant portal account to verified person + membership with landlord/authorised manager approval.
- [ ] Implement leases, `lease_units`, `lease_participants`, occupancy records/snapshots, lease documents/events, amendments/variations/renewals and termination states.
- [ ] Build **reusable minimum approval engine**: policies, requests, decisions, delegation, effective approver scope and immutable history. Lease activation uses it if required; extend approval templates in Phase 6.
- [ ] Implement multi-unit/multi-party financial responsibility allocations and occupancy policy snapshots; do not assume equal co-tenant liability.
- [ ] Validate draft and activation; lock relevant units/occupancy records in transaction to prevent simultaneous conflicting activations.
- [ ] Preserve history through lease amendments rather than overwrite signed/approved contractual states.

### API and web

- [ ] Staff tenant registration and tenant access invitation/approval, company representatives and document upload.
- [ ] Lease drafting, review/approval, activation, variation/renewal and document lifecycle screens.
- [ ] Tenant portal for authorised profile/lease information (no other tenant's records), mobile-friendly forms.

### Exit gate P3

- [ ] One party rents multiple units; one lease covers multiple units; a shared unit supports several co-tenants up to the configured limits.
- [ ] Occupants and representatives remain distinct from financial tenant parties.
- [ ] Concurrent lease activations cannot exceed max occupancy or create prohibited overlapping active leases.
- [ ] Tenant portal only reveals permitted information, including co-tenant privacy boundaries.

---

## Phase 4 — Authoritative accounting engine and financial safety

**Goal:** Implement the ledger **before** any live invoice/payment posting. Financial workflows remain disabled until this phase's gate passes.

**Prerequisite:** P1 isolation and P3 tenant/lease dimensions; P0 financial policies approved for launch scope.

### Database and core

- [ ] Build organisation chart of accounts, accounting periods, journal entries/lines, source links, reversal links and accounting-control accounts.
- [ ] Implement safe journal draft → review/approval (if policy) → posted → reversal flows; **no mutation/deletion** of posted journals/lines.
- [ ] Enforce exactly one debit/credit side, non-negative amounts, organisation-safe financial dimensions, balanced postings **at transaction/DB boundary**, closed-period checks and source uniqueness/idempotency.
- [ ] Define supported currency precision, base currency, rounding and rate snapshots; if full multi-currency is deferred, reject unsupported cross-currency operations rather than silently convert.
- [ ] Configure posting maps for rent, verified collections, security deposits (liability), refunds, expenses and adjustments; keep platform SaaS billing distinct.
- [ ] Implement accounting service application interface used by Billing, Payments, Expenses and Clearance; no other module writes journal rows directly.
- [ ] Provide ledger views, trial balance and basic source-to-journal reconciliation/report queries.

### Exit gate P4 — **hard financial safety gate**

- [ ] An unbalanced or partially written journal is impossible, including under concurrent requests and injected failures.
- [ ] Posted journals/lines are immutable across APIs and normal DB roles; reversals are linked and auditable.
- [ ] Duplicate source event/idempotency retry cannot double post.
- [ ] Deposits post to liabilities; financial periods close and prevent unauthorised posting.
- [ ] Finance access, RLS, property dimensions and reporting filters pass negative tests.

---

## Phase 5 — Billing, landlord payment instructions and verified collections

**Goal:** Complete the rent-to-ledger flow with **external payment and independent staff verification**.

**Prerequisite:** P3 active leases and P4 accounting engine.

### Billing and invoicing

- [ ] Add charge types, billing schedules, proration/due-date rules, invoice header/lines, adjustments and invoice numbering policy.
- [ ] Implement issue/void/credit lifecycle; issue and post invoice atomically with correct accounting effect.
- [ ] Build outbox-backed recurring invoice worker and unique invoice-generation keys so retries cannot duplicate rent.
- [ ] Define tenant finance green/orange/red standing from invoices, verified allocations, grace policy and dated history; never equate pending proof with paid.

### Payment verification and accounting

- [ ] Configure payment instructions and effective inheritance/override precedence (chosen in P0); snapshot displayed instruction for each submission.
- [ ] Tenant can submit amount, currency, purpose, date, external reference and optional file proof; create **pending** submission only.
- [ ] Finance team verifies independently against actual bank/mobile-money/cash account evidence; reviewer records reference, amount, currency, reason and audit trail.
- [ ] Only verified submissions create a confirmed payment, posting and later allocations; duplicates/partial/overpayments/disputes are controlled.
- [ ] Add payments, payment events, allocation state/reversals, receipt numbers/status, unapplied tenant credits, refunds and deposit receipt posting.
- [ ] Implement **basic** statement matching/reconciliation (including manual import/entry + unmatched cases). Advanced automated bank integrations are later work.
- [ ] Build tenant invoices, effective payment instructions, submissions, verified receipts and statements screens; finance verification queue and ledger links.
- [ ] Add reliable finance notification events (delivery templates and automation mature in Phase 7).

### Exit gate P5 — **hard payment launch gate**

- [ ] Submitted screenshot/reference **does not** post cash, mark invoice paid or issue receipt; independent authorised verification is mandatory.
- [ ] Verified payment creates exactly one payment, balanced journal and auditable receipt; retries and concurrent review do not duplicate anything.
- [ ] A payment can settle several invoices; several payments can settle one invoice; credits/overpayments remain explicit and reconciled.
- [ ] Tenant statements agree with journal/control account balances; receipts have payment date and verification date.
- [ ] Refund, reversal, period-close and reconciliation exceptions are tested and reviewed.

---

## Phase 6 — Maintenance, work orders, expenses, inspections and clearance

**Goal:** Complete property operations and tenant exit without conflating legal termination, physical inspection and financial clearance.

**Prerequisite:** P3 leases/occupancy, P4 ledger, P5 deposits/charges; P3 approval-engine foundation.

### Maintenance and expense operations

- [ ] Create maintenance requests, prioritisation, contractors/suppliers, work orders, assignment, work-order events and supporting private evidence.
- [ ] Add supplier registry, expenses, approval thresholds, payment evidence and accounting links; connect maintenance costs to unit/property dimensions.
- [ ] Extend common approvals for expenses, repairs and exceptions (sequential/parallel as policy, delegation, self-approval restrictions, escalation).

### Inspection and move-out

- [ ] Build versioned inspection templates/items, inspections, condition/evidence capture and secure photos.
- [ ] Implement tenant move-out request and a **clearance case** linking lease, affected units, financial settlement, evidence, handover and decisions.
- [ ] Enforce default chain: caretaker inspection → property manager verification → finance deposit settlement → landlord/delegated final approval.
- [ ] Calculate approved damage deductions, disputed items, open rent/utilities, refundable deposit balance and any outstanding amount, with appropriate journals/refunds.
- [ ] Close occupancy and update availability only through approved transitions; preserve original lease, accounting and audit history.
- [ ] Distinguish lawful termination/notice from financial/physical clearance; flag legal questions for staff rather than automate legal enforcement.

### Exit gate P6

- [ ] A caretaker cannot self-approve final clearance; manager and finance responsibilities cannot be bypassed.
- [ ] A disputed deduction is preserved as disputed, not silently treated as accepted.
- [ ] Deposit refund/deduction entries balance and keep the original liability and evidence traceable.
- [ ] A complete tenant exit passes E2E and security tests on desktop and a phone viewport.

---

## Phase 7 — Communications, reports and complete non-AI MVP

**Goal:** Bring together role-specific operational interfaces, reliable notifications and defensible reports.

**Prerequisite:** P1–P6 source data and domain events.

### Deliverables

- [ ] Templates and delivery history for email/SMS, reminders, retries, opt-out/consent where applicable and operator-visible failed/dead-letter jobs.
- [ ] Worker tasks for rent due/arrears refresh, receipt delivery, lease expiry, maintenance and inspection reminders, approval escalation and scheduled report exports.
- [ ] Landlord, director, portfolio/property manager, caretaker, finance officer and tenant dashboards, each permission-filtered.
- [ ] Occupancy/vacancy, arrears, statement, rent roll, maintenance status, audit/approval backlog and basic staff performance views.
- [ ] Ledger-based financial reports: tenant/property statements, general ledger, trial balance, P&L, balance sheet and cash flow to the extent supported by approved accounting policy.
- [ ] Pagination, filtering, asynchronous exports, low-bandwidth/responsive behavior, localisation-ready date/number/currency formatting and status descriptions (not colour alone).
- [ ] Privacy-safe report/export generation, audit of sensitive access, retention/deletion task planning and operator runbooks.

### Exit gate P7

- [ ] All key staff and tenant journeys function end-to-end without direct bank API credentials.
- [ ] Reports reconcile to authorised source records; cross-organisation and cross-property exports reveal no additional data.
- [ ] Failed jobs are retryable and visible, not silently lost; repeated jobs do not duplicate finance records.
- [ ] Business workflow, accessibility, performance and audit acceptance suites pass in staging.

---

## Phase 8 — Optional AI/document intelligence, strictly assistive

**Goal:** Add carefully scoped intelligence, not a second source of financial truth.

**Prerequisite:** P1 access/privacy controls, P3 document permission model and P7 stable authorised read models. **Not a launch dependency.**

### Deliverables

- [ ] Deploy private Python FastAPI service with service-to-service auth; core API supplies verified actor/organisation/scope context.
- [ ] Lease/document extraction with confidence scores, source citations and explicit human confirmation before changing any authoritative record.
- [ ] Authorised semantic search over private files/records; use pgvector if compatible with chosen Neon environment and only under enforced scope.
- [ ] Natural-language reporting using **allowlisted read-only operations**, never unrestricted model-generated production SQL.
- [ ] Evaluation/observability for model versions, retrieval permissions, prompt injection, costs, latency and data minimisation.
- [ ] Human review for communications, recommendations, anomalies and maintenance categorisation.
- [ ] Keep financial verification, refunds, journal posting, approval and final clearance exclusively in controlled Rust workflows.

### Exit gate P8

- [ ] Cross-organisation retrieval and prompt-injection tests cannot expose another organisation's data.
- [ ] Unsupported/low-confidence outputs request human validation rather than silently modifying records.
- [ ] Disabling AI leaves core rent, accounting and tenant exit flows fully functional.

---

## Phase 9 — Production readiness, legal sign-off and controlled release

**Goal:** Make the full operational MVP safe to operate with real customers.

**Prerequisite:** P0–P7 exit gates. P8 only if AI is part of the approved launch scope.

### Security, reliability and provider checks

- [ ] Confirm local legal/accounting review on privacy, registration/processing obligations, cross-border providers, consent, retention, statutory tax/accounting and deposit/refund policies.
- [ ] Reconfirm production configuration: Render and Neon regions/latency, PostgreSQL version/extensions, pooling and `SET LOCAL`/RLS behavior, restricted DB role, migration role, separate staging/prod accounts/projects.
- [ ] Confirm Cloudflare→Vercel/Render DNS, TLS, custom-domain, direct-origin protection, WAF/rate limits and explicit **no-cache** behaviour for private paths.
- [ ] Confirm private object-storage region, signed link expiry, encryption, malware scanning where available, retention, backup and restore.
- [ ] Implement secret rotation, MFA/admin controls, audit access, vulnerability patch cadence, alerting, uptime, error budgets, data migration/recovery and documented incident response.
- [ ] Perform load and concurrency testing on invoicing, payments, lease activation and reporting, using realistic synthetic data; agree measurable latency/concurrency targets.
- [ ] Test database and object recovery, target RTO/RPO and operation in the face of failed worker/external notification service.
- [ ] Execute production-like migration and rollback/recovery drills on staging. Never depend on irreversible blind downgrade of posted financial data.
- [ ] Complete accessibility and phone workflow tests, staged rollouts and operator/support training.

### Mandatory production E2E acceptance path

- [ ] Organisation registration + designated owner → staff invite/approval → RBAC/scopes → property/building/standalone units → tenant/co-tenants/occupants → multi-unit lease → approval/activation → recurring invoice → landlord payment instruction → tenant **submission only** → independent finance verification → one payment + allocation → balanced immutable journal + receipt + statement → maintenance/work order → move-out request → caretaker inspection → manager review → finance deposit settlement → landlord/delegate clearance → occupancy closure.
- [ ] Repeat with two organisations, malicious cross-scope identifiers, concurrency, duplicate webhook/event/API retries and failed jobs; no data leaks, duplicate postings or lost audit trail.

### Release gate P9

- [ ] Reviewed PRs, clean CI, schema/contract reconciliation, approved migrations and checked backups.
- [ ] Independent review of financial invariants and high-risk access controls is recorded.
- [ ] Staging smoke tests, end-to-end security regression and restore drill pass.
- [ ] Operational ownership, incident contacts, retention/deletion procedures and key provider authorisations are documented.
- [ ] Production release approved; monitored rollout completed; rollback/recovery strategy exercised.

---

## Phase 10 — Post-validation expansion (NOT initial dependencies)

- [ ] Optional bank/mobile-money APIs, external payment webhooks, automated reconciliation and payment-gateway integration for **AMAALPROP SaaS subscriptions** (separate ledger).
- [ ] Advanced suppliers/accounts payable, commercial leases, property service-charge allocation, metering and tax/localisation by country.
- [ ] Mature foreign-currency accounting, exchange differences and country-specific financial reports after accounting policy approval.
- [ ] Plan/entitlement management and subscription billing, if commercially needed.
- [ ] Additional AI: anomaly detection, maintenance trends and validated analytics as sufficient high-quality historical data accumulates.
- [ ] Native mobile clients consuming the same `/api/v1` contract when demand supports them.
- [ ] Revisit dedicated queues, cache, read replicas, search infrastructure and C++ performance extensions **only after profiling**. No default microservice migration.

### Exit gate P10

- [ ] Each extension has a separate ADR, measurable business need, threat model, tenant-isolation test plan and backward-compatible migration/rollout strategy.

---

## B. Cross-phase schema reconciliation checklist

Before creating each affected table, ensure these source-document concepts have a clear physical owner and lifecycle:

| Area | Confirm/add in physical schema | First implementation |
|---|---|---|
| Identity | global normalised email, owner transfer, membership cardinality, invitation/approval and account recovery | P1 |
| Scopes | roles, permission assignments, reporting vs scope vs approval authority | P1 |
| Property | optional building/level hierarchy, occupancy policy history, availability logic | P2 |
| Tenant/lease | tenant party/person vs occupant, `lease_units`, `lease_participants`, occupancy history, amendments, concurrency | P3 |
| Approval | policy, steps, requests, immutable actions, delegation and self-approval controls | P3, P6 |
| Accounting | currencies/rounding, periods/locks, journal source mapping, immutable posted lines, reversal, deposits | P4 |
| Billing | schedule rules, invoices/lines, credit notes, idempotent generation, tenant status history | P5 |
| Payments | instructions/scopes, verification evidence, payment and allocation, numbered receipts, refunds/reversals, unallocated credit, reconciliation | P5 |
| Operations | supplier/AP records, maintenance, work orders, inspection versions, clearance case and deposit settlement | P6 |
| Support | stored files, audit, outbox, idempotency, notification attempts, exports, retention | P1–P7 |
| AI | scoped embeddings, model access/audit, human extraction review, no autonomous finance | Optional P8 |

## C. Release-level negative acceptance tests (never waive)

1. An actor in Organisation A cannot see, export, update, approve or download Organisation B's records through API, DB, files, reports, worker or AI.
2. A person without membership, permission or asset scope cannot bypass enforcement with a different ID or URL.
3. Duplicate email and unauthorised owner/tenant portal activation are rejected.
4. Concurrent lease activations respect unit capacity, co-tenancy rules and overlaps.
5. A submitted payment reference/screenshot is not a verified payment and does not issue a receipt or journal.
6. Repeated verification, invoice cron, outbox event and job retry do not duplicate financial records.
7. A journal cannot be unbalanced, partially posted, edited after posting, or posted into a closed period without authorised policy.
8. Deposits never silently become rent income; overpayment and refund remain traceable liabilities/transactions.
9. An unassigned caretaker cannot final-approve clearance or see restricted tenant identities/full finance by default.
10. Physical clearance and contractual/legal termination are not collapsed into one status, and disputed deductions are not marked agreed.
11. Signed file URLs expire, file access is re-authorised and unauthenticated CDNs do not cache sensitive data.
12. Tenant A cannot read Tenant B's data even when sharing a unit, except explicitly permitted shared information.

## D. Connector readiness — read-only verification on 13 Nov 2026

This is **current integration access**, not permission to mutate or deploy AMAALPROP production resources.

| Provider/connector | Evidence checked | Status | Next action during Phase 0 |
|---|---|---|---|
| GitHub | Authenticated profile and app installation lookup | **Account connection works; installation list was empty** | Create private repository; explicitly install/grant the GitHub app repository access and verify permissions |
| Vercel | Read teams and project list | **Connected for read access** | Link new `apps/web` project; confirm team/project permissions and environments |
| Render | Read workspaces and services | **Connected for read access** | Create API/worker staging services, then test deploy access; AI service later |
| Neon | Read organisations and project list | **Connected for read access** | Create separate AMAALPROP development/staging/production projects/branches and restricted DB roles |
| Cloudflare | Authenticated identity, zones and R2 bucket read | **Connected for read access; R2 listing works** | Verify AMAALPROP domain, DNS/WAF write permissions and storage region/permissions if using R2 |
| Email/SMS, authentication, monitoring, AI inference | Vendor integrations not yet selected or tested for this product | **Not yet verified** | Select required providers and verify account permissions before relevant phase; no direct bank/mobile-money connectors required for MVP |

No AMAALPROP repo, service, database, zone, bucket, credentials or deployment was created or changed during this review. A connection can expose read APIs without having permission to create production resources; verify *resource-specific* permissions when each service is provisioned.

## E. Suggested next implementation issue

Create GitHub issue **`P0-01: Approve Rust architecture and reconcile PRD/ERD schema`** with the above D-01 through D-11 decisions and acceptance criteria, then initialize the repository and CI in the same phase. This prevents infrastructure scaffolding from locking in contradictory identity, financial or data-model assumptions.
