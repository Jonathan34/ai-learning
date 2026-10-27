# 37 — Applied Coding and System Design Practice

These exercises combine practical application code with senior system design.
Several are simplified adaptations of the public
[Hack2Hire OpenAI coding list](https://www.hack2hire.com/question-bank/companies/openai/coding-questions)
and [system design list](https://www.hack2hire.com/question-bank/companies/openai/system-design).
Treat those lists as practice material, not proof that a specific question
will appear in an interview.

The original prompts sometimes use specialist business language. The versions
here state the engineering problem in ordinary words and define any domain
term that remains.

## How to run a coding exercise

Use 45 minutes:

```text
0-5     restate the problem and ask about constraints
5-10    work two examples and choose data structures
10-30   build the smallest correct version
30-38   test normal, boundary, and failure cases
38-45   explain complexity and production extensions
```

Say what you are trading off. A correct solution with no explanation is weaker
than a simple solution whose behavior and limits are clear.

Use Python for the first pass. Rebuild C7 or C9 in TypeScript so you also
practice promises, streaming events, typed unions, and error handling in a
common API-client environment.

## Coding exercises

### C1. Fault-tolerant work queue

**Problem:** Build an in-memory queue. A worker takes one job and gets a short
lease. If it finishes, it acknowledges the job. If it crashes and the lease
expires, another worker may retry it. Stop retrying after a limit.

**Walkthrough:**

1. Store each job as `pending`, `leased`, `done`, or `dead`.
2. Keep pending job IDs in a queue.
3. Keep lease deadlines in a min-heap, ordered by earliest deadline, so
   expired work is cheap to find.
4. Return a unique lease token with each claimed job.
5. Accept completion only when the token matches the active lease.
6. Put expired work back in the queue or in a failed-work list after too many
   attempts.
7. Protect claim and acknowledge with one lock.

**Test:** empty queue, worker crash, late acknowledgement, duplicate
acknowledgement, retry limit, and two workers claiming at the same time.

**What to discuss:** this gives at-least-once delivery. The job handler must be
idempotent because a job can run more than once.

### C2. Spread through a grid

**Problem:** A grid contains healthy cells, infected cells, and walls. Every
minute, infection moves from an infected cell to its healthy up, down, left,
and right neighbors. Return the time needed to reach every reachable healthy
cell, or `-1` if some can never be reached.

**Walkthrough:** Put every initially infected cell in one queue with time
zero. Run breadth-first search in layers. Count healthy cells as they become
infected. If any remain after the queue is empty, return `-1`.

**Complexity:** `O(rows × columns)` time and space.

**Test:** empty grid, no infected cells, all infected, walls, several starting
points, and unreachable cells.

### C3. Small language parser

**Problem:** Parse expressions in this tiny language:

```text
expression := word | "(" expression "," expression ")"
```

Valid examples are `cat` and `(cat,(dog,bird))`. Return a tree or a clear
syntax error.

**Walkthrough:** First split the input into words and punctuation. Then write a
recursive `parse_expression` function. A word becomes a leaf. `(` starts a
pair and must be followed by an expression, comma, expression, and `)`.
Finally, reject any tokens left after the root expression.

**What to discuss:** error positions, recursion-depth limits, iterative
parsing for untrusted input, and separating tokenization from grammar.

### C4. Time-based compute credits

**Problem:** Accounts receive compute credits that become valid at a start
time and expire later. Jobs spend credits at a given time. Implement
`grant`, `spend`, and `balance_at`.

**Walkthrough:** Store grants and spends as immutable events. For
`balance_at(t)`, include grants whose time window contains `t` and spends at
or before `t`. Spend the credits that expire soonest first. Reject a spend
that would make the balance negative.

**What to discuss:** storing amounts as whole credit units rather than
floating-point values, event ordering, request IDs that prevent duplicate
spending, concurrent spends, and why a database transaction or ledger is
needed in production.

### C5. Rectangle board

**Problem:** Build a board that can add, move, remove, and list
axis-aligned rectangles. Rectangles may not overlap or leave the board.

**Walkthrough:** Give each rectangle an ID and four coordinates. Centralize
boundary and overlap checks. A move first validates the proposed coordinates,
then changes the stored rectangle. Start with an `O(n)` overlap scan; introduce
a spatial index only if the constraints require it.

**Test:** touching edges, complete containment, identical rectangles, moving a
rectangle over itself, and coordinates on the board boundary.

### C6. Slow comparison service

**Problem:** An asynchronous service tells you whether a guess is too low, too
high, or correct. Calls may be slow or time out. Find a hidden number in a
known range while respecting a total deadline.

**Walkthrough:** Use binary search because each response halves the remaining
range. Put a timeout around each call. Retry only transient failures and never
past the total deadline. Cancel outstanding work when the number is found.

**What to discuss:** a speculative parallel search uses more requests and may
not reduce the critical path because the next useful guess depends on the
previous answer.

### C7. Chat event aggregation

**Problem:** Given message-created, message-edited, message-deleted, and
reaction events, produce the current message view and reaction counts. Events
can be duplicated or arrive out of order.

**Walkthrough:** Deduplicate by event ID. Group by message ID. Apply events in
logical version order, not arrival order. A delete hides the message but does
not erase the event history. Reactions use their own IDs so a repeated add or
remove is harmless.

**What to discuss:** append-only logs, replay, snapshots, retention, and what
happens when an edit arrives after a delete.

### C8. Data-labeling task scheduler

**Problem:** Assign labeling tasks to workers. Tasks have a priority and
required skill. A task must not be assigned to two workers at once. Abandoned
tasks should become available again.

**Walkthrough:** Keep a priority queue per skill. Claim work with a lease and a
conditional state change. Renew long leases. On expiry, return the task to its
queue. Track attempts and move repeatedly failing tasks to review.

**What to discuss:** fairness, worker quality, agreement between labelers,
gold-standard check tasks, and separating scheduling from label quality.

### C9. Bounded tool-calling loop

**Problem:** A model may return a final answer or ask your program to call one
or more tools. Build the loop.

**Walkthrough:**

1. Call the model with the user request and available tool schemas.
2. If the response is final, return it.
3. Validate every tool name and argument.
4. Run independent read calls concurrently.
5. Require approval before side effects.
6. Return compact results or errors to the model.
7. Stop after a turn, time, and cost budget.

**Test:** unknown tool, invalid arguments, tool timeout, duplicate write,
parallel reads, refusal, and a model that never stops.

### C10. Small evaluation gate

**Problem:** Given a dataset, a baseline function, and a candidate function,
decide whether the candidate may ship.

**Walkthrough:** Each case has an input, expected properties, and one or more
graders. Run both versions, store per-case results, compare the overall score
and important failure categories, then block a candidate that crosses a
threshold or introduces a release-blocking failure.

**What to discuss:** deterministic graders, model graders calibrated against
people, paired comparisons, confidence intervals, and why one average can hide
a serious regression.

## Worked coding deep dive: the fault-tolerant queue

The hard part is ownership after a worker disappears. A plain `pop()` removes
the job before the system knows it finished. A lease makes ownership temporary.

### State

```text
pending → leased → done
             ↓ lease expires
          pending
             ↓ retry limit
            dead
```

Each claim returns `(job, lease_token, lease_deadline)`. The token prevents an
old worker from finishing a job after a new worker has claimed it.

```python
from collections import deque
from dataclasses import dataclass
from heapq import heappop, heappush
from threading import Lock
from uuid import uuid4


@dataclass
class Job:
    job_id: str
    payload: dict
    attempts: int = 0
    state: str = "pending"
    lease_token: str | None = None
    lease_deadline: float | None = None


class WorkQueue:
    def __init__(self, max_attempts=3):
        self.jobs = {}
        self.pending = deque()
        self.expiring = []
        self.dead = []
        self.max_attempts = max_attempts
        self.lock = Lock()

    def add(self, job_id, payload):
        with self.lock:
            if job_id in self.jobs:
                return
            self.jobs[job_id] = Job(job_id, payload)
            self.pending.append(job_id)

    def _expire(self, now):
        while self.expiring and self.expiring[0][0] <= now:
            deadline, job_id, token = heappop(self.expiring)
            job = self.jobs[job_id]
            if job.state != "leased" or job.lease_token != token:
                continue
            job.lease_token = None
            job.lease_deadline = None
            if job.attempts >= self.max_attempts:
                job.state = "dead"
                self.dead.append(job_id)
            else:
                job.state = "pending"
                self.pending.append(job_id)

    def claim(self, now, lease_seconds):
        with self.lock:
            self._expire(now)
            if not self.pending:
                return None
            job = self.jobs[self.pending.popleft()]
            token = str(uuid4())
            job.state = "leased"
            job.attempts += 1
            job.lease_token = token
            job.lease_deadline = now + lease_seconds
            heappush(
                self.expiring,
                (job.lease_deadline, job.job_id, token),
            )
            return job.job_id, job.payload, token, job.lease_deadline

    def acknowledge(self, job_id, token):
        with self.lock:
            job = self.jobs[job_id]
            if job.state != "leased" or job.lease_token != token:
                return False
            job.state = "done"
            job.lease_token = None
            job.lease_deadline = None
            return True
```

This version is intentionally in memory. A production design would use
durable storage, transactional claims, lease timeouts, metrics, and a separate
record that prevents repeated side effects. The state model stays the same.

## How to run a system design exercise

Use a consistent order:

```text
goal → users and scale → API and data → simplest architecture
     → failure and safety → measurement → rollout
```

Start with rough numbers. Then draw only the components needed to explain data
flow, ownership, and failure boundaries.

## System design exercises

### S1. Ask smart devices to temporarily use less power

**Problem:** During periods of unusually high electricity demand, send a
temporary power limit to several million smart devices within one minute.
Owners may refuse or override it. Measure how much demand actually changed.
Readings can arrive late. A stop command must work quickly.

Some versions of this prompt use uncommon terms:

| Term | Plain meaning |
|---|---|
| Curtail a device | Temporarily reduce how much power it uses |
| Shed load | Reduce total electricity demand |
| Cohort | A group of devices treated together |
| Holdout group | Similar devices left unchanged so we can measure what would have happened |
| Settlement feed | Final verified result sent to billing or rewards |
| Restatement policy | Rule for correcting a published result when late data arrives |

**Walkthrough:**

1. Keep three facts separate: command sent, device-reported result, and
   measured change in power use.
2. Send versioned desired state to each device through a scalable push system.
   Commands expire automatically and repeated delivery is safe.
3. Roll out in waves. Start with a small test group. Give owners an override
   and operators a fast stop command.
4. Put acknowledgements and readings in a durable event stream.
5. Measure actual reduction against a similar holdout group. An
   acknowledgement does not prove power use changed.
6. Maintain a fast live estimate for control and a slower, versioned final
   result for billing or rewards.
7. Allow corrections only during a documented window, then freeze the final
   version while retaining the raw evidence.

**Senior probes:** partial regional outage, bad command sent to millions of
devices, duplicate readings, devices that report success but do not change,
and a feedback loop that over-corrects before earlier commands take effect.

### S2. Distributed web crawler

**Problem:** Crawl a large part of the public web, avoid repeatedly fetching
the same pages, obey site limits, and make new content searchable quickly.

**Walkthrough:** A URL frontier assigns domains to workers. Per-domain queues
enforce crawl delay. Normalize and deduplicate URLs before scheduling. Fetchers
store content and emit discovered links to a durable stream. Separate parsing,
indexing, retries, and repeatedly failing work. Use content hashes to detect
changes.

**Senior probes:** robots rules, crawler traps, priority, regional failures,
very large sites, and recrawling based on change frequency.

### S3. ChatGPT-like application

**Problem:** Design a chat application that streams model responses, keeps
conversation history, supports tools and files, and serves many users.

**Walkthrough:** Authenticate at an API gateway. Store conversation metadata
separately from large files. An orchestration service builds context, calls
the model, executes tools through a restricted gateway, and streams events to
the client. Compact old history and retrieve files on demand. Trace every turn
and enforce token, time, and tool budgets.

**Senior probes:** rate limits, tenant isolation, retrying streamed requests,
duplicate tool actions, abuse, data retention, and model upgrades.

### S4. Sandboxed cloud coding environment

**Problem:** Let users run generated or hand-written code without giving that
code access to the host, other users, or unrestricted networks.

**Walkthrough:** An API creates short-lived jobs. A scheduler assigns them to
isolated sandboxes with CPU, memory, time, filesystem, and network limits.
Images are allow-listed and scanned. Inputs and outputs go through a separate
artifact service. Destroy the environment after the job and keep an audit
record.

**Senior probes:** secret handling, network allow-lists, fork bombs, image
cache poisoning, noisy neighbors, warm pools, and incident forensics.

### S5. GPU job scheduler

**Problem:** Schedule training and inference jobs across a shared GPU fleet.
Jobs request different GPU types and counts. Some need all GPUs at once.

**Walkthrough:** Keep an inventory of GPU type, memory, topology, health, and
allocation. Admit large distributed jobs only when the full group can start.
Use quotas and fair sharing across teams. Keep low-latency serving capacity
separate from interruptible batch work. Drain unhealthy nodes and checkpoint
preempted jobs.

**Senior probes:** fragmentation, topology-aware placement, reservations,
preemption, fairness, utilization, and avoiding a partially scheduled job
holding resources forever.

### S6. Reliable webhook delivery

**Problem:** Deliver customer events to their HTTP endpoints. Endpoints may be
slow, unavailable, or return errors. Customers must be able to verify and
replay events.

**Walkthrough:** Write each event to durable storage before returning success.
A queue fans deliveries out by endpoint. Sign each request. Retry transient
failures with exponential backoff and jitter. Keep a delivery ID so receivers
can deduplicate. Stop after a limit and expose replay from a failed-delivery
list.

**Senior probes:** ordering, per-customer rate limits, poison endpoints,
secret rotation, regional failure, retention, and preventing one customer
from blocking others.

### S7. Large team chat

**Problem:** Build channels, direct messages, online delivery, search, and
notifications for a large workplace chat product.

**Walkthrough:** Store each message durably before fan-out. Partition channel
history by workspace and channel. Use persistent connections for online users
and notification queues for offline users. Keep search indexing asynchronous.
Use message positions that always increase so a client can resume after
reconnecting.

**Senior probes:** very large channels, edits and deletes, unread counts,
multi-device state, presence, permission changes, and regional failover.

### S8. AI support workflow

**Problem:** Answer support questions, look up orders, and request refunds
while preventing unsupported promises or unauthorized actions.

**Walkthrough:** Route the request by intent and risk. Retrieve the current
policy. Use typed read tools for account and order data. Keep refunds behind a
deterministic policy check, idempotency key, and approval rule. Escalate when
evidence is missing or risk is high. Evaluate policy accuracy, action
correctness, escalation, latency, and cost.

Chapter 38 works this problem end to end.

## Choose problems by the gap you need to close

| If you need practice with | Start with |
|---|---|
| State and failure | C1, C7, S6 |
| Graphs and parsing | C2, C3 |
| Concurrency | C1, C6, C8 |
| AI application code | C9, C10 |
| General distributed systems | S1, S2, S7 |
| Platform safety | S4, S8 |
| Scheduling and capacity | C8, S5 |
| Customer-facing AI design | S3, S8 |

## What to write down after each attempt

- Which requirement did you miss?
- Which failure changed your design?
- Where did you add complexity before it was needed?
- Which number should have been estimated earlier?
- What would you test before committing to production?

## Go deeper

- [Practical Python for AI Infrastructure](../05-workshops/W11-python-ai-infrastructure.md)
- [Real Evaluation workshop](../05-workshops/W3-real-evaluation.md)
- [Tool-Using Agent workshop](../05-workshops/W4-tool-using-agent.md)
- [AI Platform Engineering](../03-production/16-ai-platform-engineering.md)
- [Hack2Hire OpenAI coding questions](https://www.hack2hire.com/question-bank/companies/openai/coding-questions)
- [Hack2Hire OpenAI system design questions](https://www.hack2hire.com/question-bank/companies/openai/system-design)
