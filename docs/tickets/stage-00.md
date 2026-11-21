# Stage 0 --- Project setup

**Stage goal:** a running NestJS API with a real database, config checks,
logging, health endpoints, API docs, a real-database test harness, and CI.
No business features yet.

**Order matters:** do the tickets in this order.

------------------------------------------------------------------------

### S0-01 The repository exists with a clean Git workflow

**Why:** every later ticket is a branch and a pull request; set the habit
now.
**Do:**

-   Create the repo, `.gitignore` (node_modules, `.env`, dist), a stub
    README, and a PR template listing acceptance items.
-   Agree on commit style (conventional commits: `feat:`, `fix:`,
    `chore:`, `test:`, `docs:`).
-   Protect `main`: no direct pushes, PRs must pass CI (turn this on
    after S0-09).

**Acceptance:**

-   `.env` is ignored; a committed `.env.example` exists (can be empty
    for now).
-   You made one test PR from a branch and merged it.

**Learn:** why feature branches and small PRs reduce risk, even when you
work alone.

------------------------------------------------------------------------

### S0-02 NestJS skeleton with strict TypeScript and linting

**Why:** strict types catch bugs before they run; consistent style keeps
diffs small.
**Do:**

-   Create the NestJS project.
-   Turn on `"strict": true` (plus `noUncheckedIndexedAccess` if you
    want to be stricter).
-   Add ESLint and Prettier, and scripts: `lint`, `format`, `typecheck`,
    `test`, `start:dev`, `build`.

**Acceptance:**

-   `npm run typecheck`, `lint` and `build` all pass on a fresh clone.
-   The default Nest "Hello World" controller is removed.

**Learn:** what NestJS modules, controllers and providers are, in one
sentence each.

------------------------------------------------------------------------

### S0-03 Local infrastructure with Docker Compose

**Why:** everyone (and CI) runs the same Postgres, Redis and mail catcher.
**Do:**

-   `docker-compose.yml` with Postgres, Redis, and Mailpit (catches
    emails, web UI on a local port).
-   Named volumes for Postgres data, healthchecks on each service.
-   `.env.example` with `DATABASE_URL`, `REDIS_URL`, `SMTP_HOST`,
    `SMTP_PORT`.

**Acceptance:**

-   `docker compose up -d` starts all three and they report healthy.
-   You can connect to Postgres with `psql` or a GUI client.
-   Removing the containers does not delete data (volume), and
    `docker compose down -v` does.

**Learn:** difference between an image, a container and a volume.

------------------------------------------------------------------------

### S0-04 Configuration is validated at startup

**Why:** a missing or wrong environment variable should fail on boot, not
at 3 AM in production.
**Do:**

-   Config module that reads environment variables and validates them
    with a Zod schema (types, required fields, defaults, `NODE_ENV`).
-   A typed config service the rest of the app uses; no direct
    `process.env` access elsewhere.

**Acceptance:**

-   Starting with `DATABASE_URL` missing stops the app with a clear
    message naming the variable.
-   No secret values are printed in logs.

**Test first:** a unit test that feeds the schema an invalid object and
expects a descriptive failure.
**Learn:** why config comes from the environment (twelve-factor idea) and
what must never be committed.

------------------------------------------------------------------------

### S0-05 Structured logging with a request ID

**Why:** you cannot debug what you cannot trace; the request ID links
every log line of one request.
**Do:**

-   Pino logger integrated with Nest.
-   Middleware: use the incoming `X-Request-Id` if present, otherwise
    generate one; return it in the response header; include it in every
    log line.
-   Pretty logs in development, JSON in production.
-   Redact `authorization` and `cookie` headers.

**Acceptance:**

-   Two requests produce log lines with different request IDs.
-   Sending `X-Request-Id: abc` returns `abc` and logs it.

**Learn:** why structured (JSON) logs are better than text logs, and what
should never be logged.

------------------------------------------------------------------------

### S0-06 Integration test harness on a real database

**Why:** mocking the database hides real bugs (constraints, transactions,
RLS). You will test against real Postgres from now on.
**Do:**

-   Vitest + Supertest + Testcontainers.
-   A helper that starts one Postgres container for the test run,
    creates the Nest app against it, and tears everything down.
-   Separate scripts: `test:unit` (fast) and `test:int` (uses
    containers).

**Acceptance:**

-   `npm run test:int` starts a throwaway Postgres, runs a trivial query
    test, and removes the container.
-   Runs without touching your development database.

**Learn:** the difference between unit, integration and end-to-end tests,
and why this project favors integration tests.

------------------------------------------------------------------------

### S0-07 Health and readiness endpoints

**Why:** orchestrators and monitors need to know if the app is alive and
if it can serve traffic.
**Do:**

-   `GET /health`: always 200 if the process runs (liveness).
-   `GET /ready`: 200 only if the database answers a `SELECT 1`; 503
    otherwise (readiness).

**Test first:** an integration test for `/ready` with the database up,
and one with the database connection broken (expect 503).
**Acceptance:**

-   Both tests pass.
-   Responses contain no internal details (no connection strings).

**Learn:** liveness vs. readiness, and what an orchestrator does with
each.

------------------------------------------------------------------------

### S0-08 API docs at /docs

**Why:** OpenAPI is your test client now and the contract for everyone
later.
**Do:**

-   Swagger module serving UI at `/docs` and JSON at `/docs-json`.
-   Tag health endpoints.

**Acceptance:**

-   `/docs` lists `/health` and `/ready` and "Try it out" works.

**Learn:** what OpenAPI is and why generated docs can drift from reality
if you are not careful.

------------------------------------------------------------------------

### S0-09 Continuous integration

**Why:** the machine should run lint, types and tests on every push, so
`main` stays green.
**Do:**

-   GitHub Actions workflow: install (with cache), lint, typecheck,
    unit tests, integration tests (Docker is available on GitHub-hosted
    runners).
-   Run on pull requests and on pushes to `main`.
-   Then turn on branch protection from S0-01.

**Acceptance:**

-   A PR with a deliberate type error fails CI; fixing it turns CI
    green.
-   Total CI time is written in the PR description.

**Learn:** what CI catches that local tests do not (clean environment,
forgotten files).

------------------------------------------------------------------------

## Stage 0 done when

-   `docker compose up -d` + `npm run start:dev` gives a running API.
-   `/ready` is covered by an integration test against Testcontainers.
-   CI is green on `main`.
-   `docs/learning-log.md` has the Stage 0 entry.
