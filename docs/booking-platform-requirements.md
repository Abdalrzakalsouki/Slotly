# Booking Platform --- Requirements (staged)

*Working name: **Slotly** (placeholder, rename freely).*

A self-hostable, open-source booking and scheduling platform for small
service businesses (barbers, clinics, tutors, studios). Each business is
a **tenant**. Customers book appointment slots online. The goal is to
**learn backend engineering** by building something real, and to end up
with a project other people can self-host.

Companion to `full-stack-engineering-roadmap-80-20.md`. Section numbers
below (e.g. "Roadmap 10") refer to that file.

------------------------------------------------------------------------

## 1. How to use this document

-   Work **one stage at a time**. Each stage is small (a few days of
    part-time work) and ends with something that runs and is tested.
-   Do not start a stage until the previous one's **Done when** list is
    checked.
-   Every stage must end with: tests passing, migration committed, API
    documented in OpenAPI, and a short note in `docs/learning-log.md`
    explaining in your own words what you learned and what broke.
-   **Backend first.** The frontend is thin and time-boxed (Stage 11 and
    small pieces later). Until then, test with Swagger UI, `.http`
    files, or Bruno.
-   If a stage feels too big, split it. Never skip a stage's tests to
    save time.

### Stage sizes

S = 1--2 days, M = 3--5 days, L = 1--2 weeks (part-time).

### Roadmap of stages

| Part | Stages | Theme |
|---|---|---|
| A. Foundations | 0--11 | Setup, schema, auth, tenants, RLS, availability, safe booking |
| B. Reliability | 12--16 | Background jobs, reminders, caching, pagination, audit |
| C. Product features | 17--26 | Payments, webhooks, realtime, search, uploads |
| D. Production and release | 27--32 | Docker, CI/CD, deploy, monitoring, backups, open-source release |

**Milestone after Part A:** a complete booking API without payments.
You can stop there and already have a portfolio-worthy backend.

------------------------------------------------------------------------

## 2. Scope

### In scope

-   Multi-tenant businesses with staff, services, working hours, time
    off.
-   Public booking (customers do not need an account) and a business
    dashboard API.
-   Safe booking under concurrency (no double bookings).
-   Emails, reminders, background jobs.
-   Stripe payments (deposit or full prepayment) with webhooks.
-   Outgoing webhooks for tenants.
-   Realtime dashboard and live slot availability.
-   Full-text search with Arabic support.
-   Self-hosting with Docker.

### Out of scope (v1)

-   Native mobile app, Google/Outlook calendar sync, video meetings.
-   Multi-location chains, resource booking (rooms, equipment).
-   Marketplace of businesses, reviews, loyalty points.
-   Microservices, Kubernetes, Kafka (Roadmap 20--21; not needed).

------------------------------------------------------------------------

## 3. Tech stack

| Area | Choice |
|---|---|
| Language / runtime | TypeScript (strict), Node.js |
| Framework | NestJS (modular monolith) |
| Database | PostgreSQL |
| ORM / migrations | Drizzle (hand-written SQL migrations for what Drizzle cannot express, such as exclusion constraints and RLS; check current Drizzle docs) |
| Validation | Zod |
| Cache and queues | Redis + BullMQ |
| Email | Resend in production, Mailpit in development |
| Payments | Stripe (test mode) behind a provider interface |
| Realtime | NestJS Gateway + Socket.IO (Redis adapter) |
| Object storage | MinIO locally, S3 / Cloudflare R2 in production |
| Testing | Vitest, Supertest, Testcontainers |
| Logging / errors | Pino, Sentry |
| Containers / CI | Docker, Docker Compose, GitHub Actions |
| API docs | Swagger / OpenAPI |
| Thin frontend | Next.js (only what is needed to demo the backend) |

------------------------------------------------------------------------

## 4. Domain and business rules

### Roles

-   **Owner**: full control of a business, billing, can delete it.
-   **Admin**: manages staff, services, bookings, settings.
-   **Staff**: sees and manages only their own schedule and bookings.
-   **Customer (guest)**: books with name, email, phone; no account
    required. Can view or cancel their booking through a signed link.

A **user** can belong to several businesses with a different role in
each (a **membership**).

### Core entities

`user`, `tenant` (business), `membership`, `service`, `staff`
(a membership that can be booked), `working_hours`, `time_off`,
`customer`, `booking`, `payment`, `webhook_endpoint`,
`webhook_delivery`, `audit_log`, `invitation`, `idempotency_key`,
`processed_event`.

### Business rules (these drive the tests)

| ID | Rule |
|---|---|
| B1 | All instants are stored as UTC (`timestamptz`). Each tenant has an IANA timezone (e.g. `Asia/Damascus`). Slots are calculated in the tenant's timezone, including DST changes. |
| B2 | A booking occupies its staff member from `start` to `end + buffer`. `end = start + service duration`. |
| B3 | Two active bookings for the same staff member **must never overlap**. This is enforced by the database, not only by application code. |
| B4 | Booking states: `pending_payment`, `confirmed`, `cancelled`, `completed`, `no_show`, `expired`. Only the transitions in Stage 8 are allowed. |
| B5 | Each tenant sets a minimum lead time (e.g. 2 h) and a maximum advance (e.g. 60 days). |
| B6 | Each tenant sets a cancellation window (e.g. free cancellation until 24 h before). |
| B7 | Money is stored as integer minor units (cents) with a currency. Never floats. |
| B8 | The server computes price and duration from the service. Never trust values sent by the client. |
| B9 | Every tenant-owned row has `tenant_id`. A request can only ever touch rows of the tenant it is authorized for. |
| B10 | Every action that can be retried (create booking, pay, webhook, job) must be idempotent. |

### Non-functional requirements

-   Availability query (public) answers in under 300 ms at p95 for a
    tenant with 10 staff and 60 days of bookings.
-   Booking creation is correct under 50 concurrent requests for the
    same slot: exactly one succeeds.
-   No secrets in the repository. All config from environment variables,
    validated at startup.
-   Errors follow one consistent JSON shape (Stage 2).
-   Every endpoint is covered by an authorization test (right role
    succeeds, wrong role or wrong tenant fails).

------------------------------------------------------------------------

# Part A --- Foundations

## Stage 0 --- Project setup (S)

*Practices: Roadmap 2, 3, 4, 9.*

**Build**

-   [ ] Git repo, branch workflow (feature branches, small PRs even if
    you are the only reviewer), conventional commits.
-   [ ] NestJS project, TypeScript `strict: true`, ESLint, Prettier.
-   [ ] Docker Compose with PostgreSQL, Redis (unused yet), Mailpit.
-   [ ] Config module: environment variables validated with Zod at
    startup; app refuses to boot on invalid config.
-   [ ] Pino structured logging with a request ID on every log line.
-   [ ] `GET /health` (liveness) and `GET /ready` (checks the database).
-   [ ] Swagger UI at `/docs`.
-   [ ] Test harness: Vitest + Supertest + Testcontainers starting a
    real Postgres for integration tests.
-   [ ] GitHub Actions: lint, typecheck, test on every push.

**Done when**

-   `docker compose up` + `npm run start:dev` gives a running API.
-   One integration test hits `/ready` against a Testcontainers database.
-   CI is green.

## Stage 1 --- Schema foundations and migrations (M)

*Practices: Roadmap 1, 7.*

**Build**

-   [ ] Drizzle setup, migrations committed to git, a seed script.
-   [ ] Tables: `users`, `tenants`, `memberships` with primary keys,
    foreign keys, unique constraints (e.g. unique email, unique tenant
    slug), `created_at` / `updated_at`.
-   [ ] Use `uuid` or `bigint` identity keys (decide and write down
    why in the learning log).
-   [ ] Repository layer for these tables (thin, no business logic).
-   [ ] Write 10 raw SQL queries against the schema by hand (JOINs, a
    GROUP BY, a CTE, a window function) and keep them in
    `docs/sql-practice.sql`.

**Done when**

-   Migrations run from an empty database and are repeatable in tests.
-   You can explain each constraint and what error you get when it is
    violated.

## Stage 2 --- Auth I: registration and login (M)

*Practices: Roadmap 5, 6, 8.*

**Build**

-   [ ] `POST /auth/register`, `POST /auth/login`, `POST /auth/logout`,
    `GET /me`.
-   [ ] Password hashing with Argon2id. Never log passwords or tokens.
-   [ ] Short-lived access token + refresh token with **rotation**
    (each refresh issues a new refresh token; reuse of an old one
    revokes the whole family). Refresh tokens stored hashed in the
    database.
-   [ ] Zod validation on every request body (global pipe).
-   [ ] Global exception filter: one error shape
    `{ error: { code, message, details?, requestId } }`.
-   [ ] Login rate limiting (per IP and per email).
-   [ ] Decide and write down: JWT vs. server-side sessions, and why.

**Done when**

-   Tests: wrong password, unknown email (same response and timing
    class), reused refresh token revokes the family, expired token.
-   Error responses are identical in shape across all failures.

## Stage 3 --- Auth II: email verification and password reset (S)

*Practices: Roadmap 5, 6.*

**Build**

-   [ ] Email verification token (single use, expiry, stored hashed).
-   [ ] Password reset flow (single use, short expiry, invalidates all
    sessions after reset).
-   [ ] Emails sent synchronously through a mail service interface (real
    jobs come in Stage 12). Mailpit in development.
-   [ ] Do not reveal whether an email is registered (reset endpoint
    always returns the same response).

**Done when**

-   Tests cover expired token, reused token, and tampered token.

## Stage 4 --- Tenants, memberships, RBAC (M)

*Practices: Roadmap 4, 5, 10.*

**Build**

-   [ ] `POST /tenants` (creator becomes owner), tenant slug, timezone,
    currency, default language.
-   [ ] Tenant resolved per request from the URL (`/t/:slug/...`) and
    checked against the caller's membership.
-   [ ] Roles owner / admin / staff as a NestJS guard + decorator
    (`@Roles('admin')`).
-   [ ] Invitations: invite by email with a role, accept via token,
    expire, revoke.
-   [ ] Rules: a tenant always has at least one owner; only an owner can
    remove an owner or delete the tenant.

**Done when**

-   Authorization tests: every endpoint tried with each role and with a
    user from another tenant.

## Stage 5 --- Tenant isolation with Row-Level Security (M)

*Practices: Roadmap 1, 6, 10.*

**Build**

-   [ ] `tenant_id` on every tenant-owned table, with composite unique
    constraints (unique per tenant, not global).
-   [ ] Two database roles: the **owner/migration role** and a
    **least-privilege app role** (no `BYPASSRLS`, no superuser, no DDL).
-   [ ] RLS policies on tenant tables based on
    `current_setting('app.tenant_id')`.
-   [ ] Per-request transaction that runs `SET LOCAL app.tenant_id` (or
    `set_config(..., true)`) before any query.
-   [ ] Request-scoped tenant context in NestJS.

**Done when**

-   A "cross-tenant leak" test suite: a query that forgets its `WHERE`
    clause still returns only the current tenant's rows.
-   You can explain why `SET LOCAL` must be inside a transaction and
    what goes wrong with a connection pool if it is not.

## Stage 6 --- Services and staff (S)

*Practices: Roadmap 7, 8.*

**Build**

-   [ ] CRUD for `services` (name, description, duration in minutes,
    buffer, price in cents, active flag).
-   [ ] Staff = membership that can be booked; link staff to the
    services they perform.
-   [ ] Soft delete / archive instead of hard delete (bookings keep
    referencing them).
-   [ ] Simple offset pagination (`?page=&pageSize=`) for lists. **Keep
    it simple on purpose**: you will replace it in Stage 15 and see why.

**Done when**

-   Validation tests (negative duration, price as float, other tenant's
    staff).

## Stage 7 --- Availability and slot generation (M)

*Practices: Roadmap 1, 3, 7.*

**Build**

-   [ ] `working_hours` (per staff, per weekday, multiple ranges per
    day), `time_off` (date ranges, single days).
-   [ ] A **pure function**
    `generateSlots(rules, existingBookings, range, tz)` with no
    database access.
-   [ ] `GET /public/t/:slug/availability?service=&staff=&date=` returns
    slots in UTC with the tenant timezone.
-   [ ] Handle B1 (timezones and DST), B2 (buffers), B5 (lead time,
    max advance).
-   [ ] Slot step (e.g. every 15 min) is a tenant setting.

**Done when**

-   30+ unit tests for `generateSlots`, including: DST change day,
    booking at midnight, overlapping time off, buffer between
    appointments, no staff available.

## Stage 8 --- Booking v1 and the booking state machine (M)

*Practices: Roadmap 2, 7, 9.*

**Build**

-   [ ] `bookings` table, `customers` table (per tenant).
-   [ ] `POST /public/t/:slug/bookings` (service, staff, start time,
    customer details). Server recomputes `end`, price (B8), and checks
    the slot is still free by calling the slot generator.
-   [ ] State machine, implemented as one function with an allowed
    transitions table:
    `pending_payment → confirmed | expired | cancelled`,
    `confirmed → cancelled | completed | no_show`. Anything else is
    rejected.
-   [ ] Cancel endpoint with the B6 cancellation window.

**Done when**

-   Unit tests for every allowed and forbidden transition.
-   **On purpose, this naive version still has a race condition.** Write
    a test that fires 20 parallel requests for the same slot and show
    that more than one succeeds. Keep that failing test; Stage 9 fixes
    it.

## Stage 9 --- Safe booking under concurrency (M)

*Practices: Roadmap 7, 12 (idempotency).*

**Build**

-   [ ] Enforce B3 in the database with an **exclusion constraint** on
    `(staff_id, tstzrange(start, end + buffer))` for active statuses
    (needs the `btree_gist` extension; hand-written migration).
-   [ ] Translate the constraint violation into a clean `409 Conflict`
    with code `SLOT_TAKEN`.
-   [ ] Wrap booking creation in a transaction.
-   [ ] **Idempotency-Key header**: the same key returns the original
    response instead of creating a second booking. Store keys with a
    request fingerprint and expiry.
-   [ ] Compare in your learning log: exclusion constraint vs.
    `SELECT ... FOR UPDATE` vs. a unique slot table. When would you
    choose each?

**Done when**

-   The failing test from Stage 8 now passes: exactly one of 50
    parallel requests succeeds, the rest get `SLOT_TAKEN`.
-   Retrying a request with the same idempotency key returns the same
    booking, not a new one; a different payload with the same key is
    rejected.

## Stage 10 --- Dashboard API and customer self-service (M)

*Practices: Roadmap 5, 8.*

**Build**

-   [ ] Staff/admin endpoints: list bookings (filter by date, staff,
    status), view one, cancel, mark completed / no-show, reschedule
    (re-checks availability atomically).
-   [ ] Staff only see their own bookings; admins see all.
-   [ ] Customer access with a **signed, expiring link** (no account):
    view and cancel their own booking.
-   [ ] Customer records: find-or-create by email within the tenant.

**Done when**

-   Authorization tests for staff vs. admin vs. other tenant vs. guest
    link. A guest link for booking A cannot read booking B.

## Stage 11 --- API hardening and thin client (M)

*Practices: Roadmap 6, 8.*

**Build**

-   [ ] Security pass using the OWASP Top 10 as a checklist; write the
    result in `docs/security-checklist.md`.
-   [ ] Secure headers (Helmet), strict CORS, secure cookies if used,
    request size limits, global rate limiting.
-   [ ] OpenAPI is complete and accurate; API versioning (`/v1`).
-   [ ] Mass-assignment check: no endpoint accepts fields it should not
    (e.g. `role`, `tenant_id`, `price`).
-   [ ] **Thin frontend (time-box: 3 days):** a Next.js public booking
    page (pick service, slot, enter details) and a bare dashboard list.
    No styling effort.

**Done when**

-   A person can book through the page and the admin sees it.
-   `npm audit` is clean or each finding is explained.

> **Milestone A:** complete booking API without payments. Tag `v0.1.0`.

------------------------------------------------------------------------

# Part B --- Reliability and async

## Stage 12 --- Background jobs and email (M)

*Practices: Roadmap 17.*

**Build**

-   [ ] BullMQ + Redis. Separate **worker process** (same codebase).
-   [ ] Move email sending to a job: booking confirmation to the
    customer, new-booking notice to staff.
-   [ ] Retries with exponential backoff, a maximum attempt count,
    dead-letter queue (failed jobs kept and inspectable).
-   [ ] Jobs are **idempotent**: processing the same job twice sends one
    email (store a `sent` marker keyed by booking + email type).
-   [ ] **Transactional outbox, light version:** write the "send email"
    intent in the same database transaction as the booking, and a
    dispatcher enqueues it. This prevents "booking saved, job lost."
-   [ ] Email provider interface so Resend can be swapped or mocked.

**Done when**

-   Test: the email provider throws twice then succeeds; exactly one
    email is delivered.
-   Test: the same job delivered twice produces one email.

## Stage 13 --- Reminders and scheduled jobs (S)

*Practices: Roadmap 17.*

**Build**

-   [ ] Delayed job: reminder email N hours before the appointment
    (per-tenant setting).
-   [ ] Rescheduling or cancelling removes or replaces the reminder.
-   [ ] Repeatable job: mark past confirmed bookings as `completed`
    (or flag for review).
-   [ ] Timezone-correct reminder text (shown in the tenant timezone).

**Done when**

-   Tests use a fake clock; no test waits for real time.

## Stage 14 --- Caching (S)

*Practices: Roadmap 17.*

**Build**

-   [ ] Cache-aside for tenant lookup by slug and for public
    availability results (short TTL).
-   [ ] Invalidate on booking create/cancel, working-hours change,
    time-off change.
-   [ ] Never cache anything authorization-related without including the
    user and tenant in the key.
-   [ ] Measure before and after (simple benchmark) and record it.

**Done when**

-   Test proves a new booking is reflected in availability immediately
    (no stale slot shown as free).
-   The app still works correctly if Redis is down (just slower).

## Stage 15 --- Cursor pagination, filtering, sorting (S)

*Practices: Roadmap 14.*

**Build**

-   [ ] Replace offset pagination on bookings and customers with
    **cursor (keyset) pagination**.
-   [ ] Stable ordering with a unique tiebreaker (e.g.
    `(start_time, id)`), cursor encoded as opaque base64.
-   [ ] Filters and sort options that work with cursors.
-   [ ] Add the right composite indexes; verify with `EXPLAIN ANALYZE`.
-   [ ] Demonstrate the offset problem: insert rows while paging and
    show duplicates or skips with offset but not with cursors.

**Done when**

-   Test: inserting a booking between page requests does not duplicate
    or skip any row.

## Stage 16 --- Audit log and cancellation policies (S)

*Practices: Roadmap 6, 7.*

**Build**

-   [ ] `audit_log` (who, what, when, tenant, before/after summary) for
    sensitive actions: role changes, booking cancels, settings, refunds.
-   [ ] Append-only: the app role cannot update or delete audit rows.
-   [ ] Cancellation policy engine: free cancellation until the window,
    after that a fee percentage (used by Stage 19).
-   [ ] Policy is a pure, unit-tested function.

**Done when**

-   Audit rows cannot be changed by the app database role (test it).

------------------------------------------------------------------------

# Part C --- Product features

## Stage 17 --- Payments I: Stripe Checkout (M)

*Practices: Roadmap 11.*

**Build**

-   [ ] Stripe test mode, keys from environment variables.
-   [ ] `payments` table; amount in cents + currency (B7).
-   [ ] Payment provider interface (Stripe is one implementation; keep
    the door open for local gateways).
-   [ ] Booking with a required deposit starts as `pending_payment`
    and **holds the slot** for N minutes (the exclusion constraint from
    Stage 9 already covers pending bookings).
-   [ ] `POST` creates a Stripe Checkout Session with an idempotency key
    and returns the URL. The success page does **not** mark the booking
    paid; only the webhook will (Stage 18).
-   [ ] Per-service setting: no payment / deposit % / full payment.

**Done when**

-   A test-mode card completes Checkout and the booking stays
    `pending_payment` until Stage 18. You can explain why.

## Stage 18 --- Payments II: receiving webhooks (M)

*Practices: Roadmap 12, 11.*

**Build**

-   [ ] Webhook endpoint that verifies the Stripe signature on the
    **raw body**.
-   [ ] Respond `2xx` fast; store the event, process it in a job.
-   [ ] `processed_event` table with a unique event ID: duplicates are
    ignored.
-   [ ] Handle out-of-order events (never move a booking backwards;
    compare timestamps/state).
-   [ ] Events: `checkout.session.completed`, `payment_intent.*`
    failures. Booking becomes `confirmed`, confirmation email is queued.
-   [ ] Use the Stripe CLI to replay and trigger events locally.

**Done when**

-   Tests: bad signature rejected; same event delivered 3 times
    changes state once; event arriving after cancellation does not
    resurrect the booking.

## Stage 19 --- Payments III: failures, expiry, refunds, reconciliation (M)

*Practices: Roadmap 11, 12, 17.*

**Build**

-   [ ] Scheduled job expires unpaid `pending_payment` bookings after N
    minutes → `expired`, slot freed.
-   [ ] Late payment on an expired booking: if the slot is free,
    confirm; otherwise automatic refund and notify.
-   [ ] Refund on cancellation according to the policy from Stage 16;
    refund is idempotent.
-   [ ] **Reconciliation job**: periodically compares paid Stripe
    sessions with your database and fixes mismatches. This handles
    "customer paid, server crashed before saving."
-   [ ] Payment failure handling and user-facing statuses.

**Done when**

-   Simulated crash test: webhook processed, then job killed mid-way;
    after restart the final state is correct and money is neither lost
    nor double-refunded.

## Stage 20 --- SaaS plans and tenant limits (optional, S)

*Practices: Roadmap 10, 11.*

**Build**

-   [ ] Tenant subscription (Stripe Billing, test mode): free / pro.
-   [ ] Limits per plan (staff count, bookings per month, webhooks).
-   [ ] Enforcement in one place (a guard / service), clear error
    `PLAN_LIMIT_REACHED`.
-   [ ] Subscription state kept in sync from webhooks.

**Done when**

-   Downgrading a tenant over its limit blocks new items but never
    deletes existing data.

## Stage 21 --- Outgoing webhooks for tenants (M)

*Practices: Roadmap 12, 6.*

**Build**

-   [ ] Tenants register webhook URLs and choose events
    (`booking.created`, `booking.cancelled`, `payment.succeeded`).
-   [ ] Payloads signed with HMAC and a per-endpoint secret, with a
    timestamp to prevent replay.
-   [ ] Delivery via jobs: retries with backoff, disable endpoint after
    repeated failure.
-   [ ] Delivery log (status code, response time, attempts) and manual
    replay.
-   [ ] **SSRF protection**: block private/internal IP ranges and
    redirects to them, set timeouts, limit response size.
-   [ ] Each event has a unique ID so receivers can deduplicate.

**Done when**

-   Tests: signature verifies; an endpoint pointing to `127.0.0.1` or
    a cloud metadata address is refused; failed endpoint gets retried
    then disabled.

## Stage 22 --- Realtime I: dashboard updates (M)

*Practices: Roadmap 13.*

**Build**

-   [ ] NestJS WebSocket gateway with Socket.IO.
-   [ ] Authenticate the socket handshake with the access token.
-   [ ] Rooms per tenant (and per staff for staff-only events).
-   [ ] Emit `booking.created`, `booking.updated`, `booking.cancelled`
    from the same service layer that changes the data (not from the
    controller).
-   [ ] Authorization is checked on **join and on every event**; losing
    a role or being removed from a tenant disconnects the socket.
-   [ ] Tiny demo page showing live bookings.

**Done when**

-   Tests with two socket clients from different tenants: client B
    never receives tenant A events.

## Stage 23 --- Realtime II: live slots and scaling (M)

*Practices: Roadmap 13, 17.*

**Build**

-   [ ] Public (anonymous) namespace where visitors of a booking page
    receive "slot taken" events so the UI can remove it live. Events
    contain no personal data.
-   [ ] Rate limit connections per IP, cap rooms per socket.
-   [ ] **Missed-event recovery:** events carry an increasing ID; on
    reconnect the client sends the last ID and receives what it missed
    (or refetches availability).
-   [ ] Socket.IO Redis adapter so two API instances share events;
    verify by running two instances.

**Done when**

-   Test: client disconnects, bookings happen, client reconnects and
    ends up with correct state.
-   Test with two server instances: event emitted on one reaches
    clients connected to the other.

## Stage 24 --- Search I: Postgres full-text (S)

*Practices: Roadmap 14, 1.*

**Build**

-   [ ] `tsvector` generated columns + GIN index on customers (name,
    email, phone), services (name, description), and bookings (via
    customer name and notes).
-   [ ] `GET /search?q=` returns grouped, ranked results, always scoped
    to the tenant (RLS applies).
-   [ ] Pagination on search results.
-   [ ] Use `EXPLAIN ANALYZE` to prove the index is used.

**Done when**

-   Searching never returns another tenant's rows (leak test).

## Stage 25 --- Search II: fuzzy and Arabic (S)

*Practices: Roadmap 14.*

**Build**

-   [ ] `pg_trgm` for typo-tolerant matching on names and phone
    numbers.
-   [ ] Arabic normalization: remove diacritics (tashkeel), unify
    أ/إ/آ → ا, ى → ي, ة/ه handling, tatweel; Arabic and Latin digits.
    Decide whether this lives in SQL (`unaccent` / custom config) or in
    the app, and write down why.
-   [ ] Transliteration is **out of scope**; document the limits.
-   [ ] Test data set with Arabic and English names.

**Done when**

-   "محمد", "مُحَمَّد" and "محمّد" find the same customer; a one-letter
    typo still finds a name.

## Stage 26 --- File uploads (S)

*Practices: Roadmap 8.*

**Build**

-   [ ] Presigned upload URLs to MinIO (S3-compatible) locally, R2/S3
    in production.
-   [ ] Business logo and service image: validate type and size server
    side; object keys are namespaced by tenant and random.
-   [ ] Ownership check before issuing any URL; uploads never go
    through the API server.
-   [ ] Cleanup job for orphaned uploads.

**Done when**

-   A user from tenant A cannot get an upload or download URL for
    tenant B's objects.

------------------------------------------------------------------------

# Part D --- Production and release

## Stage 27 --- Docker and self-hosting (M)

*Practices: Roadmap 15.*

**Build**

-   [ ] Multi-stage Dockerfile (small image, non-root user).
-   [ ] `docker-compose.prod.yml`: api, worker, postgres, redis,
    reverse proxy.
-   [ ] Migrations run as an explicit step, not inside app startup.
-   [ ] Graceful shutdown (finish requests and jobs on `SIGTERM`).
-   [ ] Basic Linux skills: SSH into the server, read container logs,
    check disk and memory.

**Done when**

-   A fresh machine runs the whole stack from the README with one
    command.

## Stage 28 --- CI/CD and migrations in deploy (S)

*Practices: Roadmap 16.*

**Build**

-   [ ] GitHub Actions: build image, run tests, push image, deploy on
    tag.
-   [ ] Migration strategy: backwards-compatible migrations
    (expand → migrate → contract) so a rollback of the app is safe.
-   [ ] Written rollback procedure and a practice run.

**Done when**

-   A deliberately broken migration is caught in CI before deploy.

## Stage 29 --- Deploy to a real server (M)

*Practices: Roadmap 16.*

**Build**

-   [ ] VPS or a PaaS with a managed Postgres.
-   [ ] Domain, DNS, HTTPS (TLS) through the reverse proxy.
-   [ ] Production environment variables and secrets handled safely.
-   [ ] Stripe in test mode with a real public webhook URL.

**Done when**

-   Someone else can open the public URL and make a booking.

## Stage 30 --- Observability (S)

*Practices: Roadmap 18.*

**Build**

-   [ ] Structured logs shipped somewhere searchable; request ID passes
    through API, jobs, and webhooks.
-   [ ] Sentry for errors (with tenant ID as a tag, no personal data).
-   [ ] `/health` and `/ready` wired to uptime monitoring.
-   [ ] Basic metrics: request rate, latency, queue depth, failed jobs.
-   [ ] `pg_stat_statements` and a review of the 5 slowest queries.

**Done when**

-   You can trace one booking from request to email by its request ID.

## Stage 31 --- Backups, restore and load testing (M)

*Practices: Roadmap 7, 18, 16.*

**Build**

-   [ ] Automated daily Postgres backups to object storage.
-   [ ] **Restore drill:** restore into a new database and verify data.
    A backup you never restored is not a backup.
-   [ ] Connection pooling configured and understood (pool size vs.
    Postgres limits, PgBouncer if needed).
-   [ ] Load test (k6): availability endpoint and the concurrent
    booking test at scale. Find the first bottleneck and fix it.
-   [ ] Write the results in `docs/performance.md`.

**Done when**

-   Restore drill completed and timed; NFR targets from section 4
    verified or consciously revised.

## Stage 32 --- Open-source release (S)

*Practices: all.*

**Build**

-   [ ] README (what it is, screenshot, quick start, self-host guide).
-   [ ] `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`,
    issue and PR templates.
-   [ ] License decision: MIT (maximum adoption) vs. AGPL (prevents
    closed SaaS clones). Write the reasoning.
-   [ ] Demo seed data and a public demo instance (test mode).
-   [ ] `docs/architecture.md` with diagrams: modules, booking flow,
    payment flow, webhook flow.
-   [ ] Tag `v1.0.0`.

------------------------------------------------------------------------

## 5. Failure cases (map to the stages that handle them)

| Failure | Stage |
|---|---|
| Two customers book the same slot at once | 8, 9 |
| A request is submitted twice (double click, retry) | 9, 17 |
| User guesses another tenant's booking or customer ID | 5, 10, 24, 26 |
| Email provider is down | 12 |
| A background job runs more than once | 12, 13 |
| Booking saved but the job is lost | 12 (outbox) |
| Stripe webhook arrives twice, late, or out of order | 18 |
| Customer pays but the server crashes before saving | 19 |
| Customer pays after the hold expired | 19 |
| A tenant's webhook URL is slow, down, or malicious | 21 |
| Socket client reconnects after missing events | 23 |
| Row inserted while a user is paging through a list | 15 |
| A migration fails during deploy | 28 |
| Database must be restored from backup | 31 |
| Traffic spike on the public availability endpoint | 14, 31 |

## 6. Rules to keep yourself honest

1.  **Tests before moving on.** Stages 8, 9, 18, 19 and the leak tests
    in 5, 22, 24, 26 are the ones that teach the most; do not skim them.
2.  **Write the failing test first** for every concurrency or failure
    case. Watching it fail is the lesson.
3.  **One new idea per stage.** If you are stuck for more than a day,
    split the stage.
4.  **AI is a reviewer, not the author.** Write each stage yourself,
    then ask for criticism and for ways to break it.
5.  **Keep the learning log.** For each stage: what I built, what broke,
    what I would do differently.
6.  **Real money only after Part D.** Stay in Stripe test mode until
    security, backups, and monitoring are in place.
