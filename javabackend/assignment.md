# Java Backend Developer — Hiring Assignment
### Role: Java Backend Developer (1-2 years work experience)
### Time Limit: 24 hours from receipt

---

## Context

You're joining a company that builds HR technology for the construction industry. The users are site managers, HR teams, and payroll operators managing blue-collar workforce — daily wage workers, shift-based crews, overtime-heavy schedules.

This assignment simulates your actual job. You will work with an existing open-source HRMS codebase — not build from scratch. This tests what matters to us: can you read someone else's code, design a solid schema, write clean APIs, and handle the backend complexity that real workforce management demands?

## Guidelines
1- You are encouraged to use AI Tools to complete this assignment.
2- You are expected to be able to explain the code and make changes to your code during the interview.
3- We care about your backend decisions — schema design, caching strategy, data integrity. A working API with proper validation matters more than a pretty UI.

---

## Setup

Pick any open-source HRMS or Employee Management System built with **Java + Spring Boot**. Fork it. Get it running locally with the following stack:

- **Java 17+** with **Spring Boot**
- **Hibernate/JPA** for ORM
- **Supabase** as your PostgreSQL database (free tier is fine — create a project at [supabase.com](https://supabase.com))
- **Redis** for caching (local Redis or a free cloud instance)

**Suggested repo:** [SpringBoot Employee Management System](https://github.com/amigoscode/spring-boot-fullstack-professional) — Spring Boot + JPA + PostgreSQL. Clean structure, good starting point.

You may use a different Java HRMS if you prefer — the tasks below are designed to work with any system that has employee/payroll functionality. If the repo you pick uses a different database, migrate it to Supabase (PostgreSQL).

---

## Part 1: Feature Build — Worker Attendance & Overtime Settlement Engine

**Scenario:** A construction company client needs to track daily attendance for site workers — clock-in, clock-out, automatic overtime calculation, and monthly settlement reports. Site supervisors need to know who's currently on-site in real-time, and payroll needs accurate overtime numbers at month-end.

This is the core backend feature. Build the APIs, schema, caching layer, and business logic.

---

### 1. Schema Design (Hibernate Entities)

Design and implement the following entities with proper JPA relationships and constraints:

- **Worker** — name, phone, designation (enum: `MASON`, `ELECTRICIAN`, `PLUMBER`, `SUPERVISOR`, `HELPER`), daily wage rate, active status
- **Site** — site name, location, active status
- **AttendanceLog** — worker reference, site reference, clock-in timestamp, clock-out timestamp, total hours worked, overtime hours (calculated)
- **OvertimeEntry** — worker reference, attendance reference, date, overtime hours, overtime rate applied, amount, settlement status (enum: `PENDING`, `SETTLED`)

Use proper `@Table`, `@Column`, `@Index` annotations. Constraints should enforce business rules at the database level — not just in application code.

---

### 2. REST APIs

Build the following endpoints:

**Attendance**
- `POST /api/attendance/clock-in` — Log a worker's arrival at a site. Request body: `{ workerId, siteId }`
- `POST /api/attendance/clock-out` — Log a worker's departure. Request body: `{ workerId }`. System calculates total hours and overtime automatically.
- `GET /api/attendance/active` — List all workers currently clocked in across all sites. **This must be served from Redis, not the database.**
- `GET /api/attendance/log?workerId={id}&from={date}&to={date}` — Attendance history for a worker in a date range. Supports pagination.

**Overtime**
- `GET /api/overtime/summary/{workerId}?month={YYYY-MM}` — Monthly overtime summary: total overtime hours, breakdown by date, total payout amount, settlement status.
- `POST /api/overtime/settle/{workerId}?month={YYYY-MM}` — Mark all overtime entries for that worker+month as `SETTLED`. Cannot settle the current month (only past months).

---

### 3. Redis Caching

- **Active workers cache** — When a worker clocks in, add them to a Redis hash/set with their worker ID, site info, and clock-in time. Remove on clock-out. The `GET /active` endpoint reads exclusively from Redis.
- **TTL safety net** — Set a TTL of 16 hours on each active worker entry. No one stays clocked in for more than 16 hours — if the clock-out was missed, the entry should expire and be flagged.
- **Cache invalidation** — When a worker's profile is updated (name, designation, wage rate), invalidate any cached entries that contain stale data.

---

### 4. Business Rules

These are the rules the system must enforce. We will test edge cases during review.

**Clock-in rules:**
- Worker must exist and be active
- Worker cannot clock in if already clocked in (no double entry)
- Clock-in time cannot be in the future
- Site must exist and be active

**Clock-out rules:**
- Worker must be currently clocked in (cannot clock out without clocking in)
- If total shift exceeds 16 hours, auto-flag the attendance record for review (add a `flagged` boolean)

**Overtime calculation:**
- Standard shift = 8 hours. Anything beyond 8 hours in a single attendance record = overtime
- Overtime rate: **1.5x** daily wage rate for the first 2 overtime hours, **2x** beyond that
- Monthly overtime cap: **60 hours** per worker. If a clock-out would push a worker past 60 hours for the month, still record the attendance but cap the overtime entry at whatever hours remain under 60

**Settlement rules:**
- Cannot settle current month — only completed months
- Once settled, overtime entries cannot be modified
- Settlement endpoint should return the total amount in the response

---

### 5. Error Handling

- All validation failures return structured JSON error responses — not stack traces, not generic 500s
- Use proper HTTP status codes: `400` for validation failures, `404` for not found, `409` for conflicts (duplicate clock-in, already settled)
- Error response format:
```json
{
  "error": "DUPLICATE_CLOCK_IN",
  "message": "Worker is already clocked in at Site: Greenfield Phase 2",
  "timestamp": "2026-05-25T10:30:00Z"
}
```

---

## Part 2: Ticket Blitz (5 Backend Tickets — Fast Response)

**Scenario:** You've just joined the team. It's your first week. Five tickets come in — each is a real production issue or client request. These aren't single-line fixes. Each one requires you to understand how multiple layers of the application connect — config, service, entity, repository. You'll need to trace the problem across files, not just patch the symptom.

For each ticket: fix the issue, commit with a clear message referencing the ticket number.

---

**TICKET LF-201: API calls from frontend blocked by CORS**

"The frontend team says every API call from the React app returns a CORS error in the browser. The endpoints work fine when tested from Postman. We launched the frontend on `localhost:3000` and the backend runs on `localhost:8080`."

*What's broken:* There's no CORS configuration — or it's incomplete. This isn't a one-line fix.

*Where to look:*
- Spring Security filter chain — CORS must be processed **before** Spring Security rejects the preflight `OPTIONS` request. If security blocks the preflight, no amount of `@CrossOrigin` helps.
- A `CorsConfigurationSource` bean or `WebMvcConfigurer` implementation — allowed origins, methods, and headers need to be explicitly registered.
- `application.yml` — hardcoded allowed origins don't survive across environments. Externalize them as config properties so dev/staging/prod each have their own.

*Deliverable:* Frontend at `localhost:3000` can make API calls without CORS errors. Allowed origins are configurable per environment, not hardcoded.

---

**TICKET LF-202: App crashes on startup when Redis is unavailable**

"Our Redis instance went down for maintenance last night. The entire backend refused to start — threw a `RedisConnectionException` on boot and the health check API went offline. The app should still work without caching — just slower."

*What's broken:* The app has a hard dependency on Redis at startup. No fallback, no graceful degradation.

*Where to look:*
- `application.yml` — Redis connection timeout is probably at default (infinite wait). Set a reasonable connect timeout so the app doesn't hang.
- `RedisConfig.java` (or wherever the Redis beans are configured) — the `RedisTemplate` and `CacheManager` beans fail hard if Redis isn't reachable. These need to handle connection failure gracefully.
- Create a custom `CacheErrorHandler` (implement `org.springframework.cache.interceptor.CacheErrorHandler`) — this tells Spring what to do when a cache read/write fails at runtime. Without it, a Redis timeout mid-request crashes the endpoint.
- Service layer — anywhere you use `@Cacheable`, `@CacheEvict`, or manual Redis calls, verify it degrades to DB-only when cache is unavailable.

*Deliverable:* App starts and serves requests even when Redis is completely offline. When Redis comes back, caching resumes automatically. No manual restart needed.

---

**TICKET LF-203: Attendance history endpoint returns the entire table — and it's slow even with fewer records**

"The attendance log page takes 40+ seconds to load. I checked the API directly — `/api/attendance/log` is returning every single record in the database. We have 50,000+ attendance entries and it's dumping all of them in one response. Even when I filter to a single worker for one week (only ~6 records), it still takes 3 seconds. Something else is wrong too."

*What's broken:* Two problems compounding each other. First — no pagination, no default limit, the endpoint dumps the entire table. Second — every attendance record triggers separate queries to load the associated Worker and Site entities. With 50,000 attendance records, that's 100,000+ additional queries hitting the database. This is the N+1 query problem, and pagination alone won't fix it.

*Where to look:*
- `AttendanceLog` entity — check the `@ManyToOne` relationships to Worker and Site. If they're using the default fetch type (`EAGER`), Hibernate loads every related entity immediately. If they're `LAZY`, the N+1 happens when the response serializer (Jackson) accesses the fields. Either way, individual queries per record.
- `AttendanceRepository` — the query method is likely `findAll()` with no `Pageable` parameter. Add pagination support. But also — add an `@EntityGraph` or rewrite the query with `JOIN FETCH` so Worker and Site are loaded in the same query, not as separate round-trips.
- `AttendanceService` — accept and pass through pagination parameters. Don't just add Pageable to the repo and ignore it in the service.
- `AttendanceController` — accept `page` and `size` as query parameters with sensible defaults (page 0, size 20).
- `application.yml` — configure `spring.data.web.pageable.default-page-size` and `max-page-size` as a safety net.
- Response structure — don't return a raw `List<>`. Return a response object with `content`, `totalElements`, `totalPages`, `currentPage`.

*Deliverable:* Endpoint returns paginated results by default with pagination metadata. The N+1 is eliminated — verify by enabling Hibernate SQL logging (`spring.jpa.show-sql=true`) and confirming the attendance query uses joins, not separate selects per record. The old unparameterized call still works but returns only the first page.

---

**TICKET LF-204: Overtime settlement saves partial data — and workers get wrong SMS notifications**

"We settled overtime for a worker who had 22 overtime entries in March. The settlement failed on entry #15 (bad data). But entries 1-14 are already marked SETTLED in the database and the worker already received an SMS saying 'Your March overtime of ₹4,200 has been settled.' The actual amount is wrong because 7 entries are still PENDING. Now we can't re-run settlement because it skips the already-settled ones, and the worker thinks they're getting paid."

*What's broken:* Two problems layered together. First — the settlement logic processes entries in a loop and commits each one individually. No transaction wraps the entire batch, so a failure partway through leaves the data in a half-settled state. Second — the settlement method sends an SMS notification to the worker **inside** the same transaction. So even if you fix the transaction to be atomic and it rolls back on failure, workers #1-14 already received SMS messages that are now lies.

*Where to look:*
- `OvertimeService` (settlement method) — check how `@Transactional` is applied. If it's on individual entry updates inside a loop, or missing entirely, the batch isn't atomic. The **entire settlement** for a worker+month must succeed or fail as one transaction.
- Check for `@Transactional` on the **calling** method vs the **inner** method. If the controller calls a non-transactional method that internally calls a transactional one on the **same bean**, Spring's proxy doesn't intercept it — the transaction annotation is silently ignored. This is the most common `@Transactional` mistake in Spring.
- The notification call (SMS/email to the worker) must not happen inside the transaction. If the DB work succeeds but the SMS API is down, the transaction should still commit. If the DB work fails and rolls back, the SMS should never have been sent. These are two separate concerns that must be decoupled.
- Use `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` or a Spring event publisher to fire the notification **only after the transaction successfully commits**. This requires creating an event class, a listener class, and publishing the event from the service — it's not a one-file fix.
- Handle notification failures independently — if the SMS fails after a successful settlement, log it and queue a retry. Don't crash the settlement response.

*Deliverable:* Settlement is atomic — all entries for a worker+month either settle together or none do. SMS notification is sent **only after** a successful commit, never during or before. If the notification fails, settlement data is still correct and the failure is logged. No partial state, no premature notifications.

---

**TICKET LF-205: App works locally but connections exhaust on staging under moderate traffic**

"The app runs fine on my machine for hours. On the staging server (connected to Supabase), two things happen: (1) After about 30 minutes of idle time, the API returns `SQLTransientConnectionException: Connection is not available`. Restarting fixes it temporarily. (2) Even when it's not idle, if 15-20 users hit the overtime summary endpoint at the same time, the app freezes for 10+ seconds and some requests timeout. We only have 20 concurrent users on staging — this shouldn't be a problem."

*What's broken:* Two problems, one in config and one in code. The idle connection drops happen because Supabase silently kills idle connections but HikariCP doesn't know — it hands out dead connections from the pool. The concurrency freezes happen because the overtime summary service method runs inside `@Transactional` and partway through, it makes a synchronous REST call to an external government API to fetch the latest minimum wage rates for overtime calculation. That external call takes 3-5 seconds. The entire time, the database connection is checked out from the pool, doing nothing. With a default pool of 10 connections and 20 concurrent requests, the pool exhausts in seconds.

*Where to look:*
- `OvertimeService` (or wherever the summary is calculated) — find the external API call that happens inside a `@Transactional` method. The fix is architectural: fetch the external data **before** opening the transaction, then pass it into the transactional method as a parameter. Don't hold a DB connection hostage while waiting on network I/O.
- `application.yml` — HikariCP settings need tuning for a cloud database:
  - `spring.datasource.hikari.max-lifetime` — must be shorter than Supabase's idle timeout (typically 300 seconds / 5 minutes)
  - `spring.datasource.hikari.keepalive-time` — send a keepalive query so Supabase doesn't kill idle connections
  - `spring.datasource.hikari.connection-timeout` and `maximum-pool-size` — right-size for your traffic
- `application-staging.yml` (or equivalent profile) — these settings should be environment-specific. Local DB doesn't need keepalive. Staging and prod do. If there's no profile separation, create it.
- Validate the Supabase connection string uses the **connection pooler** URL (port 6543 with PgBouncer mode), not the direct connection (port 5432) — a common Supabase-specific mistake.
- The external API client (`RestTemplate` / `WebClient`) — configure its own connection and read timeouts so a slow government API doesn't cascade into a full app freeze.

*Deliverable:* App stays connected on staging for 24+ hours without restarts. 20 concurrent requests to the overtime summary endpoint complete without pool exhaustion. Connection pool settings are environment-specific. The external API call no longer holds a database connection. README documents the Supabase connection setup.

---

## Evaluation Criteria

| What we evaluate | What we're looking for |
|---|---|
| **Schema design** | Normalized tables, proper relationships, constraints that enforce business rules at the DB level — not just in Java code |
| **Hibernate fluency** | Correct fetch strategies, no N+1 queries, proper use of `@Transactional`, lazy vs. eager loading decisions that make sense |
| **Redis usage** | Caching the right things — not everything. Proper invalidation. Graceful degradation when Redis is down |
| **Config awareness** | Environment-specific properties, externalized secrets, connection pool tuning — not hardcoded values that only work on your laptop |
| **Multi-layer debugging** | Tickets require tracing problems across config → security → service → repository. We check if you fixed the root cause or just the symptom |
| **API design** | RESTful conventions, correct HTTP status codes, consistent error responses, pagination where it matters |
| **Separation of concerns** | Transactions don't contain side effects. Network I/O doesn't hold DB connections. The candidate knows where one layer's job ends and another begins |
| **Business logic correctness** | Overtime calculation is accurate, edge cases handled (cap at 60, 16-hour flag, no double clock-in), transactions are atomic |
| **Code reading ability** | You worked within the existing codebase patterns. You didn't rewrite the project — you extended it |
| **AI usage** | We expect you to use AI tools (Claude, Copilot, etc.) — tell us which ones and how. No penalty for using AI. Penalty for not using it |

---

## Submission

1. Push your fork to a public GitHub repo
2. README must include:
   - Setup instructions (we will run it locally — include Supabase connection setup steps)
   - Which HRMS you forked and why (one line)
   - Which AI tools you used and for what
   - Any design decisions you'd call out — schema tradeoffs, caching choices, things you'd do differently with more time
3. Commits should be atomic — one per ticket (LF-201 through LF-205), separate commits for the attendance feature (schema, APIs, Redis layer, business logic)
4. Include a Postman collection or `curl` examples for all endpoints
5. Send the repo link within 24 hours

---

*Assignment designed by DeepThought — May 2026*
