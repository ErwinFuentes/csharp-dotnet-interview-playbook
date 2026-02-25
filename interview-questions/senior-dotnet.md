# Senior Backend C# Interview – Senior-Level Answers

---

## 1) What are the key differences between IEnumerable, IQueryable, and IAsyncEnumerable?

### What interviewers are looking for
- Understanding of execution model
- Expression trees knowledge
- EF Core translation behavior
- Performance awareness

### What a senior developer would answer

The key difference is **where and how the query executes**.

- `IEnumerable<T>` operates in-memory. If you call `.ToList()` before filtering, you’re pulling all data into memory and then filtering — which is dangerous with large datasets.
- `IQueryable<T>` builds an **expression tree**, not delegates. EF Core translates that tree into SQL. That means filtering, joins, projections happen at the database level.
- `IAsyncEnumerable<T>` allows streaming results asynchronously, which is useful for large datasets or I/O-bound operations to avoid buffering everything in memory.

In production, misuse of `IEnumerable` instead of `IQueryable` is a common performance issue. A senior developer is careful about where materialization happens.

---

## 2) Explain dependency injection in .NET and service lifetimes.

### What interviewers are looking for
- Deep lifecycle understanding
- Thread-safety awareness
- Architectural reasoning

### What a senior developer would answer

DI is not just about injecting services — it's about **controlling object lifetime and coupling**.

- Transient: new instance every time.
- Scoped: per request (safe for DbContext).
- Singleton: one for entire app lifetime — must be thread-safe.

The most common production bug is injecting a Scoped service into a Singleton, causing runtime failures.

In high-load systems, Singleton services must avoid shared mutable state unless properly synchronized.

A senior developer designs services with lifetime in mind, especially for background workers and hosted services.

---

## 3) What are async/await pitfalls in C#?

### What interviewers are looking for
- Thread pool understanding
- Deadlock awareness
- Real-world async mistakes

### What a senior developer would answer

The biggest mistake is blocking async code with `.Result` or `.Wait()`, which can cause thread starvation or deadlocks.

In ASP.NET Core there’s no SynchronizationContext, so deadlocks are less common — but thread pool starvation is still real under load.

Other pitfalls:
- Using `async void` (exceptions crash the process).
- Overusing `Task.Run` in web apps (wastes threads).
- Not distinguishing CPU-bound vs I/O-bound work.

A senior developer thinks in terms of scalability: async improves throughput, not speed.

---

## 4) Explain SOLID principles with backend examples.

### What interviewers are looking for
- Practical understanding
- Trade-off awareness
- Maintainability mindset

### What a senior developer would answer

SOLID is about maintainability at scale.

For example:

- SRP: A service that handles validation, persistence, and messaging is a maintenance nightmare.
- OCP: Instead of modifying payment logic, introduce a strategy pattern.
- DIP: Controllers depend on abstractions, not concrete EF implementations.

But overengineering is also bad. A senior knows when NOT to apply abstractions too early.

---

## 5) How does middleware work in ASP.NET Core?

### What interviewers are looking for
- Pipeline understanding
- Ordering importance
- Performance impact

### What a senior developer would answer

Middleware is a chain of delegates forming a request pipeline.

Order matters:
- Exception handling first
- Auth before authorization
- Endpoints last

Each middleware can short-circuit the pipeline.

Poor ordering can cause security or performance issues.

In high-performance systems, unnecessary middleware adds latency, so minimal pipelines are preferred.

---

## 6) How do you handle concurrency in backend systems?

### What interviewers are looking for
- Real distributed system knowledge
- Race condition mitigation
- Scalability awareness

### What a senior developer would answer

Concurrency isn't just about threads — it’s about distributed systems.

Techniques:
- Optimistic concurrency (RowVersion) for most cases.
- Pessimistic locking only when necessary.
- Idempotency keys for APIs.
- Distributed locks for multi-instance systems.
- Avoid shared state.

In microservices, eventual consistency is often preferable to distributed transactions.

A senior designs for horizontal scaling from day one.

---

## 7) Task vs ValueTask vs Thread?

### What interviewers are looking for
- Runtime knowledge
- Allocation awareness
- Performance reasoning

### What a senior developer would answer

- Thread = OS resource (expensive).
- Task = abstraction over async work (uses thread pool).
- ValueTask reduces allocations when results are frequently synchronous.

However, `ValueTask` complicates consumption and should only be used in hot paths proven by profiling.

Premature optimization is worse than a small allocation.

---

## 8) How would you improve performance in a high-traffic API?

### What interviewers are looking for
- Systematic thinking
- Measurement-first mindset
- Multi-layer optimization

### What a senior developer would answer

First: measure.

Then optimize:

- Database indexing and query analysis.
- Avoid N+1 queries.
- Introduce caching (Redis).
- Reduce payload size.
- Use async I/O.
- Enable response compression.
- Scale horizontally.

Performance tuning without metrics is guessing.

---

## 9) record vs class?

### What interviewers are looking for
- Equality semantics
- Immutability benefits
- DDD awareness

### What a senior developer would answer

`record` gives value-based equality and immutability by default, which is ideal for DTOs and domain value objects.

`class` is better when identity and lifecycle matter (entities).

Immutability reduces bugs in concurrent systems.

---

## 10) How would you design a scalable microservices backend?

### What interviewers are looking for
- System design depth
- Trade-off awareness
- Operational maturity

### What a senior developer would answer

First question: do we really need microservices?

If yes:

- Define bounded contexts.
- Independent databases per service.
- Event-driven communication.
- Use message broker.
- API Gateway for routing.
- Observability (logs, metrics, tracing).
- Circuit breakers and retries.

Avoid distributed transactions — use Saga pattern.

Microservices increase operational complexity, so invest heavily in monitoring and CI/CD automation.

A senior developer understands that architecture is about trade-offs, not trends.

---
