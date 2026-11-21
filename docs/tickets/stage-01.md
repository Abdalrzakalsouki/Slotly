# Stage 1 --- Schema foundations and migrations

**Stage goal:** the first three tables (`users`, `tenants`, `memberships`)
created through committed migrations, with a seed script, a thin
repository layer, and SQL you wrote by hand. No endpoints yet.

**Order matters:** do the tickets in this order.

------------------------------------------------------------------------

### S1-01 Drizzle connected, with a migration workflow

**Why:** the schema must live in git and be reproducible from an empty
database. Nothing is changed by hand in a database GUI.
**Do:**

-   Install Drizzle and its migration tool; database module that
    provides one connection pool built from the validated config.
-   Scripts: `db:generate` (create a migration from schema changes),
    `db:migrate` (apply), and a way to reset the local database.
-   Commit the generated SQL migration files; read each one before
    committing.

**Acceptance:**

-   From an empty Postgres, `db:migrate` brings the schema up with no
    manual steps.
-   Integration test harness (S0-06) applies migrations to its
    Testcontainers database before tests.

**Learn:** what a migration is, why you never edit an applied one, and
what the pool does.

------------------------------------------------------------------------

### S1-02 `users` table

**Why:** identity is the base of everything else.
**Do:**

-   Columns: `id`, `email`, `password_hash` (nullable for future OAuth),
    `display_name`, `email_verified_at` (nullable), `created_at`,
    `updated_at`.
-   Unique index on **lower(email)**, so `A@x.com` and `a@x.com` cannot
    both exist.
-   Decide the primary key type: `uuid` (v7 is time-sortable, which
    helps cursor pagination later) or `bigint` identity. Write the
    decision and reasons in `docs/learning-log.md`.

**Test first:** an integration test inserting two users whose emails
differ only by case; expect a unique violation.
**Acceptance:**

-   The test passes; the error is recognized in code (Postgres error
    code `23505`), not matched on message text.
-   `updated_at` changes on update (trigger or application rule; pick
    one and note why).

**Learn:** primary keys, unique constraints, and why `NOT NULL` and
constraints belong in the database, not only in code.

------------------------------------------------------------------------

### S1-03 `tenants` table

**Why:** a tenant is one business using the platform.
**Do:**

-   Columns: `id`, `name`, `slug`, `timezone` (IANA name),
    `currency` (3-letter code), `default_language`, timestamps.
-   Unique `slug`; check constraint for slug format (lowercase letters,
    digits, hyphens) and for currency length.

**Test first:** inserting a slug with spaces or capitals fails; a
duplicate slug fails.
**Acceptance:**

-   Constraint violations are covered by tests.
-   Timezone validity is enforced somewhere (a check against
    `pg_timezone_names`, or application validation; write down which
    and why).

**Learn:** check constraints, and the idea of enforcing invariants at the
lowest layer that can.

------------------------------------------------------------------------

### S1-04 `memberships` table (users in tenants with a role)

**Why:** a user can belong to many businesses with a different role in
each; this is the many-to-many link.
**Do:**

-   Columns: `id`, `user_id`, `tenant_id`, `role`
    (`owner` | `admin` | `staff`), `created_at`.
-   Foreign keys to `users` and `tenants`; unique `(tenant_id, user_id)`.
-   Decide `ON DELETE` behavior for each foreign key (restrict, cascade,
    or set null) and write down why.
-   Index on `user_id` (find a user's tenants) and think about whether
    `(tenant_id, user_id)` already serves the other direction.

**Test first:** adding the same user twice to the same tenant fails;
deleting a tenant with memberships behaves as you decided.
**Acceptance:**

-   Tests for both pass.
-   `role` is a Postgres enum or a check constraint (decide which and
    note the trade-off: enums are awkward to change later).

**Learn:** the three relationship types (1:1, 1:N, N:N) and how a join
table implements N:N.

------------------------------------------------------------------------

### S1-05 Idempotent seed script

**Why:** you need quick realistic data for development and demos, and the
script must be safe to run twice.
**Do:**

-   Seed: 2 tenants, 5 users, memberships with different roles
    (one user in both tenants).
-   Re-running the script does not create duplicates or crash
    (`ON CONFLICT DO NOTHING` or equivalent).
-   Seed passwords are fake and documented as development only.

**Acceptance:**

-   Running the seed twice leaves the same row counts.

**Learn:** what "idempotent" means, in your own words.

------------------------------------------------------------------------

### S1-06 Thin repositories with constraint tests

**Why:** one place for queries keeps the rest of the app clean, and
tests prove the database rules hold through the code path you actually
use.
**Do:**

-   `UsersRepository`, `TenantsRepository`, `MembershipsRepository` with
    only data access (find by id/email/slug, create, list a user's
    tenants, list a tenant's members). No business rules here.
-   Map Postgres constraint errors to small typed errors (e.g.
    `UniqueViolationError`), not strings.

**Acceptance:**

-   Integration tests cover every method, including the failure paths.
-   No repository imports anything from NestJS controllers or services.

**Learn:** why the repository pattern exists and what it costs.

------------------------------------------------------------------------

### S1-07 Ten hand-written SQL queries

**Why:** an ORM is easy to use badly if you cannot read the SQL it
generates. Writing raw SQL first builds the intuition.
**Do:** write each query by hand in `docs/sql-practice.sql` against the
seeded database, with a one-line comment on what it answers:

1.  Tenants a given user belongs to, with their role (JOIN).
2.  Number of members per tenant (GROUP BY).
3.  Tenants with more than one member (HAVING).
4.  Users who belong to no tenant (LEFT JOIN ... IS NULL).
5.  Tenants that have at least one owner (EXISTS subquery).
6.  Latest membership created per user (CTE).
7.  Members of each tenant numbered by join date
    (`row_number() OVER (PARTITION BY ...)`).
8.  The first member of each tenant (window function + filter).
9.  Create a tenant and its owner membership in **one transaction**
    (`BEGIN ... COMMIT`), then show that an error in the second insert
    rolls back the first.
10. Look up a tenant by slug: run `EXPLAIN ANALYZE` with and without the
    unique index and compare the plans.

**Acceptance:**

-   All ten run against the seeded database and give expected results.
-   For query 10 you can explain "Seq Scan" vs. "Index Scan" in one
    sentence.

**Learn:** JOINs, aggregates, CTEs, window functions, transactions, and
reading a query plan.

------------------------------------------------------------------------

## Stage 1 done when

-   Migrations run from an empty database, locally and in CI.
-   You can explain every constraint and the error it produces.
-   All repository tests pass in CI.
-   `docs/sql-practice.sql` has ten working queries.
-   `docs/learning-log.md` has the Stage 1 entry, including the primary
    key and `ON DELETE` decisions.
