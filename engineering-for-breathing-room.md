# Engineering for Breathing Room

Source: https://rfc.earendil.com/0020/

**Goal:** Controllable failure, not perfect uptime. If something breaks at 2 AM, you can stabilize fast and fix it properly later.

**Breathing room means:**
- Doing nothing for ten minutes keeps blast radius bounded
- You have explicit controls to stop things getting worse
- Recovery is clear, reversible, and safe under pressure

---

## Principles

1. **Start dumb.** One queue, one write path. Add complexity only when production signals demand it. Prefer writing it yourself badly over pulling in a complex dependency.

2. **Design data before abstractions.** Make access patterns, growth, and hotspot risks explicit early. No framework fixes bad keys, fanout, or unbounded joins.

3. **Prefer event-based state transitions.** Represent significant state changes as explicit events. Keep replay and recovery possible.

4. **Wall-clock assumptions are unsafe.** Queues stall, jobs drift. Correctness must rely on event timestamps and transition logic, not scheduler punctuality.

5. **Design writes for concurrency.** "Usually non-conflicting" is not a strategy. Use locks, atomic updates, idempotency keys, and durable retries.

6. **Rollouts are non-atomic.** Old and new behavior coexist during rollout windows. Safe sequence: migrate storage, deploy tolerant code, then rely on new semantics.

7. **Transform data once at boundaries.** Normalize at ingress, serialize at egress. Keep business logic on stable domain shapes.

8. **Observability is a functional requirement.** Minimum: structured logs, correlation IDs, latency visibility, retry/failure/dead-letter counters.

9. **Crash safely.** Fail fast on broken invariants. Restart from durable state. Make retry paths idempotent. Silent corruption is worse than visible failure.

10. **Finish migrations.** Every migration needs an owner, an end state, and a deadline. Done means deleting old paths.

---

## Stabilization Knobs

Every critical system needs operator controls that work without code changes:
- Feature flags, pause queue consumers, disable mutating writes
- Route to degraded read paths, open circuit breakers, divert to DLQ
- Stop scheduler triggers without losing accepted work

Document, test, and rehearse these.

---

## Queue / QoS

Queue depth is not customer value. During disruptions:
- Prioritize current user impact over backlog
- Use TTLs, load shedding, or tail dropping for stale tasks
- Prefer explicit replay over accidental retry storms

---

## Operational Baseline

- Runbooks have concrete "stabilize now" steps
- Alerts tied to user impact, not internal noise
- If an alert keeps firing, it's a bad alert
- Rollout plans model mixed-version coexistence
