# Full-Stack Engineering Roadmap --- 80/20

**Goal:** Become a capable full-stack engineer as quickly as possible by
focusing on the small set of skills that unlocks most real-world
application development.

**Starting point:** Existing experience with React, React Native,
TypeScript, Expo, and some PostgreSQL/Supabase.

**Learning strategy** - Spend roughly 80% of learning time on practical
skills used in real applications. - Spend roughly 20% on deeper theory
and advanced concepts. - Build one real application throughout the
roadmap. - Learn technologies when they solve a real problem; do not
collect tools for their own sake.

------------------------------------------------------------------------

## Phase 1 --- Backend foundations

*Goal: Understand how the web, servers, programming runtimes, and
databases work.*

### 1. PostgreSQL and SQL

-   [ ] Tables, schemas, and data types
-   [ ] Primary keys and foreign keys
-   [ ] Relationships (1:1, 1:N, N:N)
-   [ ] CRUD operations
-   [ ] JOINs and subqueries
-   [ ] GROUP BY, HAVING, and aggregate functions
-   [ ] CTEs and window functions
-   [ ] Constraints and indexes
-   [ ] Transactions and ACID
-   [ ] Database normalization
-   [ ] Basic query optimization (`EXPLAIN ANALYZE`)
-   [ ] PostgreSQL Row-Level Security (RLS)

**Technology:** PostgreSQL, SQL, Drizzle ORM.

### 2. Backend fundamentals

-   [ ] Client-server architecture
-   [ ] HTTP request/response lifecycle
-   [ ] REST API principles
-   [ ] HTTP methods and status codes
-   [ ] Headers, cookies, and sessions
-   [ ] JSON and serialization
-   [ ] CORS
-   [ ] Environment variables
-   [ ] Error handling
-   [ ] Input validation
-   [ ] Authentication vs. authorization
-   [ ] Middleware

**Technology:** Node.js and TypeScript.

### 3. Node.js and TypeScript

-   [ ] Node.js runtime
-   [ ] Event loop
-   [ ] Async/await and Promises
-   [ ] Error handling
-   [ ] Modules and packages
-   [ ] File system basics
-   [ ] Streams fundamentals
-   [ ] TypeScript generics
-   [ ] Interfaces and utility types
-   [ ] Strict TypeScript

### 4. Backend framework and architecture

-   [ ] NestJS fundamentals
-   [ ] Modules
-   [ ] Controllers
-   [ ] Providers and dependency injection
-   [ ] Services
-   [ ] DTOs
-   [ ] Guards
-   [ ] Pipes
-   [ ] Interceptors
-   [ ] Exception filters
-   [ ] Repository pattern
-   [ ] Modular monolith architecture

**Outcome:** Independently build a backend API.

------------------------------------------------------------------------

## Phase 2 --- Building complete applications

*Goal: Independently implement the essential capabilities of a real
application.*

### 5. Authentication and authorization

-   [ ] Registration and login
-   [ ] Password hashing (Argon2)
-   [ ] Session vs. JWT authentication
-   [ ] Access and refresh tokens
-   [ ] Refresh token rotation
-   [ ] Password reset
-   [ ] Email verification
-   [ ] OAuth (Google and Apple)
-   [ ] Role-Based Access Control (RBAC)
-   [ ] Resource ownership
-   [ ] PostgreSQL RLS

### 6. Application security

-   [ ] Input validation (Zod)
-   [ ] SQL injection prevention
-   [ ] XSS and CSRF fundamentals
-   [ ] CORS configuration
-   [ ] Secure cookies
-   [ ] Rate limiting
-   [ ] Secure HTTP headers
-   [ ] OWASP Top 10
-   [ ] Secrets management
-   [ ] Least-privilege database permissions

### 7. Database engineering

-   [ ] Schema design
-   [ ] Migrations
-   [ ] Transactions
-   [ ] Unique constraints
-   [ ] Foreign keys
-   [ ] Indexes
-   [ ] Concurrent requests
-   [ ] Basic locking (`SELECT FOR UPDATE`)
-   [ ] Pagination
-   [ ] N+1 query problems
-   [ ] Database connection pooling
-   [ ] Backup and restore fundamentals

### 8. API design and integration

-   [ ] REST API design
-   [ ] Pagination, filtering, and sorting
-   [ ] API versioning basics
-   [ ] Swagger / OpenAPI
-   [ ] Third-party API integration
-   [ ] Webhooks
-   [ ] Request validation
-   [ ] Consistent error response conventions
-   [ ] File uploads
-   [ ] Presigned URLs (S3 / R2)

### 9. Testing

-   [ ] Unit testing
-   [ ] Integration testing
-   [ ] End-to-end testing
-   [ ] API testing
-   [ ] Test databases
-   [ ] Authentication and authorization tests
-   [ ] Mocking external services

**Technology:** Vitest or Jest, Supertest, Testcontainers.

------------------------------------------------------------------------

## Phase 2b --- Real-world product features

*Goal: Build the features that paying users expect, and the failure
modes that come with them.*

### 10. Multi-tenancy

-   [ ] Tenant model (organization / workspace) and membership
-   [ ] Shared database with a `tenant_id` column on every tenant table
-   [ ] Tenant isolation with PostgreSQL RLS (`SET LOCAL app.tenant_id`)
-   [ ] Resolving the tenant per request (subdomain, header, or token)
-   [ ] NestJS tenant guard, request-scoped tenant context
-   [ ] Per-tenant roles (owner, admin, member)
-   [ ] Invitations and onboarding flow
-   [ ] Composite unique constraints (unique per tenant, not global)
-   [ ] Cross-tenant leak tests (tenant A must never see tenant B)
-   [ ] Per-tenant limits and plans

### 11. Payments (Stripe)

-   [ ] Test mode vs. live mode and API keys
-   [ ] Customers, products, and prices
-   [ ] Stripe Checkout (hosted page, never touch card data yourself)
-   [ ] One-time payments and subscriptions
-   [ ] Customer portal (cancel, update card)
-   [ ] Payment states (pending, succeeded, failed, refunded)
-   [ ] Idempotency keys on every payment request
-   [ ] Storing amounts as integers (cents), never floats
-   [ ] Syncing subscription status to your database
-   [ ] Handling failed payments and retries
-   [ ] Refunds and disputes basics
-   [ ] Platform fees / Stripe Connect (only if the product needs it)

### 12. Webhooks (receiving and sending)

-   [ ] Receiving: verify the signature on the raw request body
-   [ ] Return 2xx quickly, process asynchronously in a queue
-   [ ] Idempotent handlers (store processed event IDs)
-   [ ] Out-of-order and duplicate events
-   [ ] Retries and dead-letter handling
-   [ ] Local testing (Stripe CLI)
-   [ ] Sending: let tenants register webhook URLs
-   [ ] Signing outgoing payloads, retry with backoff
-   [ ] Delivery log and manual replay

### 13. Realtime and WebSockets

-   [ ] HTTP polling vs. SSE vs. WebSockets (when to use which)
-   [ ] NestJS gateways with Socket.IO
-   [ ] Authenticating socket connections (JWT on handshake)
-   [ ] Rooms and namespaces (one room per tenant / resource)
-   [ ] Authorization on every event, not only on connect
-   [ ] Broadcasting events from services and background jobs
-   [ ] Reconnection and missed-event recovery
-   [ ] Scaling to multiple servers (Redis adapter)
-   [ ] Presence and typing indicators
-   [ ] Realtime updates on the React / Expo client

### 14. Pagination and search

-   [ ] Offset pagination and why it breaks on large or changing data
-   [ ] Cursor (keyset) pagination
-   [ ] Stable sorting (always include a unique tiebreaker column)
-   [ ] Pagination with filters and sorting
-   [ ] Infinite scroll on the frontend (TanStack Query)
-   [ ] PostgreSQL full-text search (`tsvector`, GIN index, ranking)
-   [ ] Fuzzy search with `pg_trgm`
-   [ ] Arabic and multi-language search configuration
-   [ ] Search scoped by tenant
-   [ ] When to move to Meilisearch / Elasticsearch (later, not now)

**Technology:** Stripe, Socket.IO, Redis, PostgreSQL full-text search.

------------------------------------------------------------------------

## Phase 3 --- Production and deployment

*Goal: Make applications reliable, observable, maintainable, and ready
for real users.*

### 15. Docker and Linux

-   [ ] Linux basics
-   [ ] Processes and permissions
-   [ ] Environment variables
-   [ ] SSH
-   [ ] Docker images and containers
-   [ ] Dockerfiles
-   [ ] Docker Compose
-   [ ] Volumes and networking
-   [ ] Container logs

### 16. Deployment and CI/CD

-   [ ] Development vs. production environments
-   [ ] Deploying Node.js APIs
-   [ ] Deploying PostgreSQL
-   [ ] Domain and DNS configuration
-   [ ] HTTPS and TLS
-   [ ] Reverse proxies
-   [ ] GitHub Actions
-   [ ] Automated testing
-   [ ] Database migrations during deployment
-   [ ] Rollbacks

### 17. Background jobs and caching

-   [ ] Background processing
-   [ ] Job queues
-   [ ] Email delivery
-   [ ] Retry strategies
-   [ ] Redis fundamentals
-   [ ] Cache-aside pattern
-   [ ] TTL and cache invalidation
-   [ ] Scheduled jobs
-   [ ] Idempotency basics

**Technology:** BullMQ, Redis, Resend.

### 18. Monitoring and debugging

-   [ ] Structured logging (Pino)
-   [ ] Error tracking (Sentry)
-   [ ] Request IDs
-   [ ] Application health checks
-   [ ] Basic metrics
-   [ ] Production debugging
-   [ ] PostgreSQL performance monitoring

------------------------------------------------------------------------

## Phase 4 --- Additional skills when needed

*Goal: Add depth when the applications you build actually require it.*

### 19. Cloud engineering

-   [ ] AWS fundamentals
-   [ ] S3
-   [ ] EC2 / ECS
-   [ ] Managed PostgreSQL
-   [ ] IAM
-   [ ] Cloud networking
-   [ ] Infrastructure costs

### 20. Advanced backend engineering

-   [ ] Advanced concurrency
-   [ ] Distributed locking
-   [ ] Event-driven architecture
-   [ ] Message brokers (RabbitMQ / Kafka)
-   [ ] Transactional Outbox
-   [ ] Eventual consistency
-   [ ] Microservices
-   [ ] Distributed tracing
-   [ ] Database partitioning
-   [ ] Read replicas

### 21. Advanced DevOps and scalability

-   [ ] Terraform
-   [ ] Kubernetes
-   [ ] Advanced CI/CD
-   [ ] Autoscaling
-   [ ] Load balancing
-   [ ] High availability
-   [ ] Disaster recovery
-   [ ] Distributed systems
-   [ ] Multi-region architecture

------------------------------------------------------------------------

## Recommended technology stack

Choose one primary tool per area. Avoid spending too much time comparing
alternatives.

  Area                Recommended choice
  ------------------- -------------------------
  Language            TypeScript
  Runtime             Node.js
  Backend framework   NestJS
  Database            PostgreSQL
  ORM                 Drizzle
  Cache and jobs      Redis + BullMQ
  Validation          Zod
  API documentation   Swagger / OpenAPI
  Testing             Vitest + Testcontainers
  Logging             Pino
  Error tracking      Sentry
  Observability       OpenTelemetry (later)
  Containers          Docker
  CI/CD               GitHub Actions
  Object storage      S3 / Cloudflare R2
  Payments            Stripe (Checkout + webhooks)
  Realtime            NestJS Gateway + Socket.IO
  Search              PostgreSQL full-text + pg_trgm
  Multi-tenancy       Shared DB + tenant_id + RLS
  Cloud               AWS (learn when needed)

Keep React, Next.js, React Native, and Expo for the frontend.

------------------------------------------------------------------------

## One project: Learn by building

Use the Pledge application as the practical learning project. Build and
deploy the complete system rather than following disconnected tutorials.

### Project implementation

-   [ ] React / Next.js frontend
-   [ ] NestJS backend
-   [ ] PostgreSQL database
-   [ ] Drizzle ORM
-   [ ] Authentication and authorization
-   [ ] Business logic and transactions
-   [ ] Multi-tenant organizations with per-tenant roles and RLS
-   [ ] Stripe payments or subscriptions with webhooks
-   [ ] Realtime updates over WebSockets
-   [ ] Cursor pagination and full-text search
-   [ ] Background jobs and emails
-   [ ] Automated tests
-   [ ] Docker
-   [ ] CI/CD and deployment

### Deliberately handle real-world failure cases

-   [ ] Two users attempt to update the same resource simultaneously.
-   [ ] A request is submitted twice.
-   [ ] An email provider becomes unavailable.
-   [ ] A database migration fails.
-   [ ] A background job is processed more than once.
-   [ ] A user attempts to access another user's data.
-   [ ] The API receives substantially more traffic than usual.
-   [ ] A database needs to be restored from backup.
-   [ ] A Stripe webhook arrives twice, late, or out of order.
-   [ ] A user pays but your server crashes before saving the result.
-   [ ] A user from tenant A guesses a tenant B resource ID.
-   [ ] A WebSocket client reconnects and missed events while offline.
-   [ ] A new row is inserted while a user is paging through a list.
-   [ ] A WebSocket user loses access to a room but stays connected.

Solve these problems instead of implementing only the happy path.

------------------------------------------------------------------------

## Progress milestones

Use demonstrated ability---not tutorials watched or technologies
installed---to measure progress.

### 1. Foundation

-   [ ] Can write meaningful SQL queries.
-   [ ] Can design relational schemas.
-   [ ] Can explain HTTP and the request lifecycle.
-   [ ] Understands Node.js asynchronous execution.

### 2. Full-stack application developer

-   [ ] Can build a frontend connected to an independently developed
    backend.
-   [ ] Can implement authentication and authorization.
-   [ ] Can handle validation, transactions, and errors.
-   [ ] Can write unit and integration tests.
-   [ ] Can isolate tenants and prove it with tests.
-   [ ] Can accept payments and handle webhooks safely.
-   [ ] Can build realtime features, cursor pagination, and search.

### 3. Production-capable developer

-   [ ] Can deploy an application.
-   [ ] Can configure its database and environment.
-   [ ] Can monitor and debug production issues.
-   [ ] Can manage backups and basic recovery.

### 4. Advanced backend engineer

-   [ ] Can reason about concurrency and distributed failures.
-   [ ] Can identify scaling bottlenecks.
-   [ ] Can design reliable asynchronous processing.
-   [ ] Can make informed architecture trade-offs.

------------------------------------------------------------------------

## Study principles

1.  **Learn fundamentals properly.** Prioritize SQL, HTTP, Node.js, and
    database design.
2.  **Build while learning.** Do not wait until the production phase to
    start building applications.
3.  **Use AI as an assistant, not a replacement.** Ask it to review
    architecture, challenge decisions, and create failure scenarios.
4.  **Avoid premature complexity.** Kafka, Kubernetes, microservices,
    and distributed locks are not prerequisites for your first serious
    backend.
5.  **Deploy something real.** A backend that only runs locally does not
    teach you production engineering.
6.  **Learn on demand.** Do not study an advanced technology until you
    understand the problem it solves.

## Target

Aim for approximately **3--5 months** of part-time, focused learning and
implementation (roughly 2--3 months full-time) to become capable of
building and deploying complete full-stack applications, including
payments, realtime, and multi-tenancy. Phases 1--2 give you a working
app in about 7--10 weeks; Phase 2b and Phase 3 take the rest. This is an
intensive target, not a guarantee of mastery. Continue developing depth through real projects and production
experience.

**The objective:** Become someone who can independently design, build,
test, deploy, secure, debug, and maintain a complete application---not
someone who has merely learned a long list of technologies.
