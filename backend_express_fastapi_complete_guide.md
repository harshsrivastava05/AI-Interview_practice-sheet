# Backend Interview Prep Complete Guide: Express.js, FastAPI, and Production Backend Layers

> Scope: Backend fundamentals, production architecture layers, Express.js, FastAPI, API design, databases, authentication, security, testing, deployment, queues, caching, observability, and common interview questions.

## How to use this guide

Read this guide in three passes:

1. **Architecture pass:** Understand backend layers and why they exist.
2. **Framework pass:** Learn how the same ideas are implemented in Express.js and FastAPI.
3. **Interview pass:** Practice the questions under each topic and explain the tradeoffs clearly.

A good backend interview answer should usually include:

- What the concept is.
- Why it exists.
- How it is implemented in code.
- How it behaves in production.
- What failure modes or tradeoffs matter.

---

# 1. Backend fundamentals

## 1.1 What is a backend?

### Definition and info

A backend is the server-side part of an application. It receives requests from clients, validates input, enforces authentication and authorization, runs business logic, talks to databases or external services, and returns responses.

In a typical web product, the backend owns:

- APIs.
- Authentication and sessions.
- Business rules.
- Database reads and writes.
- Background jobs.
- Integrations with third-party services.
- Security boundaries.
- Observability and operational behavior.

The frontend should not directly own sensitive logic because browser code is visible and modifiable by users.

### Use case

A backend is needed when an application needs persisted data, user accounts, payments, notifications, file storage, recommendations, permissions, analytics, or integration with external systems.

### Production use case

In production, a backend must be reliable under concurrency, validate every input, handle partial failures, manage secrets safely, expose metrics/logs/traces, and degrade gracefully when dependencies fail.

### Example

**Express.js**

```ts
import express from "express";

const app = express();
app.use(express.json());

app.get("/health", (_req, res) => {
  res.json({ status: "ok" });
});

app.listen(3000, () => console.log("API running on port 3000"));
```

**FastAPI**

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}
```

### Most asked interview questions

**Q1. What does a backend do?**  
A backend handles request processing, validation, authentication, authorization, business logic, persistence, integrations, and response generation.

**Q2. Why should business logic not live only in the frontend?**  
Frontend code can be inspected and manipulated. Backend logic is required for trusted validation, authorization, and data integrity.

**Q3. What makes a backend production-ready?**  
Input validation, secure auth, structured errors, database transaction safety, logging, metrics, rate limits, retries, health checks, graceful shutdown, and deployment automation.

---

# 2. Backend layered architecture

Backend systems are easier to maintain when split into clear layers. Not every small project needs every layer as a separate folder, but the responsibilities should still be separated mentally.

## 2.1 Request/transport layer

### Definition and info

The transport layer receives incoming communication. For web APIs, this is usually HTTP. Other transports include WebSockets, gRPC, message queues, and event streams.

This layer should understand protocol-level details:

- HTTP methods.
- Headers.
- Cookies.
- Status codes.
- Request body.
- Query parameters.
- Path parameters.
- Content type.

It should not contain core business logic.

### Use case

Use this layer to map incoming requests to route handlers/controllers.

### Production use case

In production, the transport layer often sits behind a reverse proxy, load balancer, API gateway, or CDN. It must handle timeouts, request size limits, TLS termination, CORS, and routing.

### Example

**Express.js**

```ts
app.post("/api/users", createUserController);
```

**FastAPI**

```py
@app.post("/api/users")
def create_user():
    return {"ok": True}
```

### Interview questions

**Q1. What is the transport layer responsible for?**  
It accepts requests and maps protocol-level information to application handlers.

**Q2. Should business logic live in route handlers?**  
No. Route handlers should be thin. They should call service/domain logic.

---

## 2.2 Routing layer

### Definition and info

The routing layer maps URLs and HTTP methods to handler functions.

Examples:

- `GET /users/:id` fetches a user.
- `POST /users` creates a user.
- `PATCH /users/:id` partially updates a user.
- `DELETE /users/:id` deletes a user.

### Use case

Routing creates the public API surface of your backend.

### Production use case

In production, routes should be versioned and grouped by domain. For example:

```txt
/api/v1/auth
/api/v1/users
/api/v1/billing
/api/v1/recommendations
```

### Example

**Express.js**

```ts
import { Router } from "express";

const router = Router();

router.get("/:id", getUserController);
router.post("/", createUserController);

app.use("/api/v1/users", router);
```

**FastAPI**

```py
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.get("/{user_id}")
def get_user(user_id: int):
    return {"id": user_id}

app.include_router(router)
```

### Interview questions

**Q1. How do you organize routes in a large backend?**  
By feature/domain using routers/modules, such as auth, users, billing, admin, notifications, and analytics.

**Q2. What is the difference between route params and query params?**  
Route params identify a resource path, such as `/users/123`. Query params filter, sort, or paginate resources, such as `/users?limit=20&page=2`.

---

## 2.3 Controller/handler layer

### Definition and info

A controller or handler receives the parsed request, calls validation and service logic, and returns a response.

A controller should usually:

- Read request values.
- Call a service.
- Convert service output into HTTP response.
- Avoid direct database logic unless the app is very small.

### Use case

Controllers keep API-specific concerns separate from business logic.

### Production use case

Thin controllers make APIs easier to test and allow the same service to be reused by REST, GraphQL, CLI jobs, workers, or cron tasks.

### Example

**Express.js**

```ts
export async function createUserController(req, res, next) {
  try {
    const user = await userService.createUser(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
}
```

**FastAPI**

```py
@router.post("/", status_code=201)
def create_user(payload: CreateUserRequest):
    return user_service.create_user(payload)
```

### Interview questions

**Q1. What should not be inside a controller?**  
Complex business rules, raw SQL scattered everywhere, payment logic, permission logic duplicated across handlers, and external service orchestration.

**Q2. Why keep controllers thin?**  
Thin controllers are easier to test, reuse, refactor, and reason about.

---

## 2.4 Validation/schema/DTO layer

### Definition and info

The validation layer verifies incoming data before it reaches business logic. DTO means Data Transfer Object: a typed structure representing allowed input/output.

Validation checks:

- Required fields.
- Data types.
- String length.
- Numeric ranges.
- Enum values.
- Nested object shape.
- Cross-field constraints.
- File size/type.

### Use case

Prevents invalid or malicious data from entering the service layer.

### Production use case

Validation protects databases, prevents runtime errors, improves API error messages, and reduces security risk.

### Example

**Express.js with Zod**

```ts
import { z } from "zod";

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(1).max(100),
});

app.post("/users", async (req, res, next) => {
  const parsed = createUserSchema.safeParse(req.body);

  if (!parsed.success) {
    return res.status(400).json({ errors: parsed.error.flatten() });
  }

  const user = await userService.createUser(parsed.data);
  res.status(201).json(user);
});
```

**FastAPI with Pydantic**

```py
from pydantic import BaseModel, EmailStr, Field

class CreateUserRequest(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)
    name: str = Field(min_length=1, max_length=100)

@app.post("/users")
def create_user(payload: CreateUserRequest):
    return user_service.create_user(payload)
```

### Interview questions

**Q1. Why validate input if the frontend already validates it?**  
Frontend validation is only for UX. Backend validation is the real security and data-integrity boundary.

**Q2. What is DTO mapping?**  
DTO mapping converts raw input or database models into controlled request/response shapes.

**Q3. Why should response models avoid exposing full database entities?**  
Database entities can contain sensitive fields like password hashes, internal IDs, flags, or audit metadata.

---

## 2.5 Middleware layer

### Definition and info

Middleware runs before or after route handlers. It can inspect or modify requests/responses, enforce auth, log requests, parse JSON, handle CORS, rate limit, or catch errors.

### Use case

Use middleware for cross-cutting concerns that apply to many routes.

### Production use case

Production middleware commonly includes:

- Request ID injection.
- JSON body parsing.
- CORS.
- Authentication.
- Rate limiting.
- Logging.
- Security headers.
- Compression.
- Error handling.

### Example

**Express.js**

```ts
function requestLogger(req, _res, next) {
  console.log(`${req.method} ${req.path}`);
  next();
}

app.use(requestLogger);
```

**FastAPI**

```py
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.time() - start)
    return response
```

### Interview questions

**Q1. What is middleware?**  
Middleware is code that runs around route handling and is used for reusable request/response concerns.

**Q2. What is the danger of putting too much logic in middleware?**  
It makes request flow hard to debug and can add latency to every route.

---

## 2.6 Authentication layer

### Definition and info

Authentication verifies who the user is.

Common mechanisms:

- Username/password.
- Session cookies.
- JWT access tokens.
- OAuth2/OIDC.
- API keys.
- Magic links.
- SSO/SAML for enterprise.

### Use case

Login, session management, protected APIs, user-specific data.

### Production use case

Production auth must include password hashing, secure cookies, token expiry, refresh logic, session invalidation, account lockout/rate limits, MFA where necessary, and audit logs.

### Example

**Express.js pseudo-flow**

```ts
app.post("/login", async (req, res) => {
  const user = await userService.verifyPassword(req.body.email, req.body.password);

  if (!user) {
    return res.status(401).json({ message: "Invalid credentials" });
  }

  const token = tokenService.signAccessToken({ userId: user.id });
  res.json({ accessToken: token });
});
```

**FastAPI pseudo-flow**

```py
@app.post("/login")
def login(payload: LoginRequest):
    user = user_service.verify_password(payload.email, payload.password)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    return {"access_token": token_service.create_access_token(user.id)}
```

### Interview questions

**Q1. Authentication vs authorization?**  
Authentication asks, "Who are you?" Authorization asks, "What are you allowed to do?"

**Q2. JWT vs session cookies?**  
JWTs are stateless tokens often used for APIs/mobile clients. Sessions store server-side session state and use cookies as session identifiers. JWTs are harder to revoke unless you maintain deny lists or short lifetimes.

---

## 2.7 Authorization layer

### Definition and info

Authorization checks permissions after identity is known.

Common authorization models:

- RBAC: Role-Based Access Control.
- ABAC: Attribute-Based Access Control.
- ReBAC: Relationship-Based Access Control.
- ACL: Access Control List.
- Tenant-aware authorization.

### Use case

Check whether a user can read, create, update, or delete a resource.

### Production use case

In multi-tenant SaaS, authorization is critical. Every query must enforce tenant boundaries to prevent cross-tenant data leaks.

### Example

```ts
if (project.ownerId !== currentUser.id && currentUser.role !== "admin") {
  throw new ForbiddenError("You cannot access this project");
}
```

```py
if project.owner_id != current_user.id and current_user.role != "admin":
    raise HTTPException(status_code=403, detail="Forbidden")
```

### Interview questions

**Q1. Why is authorization harder than authentication?**  
Because permissions depend on resource ownership, roles, relationships, tenant boundaries, and business rules.

**Q2. Where should authorization be enforced?**  
At the backend/service/data-access layer. Frontend checks are only UX.

---

## 2.8 Service/business logic layer

### Definition and info

The service layer implements application-specific use cases.

Examples:

- Create user.
- Place order.
- Generate recommendation.
- Submit payment.
- Invite team member.
- Sync coding submissions.

It coordinates repositories, transactions, external services, and domain rules.

### Use case

Separates business rules from HTTP and database details.

### Production use case

The service layer becomes the main unit of application behavior and is often where idempotency, transaction boundaries, permission checks, and integration orchestration live.

### Example

```ts
class UserService {
  async createUser(input) {
    const existing = await userRepo.findByEmail(input.email);
    if (existing) throw new ConflictError("Email already exists");

    const passwordHash = await hashPassword(input.password);
    return userRepo.create({ ...input, passwordHash });
  }
}
```

```py
class UserService:
    def create_user(self, payload: CreateUserRequest):
        existing = user_repo.find_by_email(payload.email)
        if existing:
            raise ConflictError("Email already exists")
        password_hash = hash_password(payload.password)
        return user_repo.create(payload, password_hash)
```

### Interview questions

**Q1. What belongs in the service layer?**  
Business workflows, domain rules, orchestration, transaction boundaries, and calls to repositories/external systems.

**Q2. Why avoid business logic in controllers?**  
It causes duplication, weak testability, and tight coupling to HTTP.

---

## 2.9 Domain/model layer

### Definition and info

The domain layer represents core business concepts and rules. In simple CRUD apps, it may be thin. In complex apps, it contains domain entities, value objects, policies, and domain services.

### Use case

Useful when the business logic is complex and must be protected from framework/database details.

### Production use case

Payment systems, recommendation systems, permission systems, scheduling systems, and workflow engines benefit from explicit domain models.

### Example

```ts
class Subscription {
  constructor(private plan: "free" | "pro", private status: "active" | "cancelled") {}

  canUseAdvancedFeatures() {
    return this.plan === "pro" && this.status === "active";
  }
}
```

```py
class Subscription:
    def __init__(self, plan: str, status: str):
        self.plan = plan
        self.status = status

    def can_use_advanced_features(self) -> bool:
        return self.plan == "pro" and self.status == "active"
```

### Interview questions

**Q1. What is domain logic?**  
Rules that belong to the business, independent of HTTP, database, or framework.

**Q2. Does every app need DDD?**  
No. CRUD-heavy apps can stay simple. DDD is useful when business rules are complex.

---

## 2.10 Repository/data-access layer

### Definition and info

The repository layer encapsulates database queries. It hides raw SQL/ORM details from services.

### Use case

Use repositories when services should not know how persistence is implemented.

### Production use case

Repositories simplify testing, allow query optimization in one place, and reduce accidental data leakage.

### Example

```ts
class UserRepository {
  async findByEmail(email: string) {
    return prisma.user.findUnique({ where: { email } });
  }

  async create(data) {
    return prisma.user.create({ data });
  }
}
```

```py
class UserRepository:
    def __init__(self, db):
        self.db = db

    def find_by_email(self, email: str):
        return self.db.query(User).filter(User.email == email).first()
```

### Interview questions

**Q1. Repository pattern pros and cons?**  
Pros: isolation, testability, reuse. Cons: can become unnecessary abstraction if it only wraps one ORM call without adding value.

**Q2. Should every query be hidden behind a repository?**  
Not always. For small apps, direct ORM queries may be acceptable. For larger systems, repositories improve boundaries.

---

## 2.11 Persistence/database layer

### Definition and info

The persistence layer stores and retrieves data from databases.

Common stores:

- PostgreSQL/MySQL for relational data.
- MongoDB/DynamoDB for document/key-value use cases.
- Redis for cache/session/rate limit.
- Elasticsearch/OpenSearch for search.
- S3/R2/GCS for objects/files.
- Vector DB/pgvector for embeddings.

### Use case

Store user accounts, product data, orders, events, logs, recommendations, and content.

### Production use case

Production persistence must handle indexes, migrations, connection pooling, backups, transactions, consistency, retention, and observability.

### Example

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Interview questions

**Q1. SQL vs NoSQL?**  
SQL is better for structured relational data, transactions, joins, and consistency. NoSQL can be useful for flexible schemas, high-scale key-value access, or document-style data.

**Q2. What is a database transaction?**  
A transaction groups operations so they either all succeed or all fail, preserving consistency.

---

## 2.12 Cache layer

### Definition and info

A cache stores frequently used data closer to the application to reduce latency and database load.

Common cache types:

- In-memory cache.
- Redis.
- CDN cache.
- Application-level cache.
- Query cache.

### Use case

Cache read-heavy data, computed results, sessions, rate-limit counters, feature flags, or public content.

### Production use case

Caching improves latency but introduces invalidation complexity. The hardest cache problem is deciding when cached data is stale and how to update it safely.

### Example

```ts
const key = `user:${userId}`;
const cached = await redis.get(key);
if (cached) return JSON.parse(cached);

const user = await userRepo.findById(userId);
await redis.set(key, JSON.stringify(user), "EX", 60);
return user;
```

### Interview questions

**Q1. What is cache invalidation?**  
The process of removing or updating stale cached data after the source of truth changes.

**Q2. What is cache-aside?**  
The app checks the cache first. On miss, it reads from DB and writes the result to cache.

---

## 2.13 Queue/worker/background job layer

### Definition and info

Queues decouple slow or retryable work from request-response flow.

Examples:

- Send email.
- Process video.
- Generate report.
- Sync third-party data.
- Rebuild recommendations.
- Webhook processing.

### Use case

Use queues when work is slow, failure-prone, retryable, or not required before returning the HTTP response.

### Production use case

Queues need idempotency, retries, dead-letter queues, backoff, visibility timeouts, and monitoring.

### Example

```ts
await emailQueue.add("sendWelcomeEmail", { userId: user.id });
res.status(202).json({ status: "queued" });
```

```py
# FastAPI can schedule lightweight in-process background work.
# For durable jobs, use Celery, RQ, Dramatiq, Arq, or a managed queue.
from fastapi import BackgroundTasks

@app.post("/signup")
def signup(payload: SignupRequest, background_tasks: BackgroundTasks):
    user = create_user(payload)
    background_tasks.add_task(send_welcome_email, user.email)
    return {"id": user.id}
```

### Interview questions

**Q1. Why use a queue instead of doing everything inside the request?**  
To reduce response latency and handle slow/retryable work reliably.

**Q2. What is a dead-letter queue?**  
A queue where messages go after repeated processing failures, allowing manual inspection or later replay.

---

## 2.14 Integration layer

### Definition and info

The integration layer handles communication with external systems such as payment gateways, email providers, AI APIs, object storage, CRM systems, and webhooks.

### Use case

Use this layer to isolate third-party API details from business logic.

### Production use case

Third-party integrations fail. Production systems need timeouts, retries, circuit breakers, idempotency keys, logging, and provider-specific error handling.

### Example

```ts
class PaymentClient {
  async charge(input) {
    return http.post("https://payments.example.com/charges", input, {
      timeout: 5000,
      headers: { "Idempotency-Key": input.idempotencyKey },
    });
  }
}
```

### Interview questions

**Q1. How do you make third-party API calls reliable?**  
Use timeouts, retries with exponential backoff, idempotency keys, circuit breakers, and durable queues for async work.

**Q2. Why wrap external APIs behind a client class?**  
It centralizes provider-specific logic and makes testing/failover easier.

---

## 2.15 Configuration and secrets layer

### Definition and info

Configuration controls app behavior across environments. Secrets are sensitive values like database URLs, API keys, JWT signing keys, and encryption keys.

### Use case

Separate dev, staging, and production config.

### Production use case

Secrets should be stored in secret managers or platform environment variables, not committed to Git.

### Example

```ts
const config = {
  port: Number(process.env.PORT ?? 3000),
  databaseUrl: requiredEnv("DATABASE_URL"),
  jwtSecret: requiredEnv("JWT_SECRET"),
};
```

```py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    jwt_secret: str
    environment: str = "development"

settings = Settings()
```

### Interview questions

**Q1. Why avoid hardcoding secrets?**  
Secrets in code can leak through Git history, logs, builds, or shared repositories.

**Q2. What config should differ across environments?**  
Database URLs, logging level, API keys, allowed origins, feature flags, queue names, and external service endpoints.

---

## 2.16 Observability layer

### Definition and info

Observability helps you understand system behavior through:

- Logs.
- Metrics.
- Traces.
- Error reporting.
- Health checks.
- Audit logs.

### Use case

Debug production failures and measure reliability.

### Production use case

You should monitor latency, error rate, throughput, saturation, database query time, queue lag, cache hit rate, and external API failures.

### Example

```ts
logger.info({ requestId, userId, path: req.path }, "request completed");
```

```py
logger.info("request completed", extra={"request_id": request_id, "path": path})
```

### Interview questions

**Q1. Logs vs metrics vs traces?**  
Logs describe events. Metrics measure numeric trends. Traces follow a request across services.

**Q2. What is a request ID?**  
A unique ID attached to logs and responses so one request can be traced through the system.

---

## 2.17 Testing layer

### Definition and info

Backend testing verifies functionality at different levels:

- Unit tests.
- Integration tests.
- Contract tests.
- End-to-end tests.
- Load tests.
- Security tests.

### Use case

Prevent regressions and validate behavior before deployment.

### Production use case

A production-grade backend should test critical flows like auth, payment, data writes, permissions, migrations, and error handling.

### Example

**Express.js with Supertest**

```ts
import request from "supertest";

it("returns health", async () => {
  await request(app).get("/health").expect(200).expect({ status: "ok" });
});
```

**FastAPI with TestClient**

```py
from fastapi.testclient import TestClient

client = TestClient(app)

def test_health():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

### Interview questions

**Q1. Unit test vs integration test?**  
A unit test isolates one function/class. An integration test verifies multiple components working together, often including a database or HTTP layer.

**Q2. What should be mocked?**  
External services should often be mocked. Core domain logic and DB behavior should be tested realistically where possible.

---

# 3. HTTP and REST API design

## 3.1 HTTP methods

### Definition and info

HTTP methods describe the operation being requested.

Common methods:

- `GET`: read resource.
- `POST`: create resource or trigger action.
- `PUT`: replace full resource.
- `PATCH`: partially update resource.
- `DELETE`: delete resource.

### Use case

A clean REST API uses methods consistently.

### Production use case

Correct HTTP method usage improves caching, observability, API client generation, and predictability.

### Example

```txt
GET    /users/123
POST   /users
PATCH  /users/123
DELETE /users/123
```

### Interview questions

**Q1. PUT vs PATCH?**  
PUT usually replaces the full resource. PATCH updates selected fields.

**Q2. Should GET mutate data?**  
No. GET should be safe and should not change server state.

---

## 3.2 HTTP status codes

### Definition and info

Status codes communicate request outcome.

Common codes:

- `200 OK`
- `201 Created`
- `202 Accepted`
- `204 No Content`
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `422 Unprocessable Entity`
- `429 Too Many Requests`
- `500 Internal Server Error`

### Use case

Clients rely on status codes to handle responses correctly.

### Production use case

Consistent status codes reduce client bugs and make monitoring more meaningful.

### Example

```ts
res.status(201).json(createdUser);
```

```py
from fastapi import status

@app.post("/users", status_code=status.HTTP_201_CREATED)
def create_user():
    return {"id": 1}
```

### Interview questions

**Q1. 401 vs 403?**  
401 means the user is not authenticated. 403 means the user is authenticated but not allowed.

**Q2. 400 vs 422?**  
400 is generic malformed request. 422 is commonly used when the request shape is valid but semantic validation fails.

---

## 3.3 API resource modeling

### Definition and info

API resource modeling defines nouns/resources and operations around them.

Good resource examples:

```txt
/users
/users/{id}
/orders
/orders/{id}/items
```

Weak examples:

```txt
/getUser
/createNewOrder
/deleteOneUserNow
```

### Use case

Makes APIs predictable and self-documenting.

### Production use case

Clear resource modeling helps frontend teams, mobile teams, and external API consumers.

### Example

```txt
POST /organizations/{orgId}/members
DELETE /organizations/{orgId}/members/{userId}
```

### Interview questions

**Q1. What makes a RESTful API clean?**  
Resource-oriented URLs, correct HTTP methods, status codes, predictable errors, pagination, filtering, auth, and versioning.

**Q2. When is an action endpoint acceptable?**  
When the operation is not naturally CRUD, such as `/orders/{id}/cancel` or `/payments/{id}/capture`.

---

## 3.4 Pagination, filtering, and sorting

### Definition and info

Pagination limits returned data. Filtering narrows results. Sorting orders results.

Types:

- Offset pagination: `?limit=20&offset=40`
- Page pagination: `?page=3&pageSize=20`
- Cursor pagination: `?cursor=abc&limit=20`

### Use case

Prevent huge responses and database overload.

### Production use case

Cursor pagination is preferred for high-volume feeds because offset pagination gets slower and unstable as data changes.

### Example

```txt
GET /posts?limit=20&cursor=eyJjcmVhdGVkQXQiOiIyMDI2In0=
```

### Interview questions

**Q1. Offset vs cursor pagination?**  
Offset is simpler but can be slow and inconsistent on changing data. Cursor is more stable and scalable for large datasets.

**Q2. Why enforce max page size?**  
To prevent clients from requesting huge payloads that overload the database or API.

---

## 3.5 Idempotency

### Definition and info

An operation is idempotent if running it multiple times has the same final effect as running it once.

Examples:

- `GET` should be idempotent.
- `PUT` should be idempotent.
- `DELETE` is usually idempotent.
- `POST` is not automatically idempotent.

### Use case

Important for retries, payments, background jobs, and network failures.

### Production use case

For payment/order creation, clients should send an idempotency key. The backend stores the key and returns the previous result if the same request is retried.

### Example

```ts
const existing = await idempotencyRepo.findByKey(key);
if (existing) return existing.response;

const order = await orderService.createOrder(input);
await idempotencyRepo.save(key, order);
return order;
```

### Interview questions

**Q1. Why is idempotency important?**  
Because clients and workers retry after timeouts/failures. Without idempotency, retries can duplicate charges, orders, or side effects.

**Q2. How do you implement idempotency?**  
Accept an idempotency key, store request/result atomically, and return the stored result on duplicate attempts.

---

## 3.6 Error response design

### Definition and info

Error responses should be consistent and machine-readable.

Common shape:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": [
      { "field": "email", "message": "Invalid email" }
    ],
    "requestId": "req_123"
  }
}
```

### Use case

Helps clients show correct messages and helps engineers debug.

### Production use case

Do not leak stack traces, SQL errors, secrets, or internal implementation details to users.

### Example

```ts
app.use((err, req, res, _next) => {
  logger.error({ err, requestId: req.id });

  res.status(err.statusCode ?? 500).json({
    error: {
      code: err.code ?? "INTERNAL_ERROR",
      message: err.publicMessage ?? "Something went wrong",
      requestId: req.id,
    },
  });
});
```

### Interview questions

**Q1. Why standardize API errors?**  
So clients can handle failures consistently and logs can be correlated with request IDs.

**Q2. Should stack traces be returned to clients?**  
No. Stack traces should be logged internally only.

---

# 4. Express.js complete guide

## 4.1 What is Express.js?

### Definition and info

Express.js is a minimal, flexible web framework for Node.js. It provides routing and middleware primitives, but it is intentionally unopinionated about architecture, validation, database access, and project structure.

Express is lightweight. That is both a strength and a risk:

- Strength: freedom, small core, easy to start.
- Risk: teams must define architecture and conventions themselves.

### Use case

Use Express for REST APIs, BFFs, webhooks, microservices, internal tools, and lightweight HTTP services.

### Production use case

In production, Express is usually paired with:

- TypeScript.
- Zod/Joi/Yup for validation.
- Prisma/Drizzle/TypeORM/Knex for DB.
- Passport or custom auth.
- Helmet for security headers.
- Pino/Winston for logging.
- Jest/Vitest and Supertest for testing.
- Redis for cache/rate limit/session.
- BullMQ or similar for queues.

### Example

```ts
import express from "express";

const app = express();
app.use(express.json());

app.get("/api/v1/users/:id", async (req, res) => {
  res.json({ id: req.params.id });
});

app.listen(3000);
```

### Interview questions

**Q1. Why is Express called unopinionated?**  
Because it does not enforce project structure, ORM, validation library, auth system, or dependency injection pattern.

**Q2. When should you not use Express?**  
If you need strong built-in structure, automatic validation, dependency injection, OpenAPI generation, or async-first Python data APIs, another framework may fit better.

---

## 4.2 Node.js event loop

### Definition and info

Node.js uses an event loop to handle asynchronous non-blocking I/O. JavaScript runs on a single main thread by default, while many I/O operations are delegated to the OS or libuv worker pool.

This means Node.js is strong for I/O-heavy APIs but weak for CPU-heavy tasks inside the request thread.

### Use case

Good for APIs that spend most time waiting on databases, cache, network calls, or file I/O.

### Production use case

Do not block the event loop with CPU-heavy work, huge JSON parsing, synchronous file operations, expensive crypto, or long loops. Move CPU-heavy work to workers, queues, or separate services.

### Example

Bad:

```ts
app.get("/report", (_req, res) => {
  const result = generateHugeCpuHeavyReportSync();
  res.json(result);
});
```

Better:

```ts
app.post("/reports", async (_req, res) => {
  const job = await reportQueue.add("generate-report", {});
  res.status(202).json({ jobId: job.id });
});
```

### Interview questions

**Q1. What is the Node.js event loop?**  
It is the mechanism that lets Node handle many concurrent I/O operations without one OS thread per request.

**Q2. Why is blocking the event loop bad?**  
One CPU-heavy task can delay all other requests handled by that process.

**Q3. Is Node.js single-threaded?**  
JavaScript execution is single-threaded by default, but Node uses background workers and OS facilities for many I/O tasks.

---

## 4.3 Express application object

### Definition and info

The Express app object is created by calling `express()`. It registers middleware, routes, settings, and starts request handling.

### Use case

Central entry point for an Express API.

### Production use case

Export the app separately from the server listener. This improves testability and graceful startup/shutdown.

### Example

```ts
// app.ts
export const app = express();
app.use(express.json());
app.use("/api/v1/users", usersRouter);

// server.ts
import { app } from "./app";

const server = app.listen(process.env.PORT ?? 3000);
```

### Interview questions

**Q1. Why separate app and server files?**  
So tests can import the app without opening a network port, and server lifecycle can be managed separately.

**Q2. Where do you register global middleware?**  
Usually in the app bootstrap file before registering routes.

---

## 4.4 Express routing

### Definition and info

Express routing maps HTTP methods and paths to handler functions.

### Use case

Implement REST endpoints.

### Production use case

Use `express.Router()` to modularize routes by domain.

### Example

```ts
const router = Router();

router.get("/", listUsers);
router.get("/:id", getUser);
router.post("/", createUser);
router.patch("/:id", updateUser);
router.delete("/:id", deleteUser);

app.use("/api/v1/users", router);
```

### Interview questions

**Q1. What is `express.Router()`?**  
A mini-router used to group related routes and middleware.

**Q2. What is route order in Express?**  
Routes and middleware execute in the order they are registered, so order matters.

---

## 4.5 Express middleware

### Definition and info

Express middleware functions receive `req`, `res`, and `next`. They can end the response or pass control by calling `next()`.

Types:

- Application-level middleware.
- Router-level middleware.
- Error-handling middleware.
- Built-in middleware.
- Third-party middleware.

### Use case

Cross-cutting request logic.

### Production use case

Use middleware for auth, logging, CORS, rate limits, security headers, compression, and request parsing.

### Example

```ts
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.replace("Bearer ", "");

  if (!token) {
    return res.status(401).json({ message: "Missing token" });
  }

  req.user = verifyToken(token);
  next();
}

app.get("/me", requireAuth, getMe);
```

### Interview questions

**Q1. What does `next()` do?**  
It passes control to the next middleware or route handler.

**Q2. What happens if middleware neither sends a response nor calls `next()`?**  
The request hangs until it times out.

---

## 4.6 Express error handling

### Definition and info

Express error middleware has four arguments: `(err, req, res, next)`. Calling `next(err)` skips normal middleware and routes and passes control to error handlers.

In Express 5, rejected promises from async route handlers are passed to the error handler automatically. In older Express 4 code, you often see wrappers such as `asyncHandler(fn)`.

### Use case

Centralize error responses and logging.

### Production use case

Use custom error classes and a global error handler to avoid leaking implementation details.

### Example

```ts
class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    public publicMessage: string
  ) {
    super(publicMessage);
  }
}

app.use((err, req, res, _next) => {
  req.log?.error({ err }, "request failed");

  const status = err.statusCode ?? 500;
  res.status(status).json({
    error: {
      code: err.code ?? "INTERNAL_ERROR",
      message: err.publicMessage ?? "Something went wrong",
    },
  });
});
```

### Interview questions

**Q1. How does Express detect error middleware?**  
By the function signature with four parameters: `err, req, res, next`.

**Q2. Why centralize error handling?**  
To make API responses consistent and avoid duplicating try/catch logic everywhere.

---

## 4.7 Express request and response objects

### Definition and info

`req` contains request data: params, query, body, headers, cookies, IP, route info. `res` is used to send the response.

### Use case

Controllers use these objects to read input and return output.

### Production use case

Avoid passing raw `req`/`res` into deep service layers. Extract typed values and call services.

### Example

```ts
app.get("/users/:id", (req, res) => {
  const id = req.params.id;
  const includePosts = req.query.includePosts === "true";
  res.json({ id, includePosts });
});
```

### Interview questions

**Q1. Difference between `req.params`, `req.query`, and `req.body`?**  
`params` come from route path, `query` from URL query string, and `body` from request payload.

**Q2. Why validate all three?**  
All are user-controlled input.

---

## 4.8 Express validation

### Definition and info

Express does not include schema validation by default. You must add validation with libraries such as Zod, Joi, Yup, Valibot, or express-validator.

### Use case

Validate request bodies, query params, headers, and route params.

### Production use case

A common production pattern is a reusable validation middleware.

### Example

```ts
function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ errors: result.error.flatten() });
    }
    req.body = result.data;
    next();
  };
}

router.post("/", validateBody(createUserSchema), createUserController);
```

### Interview questions

**Q1. Does Express validate request bodies automatically?**  
No. You need validation middleware or a validation library.

**Q2. What should you validate besides body?**  
Params, query strings, headers, cookies, files, and business-level constraints.

---

## 4.9 Express project structure

### Definition and info

Express does not force structure. A maintainable backend usually separates routes, controllers, services, repositories, middleware, config, and tests.

### Use case

Prevent large unmaintainable route files.

### Production use case

A clean structure helps onboarding, testing, scaling, and ownership by domain.

### Example

```txt
src/
  app.ts
  server.ts
  config/
  modules/
    users/
      users.routes.ts
      users.controller.ts
      users.service.ts
      users.repository.ts
      users.schema.ts
  middleware/
  errors/
  db/
  tests/
```

### Interview questions

**Q1. How would you structure a large Express app?**  
By feature modules with route/controller/service/repository/schema files, plus shared middleware, config, db, and error utilities.

**Q2. Why avoid one giant `server.ts`?**  
It becomes hard to test, refactor, review, and reason about.

---

## 4.10 Express authentication

### Definition and info

Express authentication can be implemented manually or with libraries such as Passport, session middleware, JWT libraries, OAuth clients, or auth platforms.

### Use case

Protect APIs and identify users.

### Production use case

Use secure password hashing, short-lived access tokens, refresh token rotation, HTTP-only cookies where appropriate, and rate limits on login routes.

### Example

```ts
function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith("Bearer ")) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  try {
    req.user = verifyJwt(header.slice(7));
    next();
  } catch {
    res.status(401).json({ message: "Invalid token" });
  }
}
```

### Interview questions

**Q1. Where should auth middleware run?**  
On protected route groups, before the controller.

**Q2. Why use HTTP-only cookies?**  
They reduce token theft risk from XSS because JavaScript cannot read them.

---

## 4.11 Express security

### Definition and info

Express security requires deliberate configuration because Express is minimal.

Important protections:

- Helmet/security headers.
- CORS restrictions.
- Input validation.
- Rate limiting.
- Secure cookies.
- TLS in production.
- Avoid dependency vulnerabilities.
- Prevent open redirects.
- Avoid leaking stack traces.

### Use case

Protect APIs and users.

### Production use case

Apply security headers, restrict origins, rate-limit auth routes, validate input, and keep dependencies patched.

### Example

```ts
import helmet from "helmet";
import cors from "cors";

app.use(helmet());
app.use(cors({ origin: ["https://app.example.com"], credentials: true }));
```

### Interview questions

**Q1. What is Helmet?**  
A middleware package that sets security-related HTTP headers.

**Q2. What is CORS?**  
Browser-enforced policy that controls which origins can read responses from your API.

---

## 4.12 Express database access

### Definition and info

Express itself has no database layer. You choose libraries such as Prisma, Drizzle, TypeORM, Sequelize, Knex, Mongoose, or raw SQL clients.

### Use case

Persist application data.

### Production use case

Use connection pooling, migrations, indexes, transactions, and query monitoring.

### Example with Prisma

```ts
const user = await prisma.user.create({
  data: {
    email: input.email,
    passwordHash,
  },
});
```

### Interview questions

**Q1. What should you consider when choosing an ORM?**  
Type safety, migration support, query flexibility, performance, transaction support, ecosystem maturity, and team familiarity.

**Q2. Why use migrations?**  
To version database schema changes and apply them reliably across environments.

---

## 4.13 Express transactions

### Definition and info

Transactions ensure multiple database operations succeed or fail together.

### Use case

Creating an order and its line items, deducting credits, transferring money, or updating multiple related tables.

### Production use case

Use transactions around consistency-sensitive workflows. Keep transaction duration short.

### Example

```ts
await prisma.$transaction(async (tx) => {
  const order = await tx.order.create({ data: orderData });
  await tx.orderItem.createMany({ data: items.map(i => ({ ...i, orderId: order.id })) });
});
```

### Interview questions

**Q1. Why keep transactions short?**  
Long transactions hold locks longer and can reduce database concurrency.

**Q2. What can go wrong without transactions?**  
Partial writes can leave inconsistent data.

---

## 4.14 Express logging

### Definition and info

Logging records events and errors. Express apps often use Morgan for request logs and Pino/Winston for structured application logs.

### Use case

Debug requests, track failures, audit behavior.

### Production use case

Use structured JSON logs with request IDs, user IDs, route names, latency, and error details.

### Example

```ts
app.use((req, res, next) => {
  req.id = crypto.randomUUID();
  res.setHeader("X-Request-ID", req.id);
  next();
});
```

### Interview questions

**Q1. Why structured logs?**  
They are easier to query, filter, and aggregate in log systems.

**Q2. What should not be logged?**  
Passwords, tokens, full credit card data, secrets, and sensitive personal data.

---

## 4.15 Express testing

### Definition and info

Express apps are commonly tested with Jest/Vitest and Supertest.

### Use case

Test routes, middleware, controllers, and integration flows.

### Production use case

Run tests in CI with a test database and isolated test data.

### Example

```ts
import request from "supertest";
import { app } from "../src/app";

it("creates user", async () => {
  const res = await request(app)
    .post("/api/v1/users")
    .send({ email: "a@example.com", password: "password123", name: "A" })
    .expect(201);

  expect(res.body.email).toBe("a@example.com");
});
```

### Interview questions

**Q1. How do you test Express middleware?**  
By mounting it in a test app or testing through a route with Supertest.

**Q2. Why export the app separately from `listen()`?**  
So Supertest can test the app without starting a real server port.

---

## 4.16 Express production deployment

### Definition and info

Express apps run on Node.js. They can be deployed on VMs, Docker, Kubernetes, serverless platforms, or PaaS providers.

### Use case

Serve APIs in production.

### Production use case

Use process managers, health checks, graceful shutdown, environment config, horizontal scaling, reverse proxy, TLS, logs, and metrics.

### Example graceful shutdown

```ts
const server = app.listen(port);

process.on("SIGTERM", () => {
  server.close(async () => {
    await prisma.$disconnect();
    process.exit(0);
  });
});
```

### Interview questions

**Q1. What is graceful shutdown?**  
Stopping the server from accepting new requests, allowing in-flight requests to finish, closing DB/queue connections, then exiting.

**Q2. How do you scale Express?**  
Run multiple Node processes/containers behind a load balancer and use shared external state for sessions/cache/queues.

---

# 5. FastAPI complete guide

## 5.1 What is FastAPI?

### Definition and info

FastAPI is a modern Python web framework for building APIs. It uses Python type hints and Pydantic models for validation, serialization, and automatic OpenAPI documentation.

FastAPI is built on ASGI, Starlette, and Pydantic.

### Use case

Use FastAPI for APIs, AI/ML services, data-heavy systems, internal tools, microservices, model-serving APIs, and Python-first backend systems.

### Production use case

FastAPI is strong when backend logic is close to Python data/ML ecosystems or when automatic validation and OpenAPI docs are important.

### Example

```py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items")
def create_item(item: Item):
    return item
```

### Interview questions

**Q1. Why is FastAPI popular?**  
Because it combines high performance, type-based validation, automatic docs, async support, and simple developer experience.

**Q2. What is FastAPI built on?**  
It is built on Starlette for web/ASGI functionality and Pydantic for data validation/serialization.

---

## 5.2 ASGI and Uvicorn

### Definition and info

ASGI stands for Asynchronous Server Gateway Interface. It is the Python async server interface used by FastAPI. Uvicorn is a common ASGI server used to run FastAPI apps.

### Use case

ASGI enables async request handling, WebSockets, and modern Python web concurrency.

### Production use case

Run FastAPI behind a production server setup, commonly Uvicorn workers behind Gunicorn or managed platform process runners.

### Example

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### Interview questions

**Q1. WSGI vs ASGI?**  
WSGI is the older synchronous Python web interface. ASGI supports async and WebSockets.

**Q2. What does Uvicorn do?**  
It runs the FastAPI ASGI application and handles network-level HTTP/WebSocket serving.

---

## 5.3 FastAPI path operations

### Definition and info

A path operation is a function decorated with an HTTP method and path, such as `@app.get('/users/{id}')`.

### Use case

Define API endpoints.

### Production use case

Use routers to organize path operations by domain.

### Example

```py
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id}
```

### Interview questions

**Q1. What is a path operation in FastAPI?**  
An endpoint function mapped to a path and HTTP method.

**Q2. Why do type annotations matter in FastAPI?**  
They drive validation, conversion, editor support, and OpenAPI docs.

---

## 5.4 Path, query, body, header, and cookie parameters

### Definition and info

FastAPI can extract parameters from different parts of the request:

- Path: `/users/{id}`.
- Query: `/users?limit=10`.
- Body: JSON payload.
- Header: request headers.
- Cookie: request cookies.

### Use case

Build strongly typed API inputs.

### Production use case

Use explicit validation constraints for safer APIs.

### Example

```py
from typing import Annotated
from fastapi import Query, Header

@app.get("/items/{item_id}")
def read_item(
    item_id: int,
    limit: Annotated[int, Query(gt=0, le=100)] = 20,
    user_agent: Annotated[str | None, Header()] = None,
):
    return {"item_id": item_id, "limit": limit, "user_agent": user_agent}
```

### Interview questions

**Q1. How does FastAPI know where a parameter comes from?**  
Path variables come from matching route names. Pydantic models usually represent body data. `Query`, `Header`, `Cookie`, and `Path` make sources explicit.

**Q2. How do you limit a query param to max 100?**  
Use `Query(le=100)`.

---

## 5.5 Pydantic models

### Definition and info

Pydantic models define typed data schemas for validation and serialization.

### Use case

Request body validation, response models, settings, nested JSON structures.

### Production use case

Pydantic reduces manual validation code and ensures API contracts stay explicit.

### Example

```py
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)
    full_name: str | None = None
```

### Interview questions

**Q1. What does Pydantic do in FastAPI?**  
It validates input, converts data types, serializes responses, and powers schema generation.

**Q2. Why define separate create/update/read models?**  
Because input and output shapes differ. For example, output should not include password fields.

---

## 5.6 Response models

### Definition and info

Response models define the expected output shape. FastAPI can filter, validate, and document response data.

### Use case

Avoid leaking internal fields and provide accurate OpenAPI docs.

### Production use case

Use response models to prevent exposing password hashes, internal flags, secret tokens, or unneeded database columns.

### Example

```py
class UserPublic(BaseModel):
    id: int
    email: EmailStr
    full_name: str | None = None

@app.get("/users/{user_id}", response_model=UserPublic)
def get_user(user_id: int):
    return user_repo.get(user_id)
```

### Interview questions

**Q1. Why use response models?**  
To control output shape, improve docs, and avoid leaking internal data.

**Q2. Can FastAPI filter response fields?**  
Yes, response models can filter returned objects to declared fields.

---

## 5.7 FastAPI dependency injection

### Definition and info

FastAPI has a dependency injection system using `Depends()` and `Security()`. Endpoint functions declare dependencies, and FastAPI resolves them.

Dependencies can provide:

- Database sessions.
- Current user.
- Permissions.
- Pagination parameters.
- Config.
- External clients.

### Use case

Share reusable logic across endpoints.

### Production use case

Use dependencies for auth, DB sessions, tenant resolution, permission checks, and common filters.

### Example

```py
from fastapi import Depends

async def get_current_user(token: str = Depends(oauth2_scheme)):
    return decode_token(token)

@app.get("/me")
def get_me(current_user = Depends(get_current_user)):
    return current_user
```

### Interview questions

**Q1. What is dependency injection in FastAPI?**  
A way for route functions to declare required dependencies that FastAPI resolves automatically.

**Q2. Why use dependencies for DB sessions?**  
To consistently open, inject, commit/rollback, and close sessions per request.

---

## 5.8 FastAPI routers

### Definition and info

`APIRouter` groups related endpoints and can define prefixes, tags, dependencies, and responses.

### Use case

Modular route organization.

### Production use case

Use one router per domain module.

### Example

```py
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/{user_id}")
def get_user(user_id: int):
    return {"id": user_id}

app.include_router(router, prefix="/api/v1")
```

### Interview questions

**Q1. Why use APIRouter?**  
To organize endpoints by feature and avoid one huge app file.

**Q2. Can dependencies be applied to a whole router?**  
Yes, router-level dependencies can apply to every endpoint in that router.

---

## 5.9 FastAPI middleware

### Definition and info

Middleware runs for every request before it reaches path operations and for every response before it returns.

### Use case

Logging, request IDs, CORS, timing, auth-like cross-cutting behavior.

### Production use case

Keep middleware lightweight. Use dependencies for route-specific logic.

### Example

```py
@app.middleware("http")
async def request_id_middleware(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id
    return response
```

### Interview questions

**Q1. Middleware vs dependency?**  
Middleware wraps all matching requests globally. Dependencies are route-aware and better for business-level requirements like current user or DB session.

**Q2. What should not be in middleware?**  
Heavy database queries or complex business logic that does not apply to every request.

---

## 5.10 FastAPI exception handling

### Definition and info

FastAPI uses `HTTPException` for expected API errors and custom exception handlers for global error formatting.

### Use case

Return structured errors.

### Production use case

Centralize error responses and log unexpected exceptions.

### Example

```py
from fastapi import HTTPException

@app.get("/users/{user_id}")
def get_user(user_id: int):
    user = user_repo.get(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Interview questions

**Q1. When should you raise `HTTPException`?**  
For expected API-level failures like not found, unauthorized, forbidden, or invalid input.

**Q2. How do you customize global error handling?**  
Use `@app.exception_handler(SomeException)`.

---

## 5.11 FastAPI authentication and security

### Definition and info

FastAPI provides security utilities for OAuth2, bearer tokens, API keys, HTTP Basic, scopes, and OpenAPI integration.

### Use case

Protect APIs and document auth requirements automatically.

### Production use case

Use secure password hashing, token expiry, refresh token strategy, scoped permissions, and route-level dependencies.

### Example

```py
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    payload = verify_jwt(token)
    return user_repo.get(payload["sub"])

@app.get("/me")
def me(user = Depends(get_current_user)):
    return user
```

### Interview questions

**Q1. What is `OAuth2PasswordBearer`?**  
A FastAPI security dependency that extracts bearer tokens from the Authorization header and documents the flow in OpenAPI.

**Q2. Does FastAPI automatically make JWT secure?**  
No. You must choose secure algorithms, validate claims, set expirations, hash passwords, and manage refresh/revocation.

---

## 5.12 FastAPI database access

### Definition and info

FastAPI does not force a database ORM. Common choices include SQLAlchemy, SQLModel, Tortoise ORM, Prisma Python, asyncpg, psycopg, and Motor for MongoDB.

### Use case

Persist application data.

### Production use case

Use a real database server like PostgreSQL for production, with migrations, connection pooling, transactions, indexes, and backups.

### Example with SQLAlchemy-style dependency

```py
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.query(User).filter(User.id == user_id).first()
```

### Interview questions

**Q1. Does FastAPI include an ORM?**  
No. It works with many database libraries, but you choose the persistence layer.

**Q2. Why inject DB sessions with dependencies?**  
It creates a consistent per-request lifecycle for database sessions.

---

## 5.13 FastAPI async vs sync

### Definition and info

FastAPI supports both `def` and `async def` path operations.

Use `async def` when calling async libraries. Use regular `def` for blocking sync code, especially when using sync database drivers.

### Use case

Async helps when handling many concurrent I/O-bound requests with async-compatible libraries.

### Production use case

Do not call blocking sync code directly inside `async def`. It can block the event loop.

### Example

```py
@app.get("/external")
async def call_external_api():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
    return response.json()
```

### Interview questions

**Q1. When should you use `async def` in FastAPI?**  
When the function awaits async I/O, such as async database drivers or async HTTP clients.

**Q2. What is a common async mistake?**  
Using blocking libraries inside `async def`, which blocks the event loop.

---

## 5.14 FastAPI background tasks

### Definition and info

FastAPI `BackgroundTasks` can run simple tasks after sending a response.

### Use case

Send lightweight emails, write logs, perform small post-response work.

### Production use case

For durable jobs, use a real queue. FastAPI background tasks are not a replacement for Celery/RQ/Dramatiq/Arq when reliability matters.

### Example

```py
from fastapi import BackgroundTasks

@app.post("/invite")
def invite_user(payload: InviteRequest, background_tasks: BackgroundTasks):
    invite = create_invite(payload)
    background_tasks.add_task(send_invite_email, invite.email)
    return {"status": "created"}
```

### Interview questions

**Q1. Are FastAPI background tasks durable?**  
No. If the process crashes, tasks may be lost. Use a queue for durable work.

**Q2. When are background tasks appropriate?**  
For lightweight best-effort work that can run after the response.

---

## 5.15 FastAPI lifespan events

### Definition and info

Lifespan events run startup and shutdown logic using an async context manager.

### Use case

Initialize models, database pools, clients, caches, and close them on shutdown.

### Production use case

Use lifespan to safely initialize and tear down expensive resources.

### Example

```py
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.cache = await create_cache_client()
    yield
    await app.state.cache.close()

app = FastAPI(lifespan=lifespan)
```

### Interview questions

**Q1. What are lifespan events used for?**  
Startup and shutdown resource management.

**Q2. Why not create DB clients inside every route?**  
It is inefficient and can exhaust connections. Use shared pools/sessions with proper lifecycle management.

---

## 5.16 FastAPI OpenAPI documentation

### Definition and info

FastAPI automatically generates OpenAPI schema and interactive docs based on path operations, type hints, Pydantic models, and security declarations.

### Use case

API exploration, client generation, testing, and documentation.

### Production use case

Keep response models, tags, summaries, descriptions, and error responses accurate. For private APIs, protect or disable docs in production if needed.

### Example

```py
@app.post(
    "/users",
    response_model=UserPublic,
    summary="Create a user",
    tags=["users"],
)
def create_user(payload: UserCreate):
    return user_service.create_user(payload)
```

### Interview questions

**Q1. How does FastAPI generate docs?**  
From route decorators, type hints, Pydantic models, response models, and security dependencies.

**Q2. Why are auto docs useful?**  
They reduce documentation drift and make the API easier to test and consume.

---

## 5.17 FastAPI project structure

### Definition and info

FastAPI does not force one structure, but production apps should separate routers, schemas, services, repositories, dependencies, config, and tests.

### Use case

Maintainability.

### Production use case

Organize by domain modules to avoid one giant `main.py`.

### Example

```txt
app/
  main.py
  core/
    config.py
    security.py
  db/
    session.py
    models.py
  modules/
    users/
      router.py
      schemas.py
      service.py
      repository.py
  dependencies/
  middleware/
  tests/
```

### Interview questions

**Q1. How do you structure a large FastAPI app?**  
Use routers and domain modules, with separate schemas, services, repositories, dependencies, and tests.

**Q2. Why avoid putting everything in `main.py`?**  
It becomes hard to test, maintain, and scale across teams.

---

## 5.18 FastAPI testing

### Definition and info

FastAPI provides testing support through Starlette's TestClient. You can test APIs by making requests against the app object.

### Use case

Test endpoints, dependencies, auth, validation, and integration flows.

### Production use case

Override dependencies for test DB sessions, fake auth users, or mocked external clients.

### Example

```py
from fastapi.testclient import TestClient

client = TestClient(app)

def test_create_user():
    response = client.post("/users", json={
        "email": "a@example.com",
        "password": "password123",
        "name": "A"
    })
    assert response.status_code == 201
```

### Interview questions

**Q1. How do you mock dependencies in FastAPI tests?**  
Use `app.dependency_overrides`.

**Q2. What should endpoint tests verify?**  
Status code, response body, validation behavior, authorization behavior, and database side effects.

---

## 5.19 FastAPI production deployment

### Definition and info

FastAPI apps are deployed as ASGI applications, commonly with Uvicorn workers, containers, or managed platforms.

### Use case

Serve APIs in production.

### Production use case

Use environment-based config, multiple workers, health checks, graceful shutdown, logs/metrics/traces, reverse proxy, and resource limits.

### Example Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Interview questions

**Q1. What runs a FastAPI app in production?**  
An ASGI server such as Uvicorn, often inside containers or behind a reverse proxy/load balancer.

**Q2. How do you scale FastAPI?**  
Run multiple workers/containers behind a load balancer and use external shared systems for DB, cache, sessions, and queues.

---

# 6. Express.js vs FastAPI

## 6.1 High-level comparison

| Area | Express.js | FastAPI |
|---|---|---|
| Language | JavaScript/TypeScript | Python |
| Runtime | Node.js | ASGI/Python |
| Philosophy | Minimal, unopinionated | Type-driven, batteries for validation/docs |
| Validation | External library needed | Built in through Pydantic/type hints |
| API docs | External setup usually needed | Automatic OpenAPI docs |
| Async model | Node event loop/promises | async/await with ASGI |
| Best fit | JS/TS backend, BFF, lightweight APIs | Python APIs, ML/data APIs, typed APIs |
| Structure | You define conventions | More built-in conventions, still flexible |

### Interview questions

**Q1. When would you choose Express.js?**  
When the team is JS/TS-first, needs a lightweight flexible backend, or wants strong integration with Node ecosystem.

**Q2. When would you choose FastAPI?**  
When the backend is Python-first, needs strong validation/docs, integrates with ML/data tooling, or benefits from type-driven API contracts.

**Q3. Which is faster?**  
Raw framework benchmarks are less important than production bottlenecks. Database queries, external APIs, cache strategy, serialization, and architecture usually dominate real latency.

---

# 7. Databases and persistence topics

## 7.1 Relational database design

### Definition and info

Relational databases store structured data in tables with relationships. PostgreSQL and MySQL are common choices.

### Use case

Users, orders, payments, inventory, permissions, recommendations, analytics events.

### Production use case

Use normalized schemas for consistency, indexes for query performance, constraints for integrity, and migrations for controlled schema evolution.

### Example

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE organization_members (
  organization_id UUID REFERENCES organizations(id),
  user_id UUID NOT NULL,
  role TEXT NOT NULL,
  PRIMARY KEY (organization_id, user_id)
);
```

### Interview questions

**Q1. What is normalization?**  
Structuring data to reduce duplication and preserve consistency.

**Q2. What are indexes?**  
Data structures that speed up reads but add write/storage overhead.

---

## 7.2 Transactions and isolation

### Definition and info

Transactions provide ACID guarantees:

- Atomicity.
- Consistency.
- Isolation.
- Durability.

Isolation levels control how concurrent transactions see each other's changes.

### Use case

Money movement, inventory updates, order creation, account credits.

### Production use case

Choose transaction boundaries carefully. Too little transaction use causes inconsistency. Too much or too long transactions cause lock contention.

### Example

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

### Interview questions

**Q1. What are ACID properties?**  
Atomicity, Consistency, Isolation, and Durability.

**Q2. What is a race condition?**  
When concurrent operations produce incorrect results because they read/write shared state without proper coordination.

---

## 7.3 ORM vs raw SQL

### Definition and info

An ORM maps database tables to language objects. Raw SQL writes queries directly.

### Use case

ORMs improve productivity. Raw SQL gives maximum query control.

### Production use case

Use ORM for common CRUD and raw SQL for complex performance-critical queries when needed.

### Example

**ORM-style**

```ts
await prisma.user.findMany({ where: { status: "active" } });
```

**SQL-style**

```sql
SELECT * FROM users WHERE status = 'active';
```

### Interview questions

**Q1. ORM pros and cons?**  
Pros: productivity, type mapping, migrations, less boilerplate. Cons: hidden queries, N+1 problems, less control over complex SQL.

**Q2. What is the N+1 query problem?**  
Fetching one list and then issuing one extra query per item, causing many unnecessary database round trips.

---

## 7.4 Migrations

### Definition and info

Migrations are versioned database schema changes.

### Use case

Create tables, add columns, create indexes, change constraints.

### Production use case

Migrations must be backward compatible when deploying zero-downtime systems.

### Example

```sql
ALTER TABLE users ADD COLUMN last_login_at TIMESTAMPTZ;
CREATE INDEX idx_users_last_login_at ON users(last_login_at);
```

### Interview questions

**Q1. Why use migrations?**  
To apply schema changes consistently across environments.

**Q2. What is a backward-compatible migration?**  
A migration that does not break old and new application versions during rolling deploys.

---

# 8. Authentication, authorization, and sessions

## 8.1 Password storage

### Definition and info

Passwords must never be stored in plain text. Store password hashes using slow, salted algorithms such as Argon2, bcrypt, or scrypt.

### Use case

Login systems.

### Production use case

Use strong hashing, rate-limit login attempts, protect reset flows, and never log passwords.

### Example

```ts
const hash = await bcrypt.hash(password, 12);
const valid = await bcrypt.compare(password, hash);
```

### Interview questions

**Q1. Hashing vs encryption?**  
Hashing is one-way and used for passwords. Encryption is reversible with a key and used when data must be recovered.

**Q2. Why salt passwords?**  
To prevent attackers from using precomputed rainbow tables and to make identical passwords produce different hashes.

---

## 8.2 JWT

### Definition and info

JWT is a signed token format containing claims. It is often used as a bearer access token.

### Use case

Stateless API authentication.

### Production use case

Use short-lived access tokens, validate issuer/audience/expiry, rotate refresh tokens, and never store sensitive data inside JWT payloads.

### Example payload

```json
{
  "sub": "user_123",
  "role": "admin",
  "exp": 1790000000
}
```

### Interview questions

**Q1. Can JWTs be revoked?**  
Not easily if purely stateless. Use short expiry, token versioning, deny lists, or server-side sessions if revocation is important.

**Q2. Is JWT payload encrypted?**  
Usually no. Standard JWTs are signed, not encrypted. Do not put secrets in payloads.

---

## 8.3 Sessions and cookies

### Definition and info

Session auth stores session state on the server or in a session store and sends a session ID in a cookie.

### Use case

Traditional web apps and browser-based auth.

### Production use case

Use HTTP-only, Secure, SameSite cookies. Store sessions in Redis/database for multi-instance deployments.

### Example

```ts
res.cookie("sid", sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
});
```

### Interview questions

**Q1. Why HTTP-only cookies?**  
They cannot be read by JavaScript, reducing token theft from XSS.

**Q2. What does SameSite do?**  
It controls whether cookies are sent on cross-site requests, helping reduce CSRF risk.

---

# 9. Security topics

## 9.1 Input validation and sanitization

### Definition and info

Validation checks whether input is acceptable. Sanitization transforms input into a safer form.

### Use case

Prevent malformed data, SQL injection, NoSQL injection, command injection, XSS, and invalid business states.

### Production use case

Validate all external input and use parameterized queries.

### Example

```ts
const schema = z.object({ id: z.string().uuid() });
```

### Interview questions

**Q1. Validation vs sanitization?**  
Validation rejects bad input. Sanitization modifies input to make it safer or normalized.

**Q2. How do you prevent SQL injection?**  
Use parameterized queries/ORMs and avoid string-concatenated SQL.

---

## 9.2 CORS

### Definition and info

CORS controls browser access from one origin to another. It is enforced by browsers, not by backend-to-backend clients.

### Use case

Allow a frontend domain to call an API domain.

### Production use case

Do not use wildcard origins with credentials. Whitelist known frontend origins.

### Example

```ts
app.use(cors({
  origin: ["https://app.example.com"],
  credentials: true,
}));
```

```py
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Interview questions

**Q1. Does CORS protect your API from all attackers?**  
No. It is a browser policy. Non-browser clients can still call your API.

**Q2. Why is `origin: *` dangerous with credentials?**  
It can expose authenticated responses to untrusted origins.

---

## 9.3 CSRF

### Definition and info

CSRF tricks a browser into sending authenticated requests using existing cookies.

### Use case

Relevant when using cookie-based authentication.

### Production use case

Use SameSite cookies, CSRF tokens, origin checks, and avoid unsafe GET mutations.

### Example

```txt
POST /transfer-money
Cookie: sid=valid-session
```

If a malicious site can trigger this request, CSRF protection is needed.

### Interview questions

**Q1. Is CSRF a concern for bearer tokens in Authorization headers?**  
Less commonly, because browsers do not automatically attach custom Authorization headers to cross-site forms.

**Q2. How do you prevent CSRF?**  
SameSite cookies, CSRF tokens, origin/referer checks, and using proper HTTP methods.

---

## 9.4 Rate limiting

### Definition and info

Rate limiting restricts how many requests a client can make in a time window.

### Use case

Prevent abuse, brute force, scraping, and accidental overload.

### Production use case

Use Redis-backed distributed rate limiting for multi-instance deployments.

### Example

```ts
import rateLimit from "express-rate-limit";

app.use("/login", rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 10,
}));
```

### Interview questions

**Q1. Why Redis for rate limiting?**  
Because all app instances need shared counters.

**Q2. What routes need stricter limits?**  
Login, password reset, OTP, signup, scraping-prone endpoints, and expensive AI/search endpoints.

---

# 10. Caching and performance

## 10.1 Redis

### Definition and info

Redis is an in-memory data store commonly used for caching, sessions, rate limiting, queues, and distributed locks.

### Use case

Fast access to frequently used data.

### Production use case

Use TTLs, eviction policy awareness, memory monitoring, and avoid treating cache as the only source of truth unless designed that way.

### Example

```ts
await redis.set("feature-flags", JSON.stringify(flags), "EX", 300);
```

### Interview questions

**Q1. What is TTL?**  
Time to live: how long a cached key remains valid before expiring.

**Q2. Can Redis be used as a queue?**  
Yes, libraries like BullMQ use Redis for job queues, but operational requirements still matter.

---

## 10.2 Database performance

### Definition and info

Database performance depends on query shape, indexes, data volume, locks, connection pooling, and schema design.

### Use case

Reduce latency and scale API throughput.

### Production use case

Use `EXPLAIN`, slow query logs, indexes, pagination, denormalization where justified, and connection pool sizing.

### Example

```sql
CREATE INDEX idx_submissions_user_created_at
ON submissions(user_id, created_at DESC);
```

### Interview questions

**Q1. How do you debug a slow API?**  
Break down latency: app time, DB query time, external API time, cache hit/miss, serialization, network.

**Q2. Can indexes hurt performance?**  
Yes. They speed reads but slow writes and use storage.

---

# 11. Queues, events, and workers

## 11.1 Background workers

### Definition and info

Workers process jobs outside the request path.

### Use case

Emails, imports, exports, report generation, recommendation refreshes, webhook retries.

### Production use case

Use idempotent handlers and track job status.

### Example

```ts
worker.process("send-email", async (job) => {
  await emailClient.send(job.data);
});
```

### Interview questions

**Q1. What makes a worker handler safe?**  
Idempotency, retries, timeouts, structured logging, and clear failure handling.

**Q2. What is queue lag?**  
The delay between when a job is enqueued and when it is processed.

---

## 11.2 Event-driven architecture

### Definition and info

Event-driven architecture records or publishes facts that happened, such as `UserCreated`, `PaymentSucceeded`, or `SubmissionFailed`.

### Use case

Decouple systems and trigger async workflows.

### Production use case

Use an outbox pattern or transactional event table to avoid losing events when database writes and message publishing must both happen.

### Example

```json
{
  "eventType": "UserCreated",
  "eventId": "evt_123",
  "occurredAt": "2026-06-29T10:00:00Z",
  "payload": { "userId": "user_123" }
}
```

### Interview questions

**Q1. What is the outbox pattern?**  
Save business data and an event in the same DB transaction, then a worker publishes the event later.

**Q2. Why are events usually immutable?**  
They represent facts that happened. Corrections should be new events, not mutation of history.

---

# 12. Observability and operations

## 12.1 Health checks

### Definition and info

Health checks tell infrastructure whether an app instance is alive and ready to receive traffic.

Types:

- Liveness: process is alive.
- Readiness: app can serve traffic.

### Use case

Load balancers and orchestrators use health checks.

### Production use case

Readiness may check database/cache connectivity. Liveness should be lightweight.

### Example

```ts
app.get("/health/live", (_req, res) => res.json({ status: "alive" }));
app.get("/health/ready", async (_req, res) => {
  await db.ping();
  res.json({ status: "ready" });
});
```

### Interview questions

**Q1. Liveness vs readiness?**  
Liveness checks if the process should be restarted. Readiness checks if it should receive traffic.

**Q2. Should health checks require auth?**  
Usually internal health checks are network-restricted instead of user-authenticated.

---

## 12.2 Metrics and tracing

### Definition and info

Metrics are numeric measurements. Tracing follows a request through services.

### Use case

Understand latency, errors, bottlenecks, and dependency failures.

### Production use case

Track RED metrics: Rate, Errors, Duration. Also track saturation such as CPU, memory, DB pool usage, queue depth.

### Example metrics

```txt
http_requests_total
http_request_duration_seconds
queue_jobs_waiting_total
database_query_duration_seconds
```

### Interview questions

**Q1. What metrics matter for APIs?**  
Request rate, error rate, latency percentiles, throughput, dependency latency, DB pool usage, and queue lag.

**Q2. Why percentiles instead of averages?**  
Averages hide tail latency. p95/p99 show what slower users experience.

---

# 13. Deployment, scaling, and reliability

## 13.1 Horizontal scaling

### Definition and info

Horizontal scaling means running more app instances. Vertical scaling means giving one instance more CPU/RAM.

### Use case

Handle more traffic and improve availability.

### Production use case

Stateless app instances are easier to scale. Store sessions/cache/queues in external shared systems.

### Example

```txt
Load Balancer
  -> API instance 1
  -> API instance 2
  -> API instance 3
```

### Interview questions

**Q1. Why should API servers be stateless?**  
So any instance can handle any request, making load balancing and scaling easier.

**Q2. What state should be externalized?**  
Sessions, cache, uploads, queues, jobs, locks, and database state.

---

## 13.2 Graceful shutdown

### Definition and info

Graceful shutdown stops accepting new requests, lets active requests finish, closes resources, and exits cleanly.

### Use case

Deployments, autoscaling, crashes, restarts.

### Production use case

Without graceful shutdown, deployments can drop in-flight requests or corrupt job processing.

### Example

```py
# FastAPI resource cleanup is commonly handled in lifespan.
@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.db = create_pool()
    yield
    await app.state.db.close()
```

### Interview questions

**Q1. Why is graceful shutdown needed?**  
To avoid cutting off active requests and to close DB/queue connections safely.

**Q2. What signals usually trigger shutdown?**  
`SIGTERM` and `SIGINT`.

---

## 13.3 Load balancing

### Definition and info

A load balancer distributes traffic across backend instances.

### Use case

Improve throughput and availability.

### Production use case

Use health checks, timeouts, TLS termination, sticky sessions only when necessary, and proper proxy headers.

### Example

```txt
Client -> CDN/API Gateway -> Load Balancer -> API containers -> DB/Cache/Queue
```

### Interview questions

**Q1. What is sticky session?**  
A load balancer strategy that sends a user to the same backend instance. It is often avoided by externalizing session state.

**Q2. What happens if one backend instance dies?**  
The load balancer should stop routing traffic to it after failed health checks.

---

# 14. API styles beyond REST

## 14.1 GraphQL

### Definition and info

GraphQL lets clients request exactly the fields they need from a typed schema.

### Use case

Complex frontend data requirements, mobile clients, graph-like relationships.

### Production use case

GraphQL needs query complexity limits, auth at field/resource level, caching strategy, and N+1 prevention with DataLoader.

### Example query

```graphql
query {
  user(id: "123") {
    id
    name
    posts {
      title
    }
  }
}
```

### Interview questions

**Q1. REST vs GraphQL?**  
REST exposes resource endpoints. GraphQL exposes a schema and lets clients select fields.

**Q2. What is a GraphQL N+1 problem?**  
Resolvers trigger repeated DB queries for nested fields unless batched/cached.

---

## 14.2 gRPC

### Definition and info

gRPC is a high-performance RPC framework using Protocol Buffers and HTTP/2.

### Use case

Service-to-service communication with strict contracts and low latency.

### Production use case

Common in microservices, especially internal APIs.

### Example proto

```proto
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
}
```

### Interview questions

**Q1. When choose gRPC over REST?**  
For internal service-to-service calls needing strong contracts, streaming, and performance.

**Q2. Why is REST often easier for public APIs?**  
HTTP/JSON is widely understood, easy to test, and browser/tooling friendly.

---

## 14.3 WebSockets and Server-Sent Events

### Definition and info

WebSockets provide bidirectional persistent communication. Server-Sent Events provide server-to-client streaming over HTTP.

### Use case

Chat, notifications, live dashboards, collaboration, game state, AI streaming.

### Production use case

Requires connection management, horizontal scaling, pub/sub, heartbeats, and backpressure handling.

### Example

```txt
WebSocket: client <-> server
SSE: client <- server
```

### Interview questions

**Q1. WebSocket vs SSE?**  
WebSocket is bidirectional. SSE is one-way server-to-client and simpler for streaming updates.

**Q2. What is hard about WebSockets in production?**  
Connection scaling, reconnects, message ordering, auth refresh, and broadcasting across instances.

---

# 15. Practical production backend checklist

## 15.1 API checklist

- Clear route naming.
- Versioning strategy.
- Input validation.
- Response models.
- Consistent errors.
- Pagination limits.
- Idempotency for unsafe retries.
- Request IDs.
- OpenAPI docs where useful.

## 15.2 Security checklist

- Server-side auth checks.
- Authorization on every protected resource.
- Strong password hashing.
- Secure cookies/JWT handling.
- CORS restrictions.
- CSRF protection when using cookies.
- Rate limits.
- Security headers.
- Secret management.
- Dependency scanning.
- No stack traces in responses.

## 15.3 Database checklist

- Migrations.
- Indexes.
- Constraints.
- Transactions.
- Backups.
- Connection pooling.
- Slow query monitoring.
- Rollback plan.
- Seed/test data separation.

## 15.4 Reliability checklist

- Health checks.
- Graceful shutdown.
- Timeouts.
- Retries with backoff.
- Circuit breakers for dependencies.
- Queue DLQ.
- Idempotent jobs.
- Metrics/logs/traces.
- Alerting.

## 15.5 Deployment checklist

- Environment variables validated at startup.
- Docker image or reproducible build.
- CI tests.
- DB migrations controlled.
- Rollback strategy.
- Horizontal scaling support.
- Reverse proxy/load balancer config.
- TLS.
- Static asset/file storage externalized.

---

# 16. Common backend interview questions: final revision set

## Q1. Explain a typical backend request lifecycle.

A client sends an HTTP request. The request reaches a load balancer or gateway, then the backend app. Middleware handles request ID, logging, CORS, parsing, auth, and rate limits. The router maps the request to a controller. The controller validates input and calls the service layer. The service layer enforces business rules, authorization, transactions, repository calls, cache, queue jobs, or external APIs. The controller returns a structured response. Logs, metrics, and traces are emitted throughout.

## Q2. How would you design a production CRUD API?

Use resource-oriented routes, DTO validation, response models, service layer, repository layer, database constraints, pagination, consistent errors, auth checks, request IDs, structured logs, tests, migrations, and health checks.

## Q3. How do you handle slow tasks in a backend?

Do not block the request. Create a background job in a durable queue, return `202 Accepted` with a job ID, process asynchronously, store status, retry on failure, and use DLQ after repeated failures.

## Q4. How do you prevent duplicate payments/orders?

Use idempotency keys. Store the key and result in a transaction. If the same key is retried, return the existing result instead of creating a duplicate side effect.

## Q5. How do you debug high API latency?

Break latency into request parsing, auth, service logic, database queries, cache hit/miss, external API calls, serialization, and network. Use logs, metrics, traces, and database query plans.

## Q6. How do you secure an API?

Validate input, authenticate users, authorize every protected resource, use secure cookies/tokens, rate-limit sensitive routes, use TLS, set security headers, prevent injection, avoid leaking stack traces, store secrets safely, and monitor abuse.

## Q7. Express.js vs FastAPI: which one should you choose?

Choose Express.js for JS/TS teams, Node ecosystem, BFFs, lightweight APIs, and flexibility. Choose FastAPI for Python teams, ML/data APIs, automatic validation/docs, and type-driven backend development. The better choice depends on team skills, ecosystem needs, deployment constraints, and product domain.

## Q8. What is the biggest mistake juniors make in backend design?

Putting everything in route handlers: validation, database calls, business logic, auth, error formatting, and external integrations. This works for demos but becomes unmaintainable in production.

## Q9. What is the best general backend architecture for a startup MVP?

A modular monolith is usually better than microservices. Use one backend codebase with clear modules, a relational database, Redis if needed, a queue for background jobs, strong observability, and clean boundaries. Split services only when there is real scaling or ownership pressure.

## Q10. What is the difference between a modular monolith and microservices?

A modular monolith is one deployable application with internal module boundaries. Microservices are multiple independently deployed services communicating over the network. Microservices add operational complexity and should not be the default for early products.

---

# 17. Recommended study order

1. HTTP methods/status codes/errors.
2. Backend layered architecture.
3. Express routing/middleware/error handling.
4. FastAPI path operations/Pydantic/dependencies.
5. Validation and DTOs.
6. AuthN/AuthZ.
7. SQL, indexes, transactions, migrations.
8. Caching and Redis.
9. Queues and workers.
10. Observability.
11. Security.
12. Deployment and scaling.
13. Testing.
14. REST vs GraphQL vs gRPC vs WebSockets.

---

# 18. Express.js mini project architecture example

```txt
src/
  app.ts
  server.ts
  config/
    env.ts
  db/
    prisma.ts
  middleware/
    auth.ts
    error-handler.ts
    request-id.ts
    rate-limit.ts
  modules/
    auth/
      auth.routes.ts
      auth.controller.ts
      auth.service.ts
      auth.schema.ts
    users/
      users.routes.ts
      users.controller.ts
      users.service.ts
      users.repository.ts
      users.schema.ts
  shared/
    errors.ts
    logger.ts
    result.ts
  tests/
```

## Example flow: create user

```txt
POST /api/v1/users
  -> request-id middleware
  -> JSON parser
  -> rate limiter
  -> validateBody(createUserSchema)
  -> createUserController
  -> userService.createUser
  -> userRepository.findByEmail
  -> hash password
  -> userRepository.create
  -> response 201
```

---

# 19. FastAPI mini project architecture example

```txt
app/
  main.py
  core/
    config.py
    security.py
    logging.py
  db/
    session.py
    models.py
  middleware/
    request_id.py
  modules/
    auth/
      router.py
      schemas.py
      service.py
    users/
      router.py
      schemas.py
      service.py
      repository.py
  dependencies/
    auth.py
    database.py
  tests/
```

## Example flow: create user

```txt
POST /api/v1/users
  -> ASGI server
  -> middleware
  -> router match
  -> Pydantic request validation
  -> dependency resolution
  -> endpoint function
  -> user_service.create_user
  -> repository/database
  -> response_model filtering
  -> JSON response
```

---

# 20. Final production recommendation

For interviews and real projects, explain backend systems like this:

```txt
Client
  -> CDN / API Gateway / Load Balancer
  -> Backend route/controller
  -> Middleware: auth, logging, rate limit, CORS
  -> Validation schema / DTO
  -> Service/business logic
  -> Authorization policy
  -> Repository/data-access layer
  -> Database/cache/queue/external API
  -> Structured response/error
  -> Logs/metrics/traces
```

For a startup/MVP, prefer:

- Modular monolith.
- PostgreSQL as primary database.
- Redis only when needed.
- One queue for background jobs.
- Strong validation and auth from day one.
- Clean service/repository boundaries.
- Observability early, not after things break.

Avoid prematurely splitting into microservices. The main bottleneck early is usually unclear boundaries, weak validation, missing tests, poor database design, and no operational visibility — not the absence of microservices.

---

# 21. Primary references used

- Express.js official site: https://expressjs.com/
- Express routing guide: https://expressjs.com/en/guide/routing.html
- Express middleware guide: https://expressjs.com/en/guide/using-middleware.html
- Express error handling guide: https://expressjs.com/en/guide/error-handling.html
- Express production security best practices: https://expressjs.com/en/advanced/best-practice-security.html
- Express production performance best practices: https://expressjs.com/en/advanced/best-practice-performance.html
- FastAPI official docs: https://fastapi.tiangolo.com/
- FastAPI tutorial: https://fastapi.tiangolo.com/tutorial/
- FastAPI dependencies: https://fastapi.tiangolo.com/tutorial/dependencies/
- FastAPI security: https://fastapi.tiangolo.com/tutorial/security/
- FastAPI SQL databases: https://fastapi.tiangolo.com/tutorial/sql-databases/
- FastAPI middleware: https://fastapi.tiangolo.com/tutorial/middleware/
- FastAPI background tasks: https://fastapi.tiangolo.com/tutorial/background-tasks/
- FastAPI lifespan events: https://fastapi.tiangolo.com/advanced/events/
- Node.js event loop guide: https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick
- Node.js non-blocking/event-loop guidance: https://nodejs.org/learn/asynchronous-work/dont-block-the-event-loop
- SQLAlchemy ORM/session docs: https://docs.sqlalchemy.org/en/latest/orm/session.html
