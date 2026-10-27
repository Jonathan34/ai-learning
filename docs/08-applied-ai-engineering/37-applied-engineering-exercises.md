# 37 — Applied Coding and System Design Practice

These exercises combine practical application code with senior system design.
Several are simplified adaptations of the public
[Hack2Hire OpenAI coding list](https://www.hack2hire.com/question-bank/companies/openai/coding-questions)
and [system design list](https://www.hack2hire.com/question-bank/companies/openai/system-design).
The lists are starting points for skill practice rather than a required set of
questions.

The original prompts sometimes use specialist business language. The versions
here state the engineering problem in ordinary words and define any domain
term that remains.

Use these exercises to become proficient in practical coding and system
design. They can also help with job interviews if you need that practice.

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

Each reference solution is collapsed. Try the exercise first, then compare the
reasoning, tests, and trade-offs instead of memorizing the code.

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

<!-- markdownlint-disable MD046 -->

??? note "Reference solution"

    The hard part is ownership after a worker disappears. A plain `pop()`
    removes the job before the system knows it finished. A lease makes
    ownership temporary.

    ```text
    pending → leased → done
                 ↓ lease expires
              pending
                 ↓ retry limit
                dead
    ```

    Each claim returns `(job, lease_token, lease_deadline)`. The token stops
    an old worker from completing a job after another worker has claimed it.

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
                _, job_id, token = heappop(self.expiring)
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


    if __name__ == "__main__":
        queue = WorkQueue(max_attempts=2)
        queue.add("image-42", {"file": "photo.png"})

        first_claim = queue.claim(now=0, lease_seconds=5)
        retry = queue.claim(now=6, lease_seconds=5)

        stale_result = queue.acknowledge("image-42", first_claim[2])
        completed = queue.acknowledge("image-42", retry[2])

        assert retry[0] == "image-42"
        assert stale_result is False
        assert completed is True
        print("Retried job:", retry[0])
        print("Old lease accepted:", stale_result)
        print("Final state:", queue.jobs["image-42"].state)
    ```

    Sample output:

    ```text
    Retried job: image-42
    Old lease accepted: False
    Final state: done
    ```

    Queue operations are `O(1)` except heap work, which is `O(log n)`. This
    version is intentionally in memory. A production version needs durable
    storage, transactional claims, metrics, and a separate record that
    prevents repeated side effects.

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

??? note "Reference solution"

    This is a multi-source breadth-first search. Starting with every infected
    cell matters: one queue layer then represents one minute across the whole
    grid, not one minute per starting point.

    This version uses `0` for a wall, `1` for healthy, and `2` for infected.

    ```python
    from collections import deque


    def spread_time(grid):
        if not grid:
            return 0

        rows = len(grid)
        columns = len(grid[0])
        if any(len(row) != columns for row in grid):
            raise ValueError("grid must be rectangular")

        queue = deque()
        healthy = 0

        for row in range(rows):
            for column in range(columns):
                value = grid[row][column]
                if value == 2:
                    queue.append((row, column))
                elif value == 1:
                    healthy += 1
                elif value != 0:
                    raise ValueError(f"invalid cell value: {value}")

        minutes = 0
        directions = ((1, 0), (-1, 0), (0, 1), (0, -1))

        while queue and healthy:
            for _ in range(len(queue)):
                row, column = queue.popleft()
                for row_step, column_step in directions:
                    next_row = row + row_step
                    next_column = column + column_step
                    inside = (
                        0 <= next_row < rows
                        and 0 <= next_column < columns
                    )
                    if inside and grid[next_row][next_column] == 1:
                        grid[next_row][next_column] = 2
                        healthy -= 1
                        queue.append((next_row, next_column))
            minutes += 1

        return minutes if healthy == 0 else -1


    if __name__ == "__main__":
        sample = [
            [2, 1, 1],
            [1, 1, 0],
            [0, 1, 2],
        ]
        result = spread_time(sample)

        assert result == 2
        print("Minutes to infect every reachable cell:", result)
    ```

    Sample output:

    ```text
    Minutes to infect every reachable cell: 2
    ```

    Each cell enters the queue at most once, so time and space are
    `O(rows × columns)`. The function changes the grid to mark visited cells.
    Copy the grid first when callers expect their input to remain unchanged.

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

??? note "Reference solution"

    Keep tokenization and parsing separate. The tokenizer answers, "What are
    the pieces?" The parser answers, "Are those pieces in a valid order?"
    Keeping positions on tokens makes errors useful.

    ```python
    def tokenize(text):
        tokens = []
        index = 0

        while index < len(text):
            character = text[index]

            if character.isspace():
                index += 1
                continue

            punctuation = {
                "(": "LEFT_PAREN",
                ")": "RIGHT_PAREN",
                ",": "COMMA",
            }
            if character in punctuation:
                tokens.append((punctuation[character], character, index))
                index += 1
                continue

            if character.isalnum() or character == "_":
                start = index
                while index < len(text):
                    character = text[index]
                    if not (character.isalnum() or character == "_"):
                        break
                    index += 1
                tokens.append(("WORD", text[start:index], start))
                continue

            raise SyntaxError(
                f"unexpected character {character!r} at position {index}"
            )

        tokens.append(("END", "", len(text)))
        return tokens


    class Parser:
        def __init__(self, tokens):
            self.tokens = tokens
            self.index = 0

        def current(self):
            return self.tokens[self.index]

        def consume(self, expected):
            kind, value, position = self.current()
            if kind != expected:
                raise SyntaxError(
                    f"expected {expected} at position {position}, got {kind}"
                )
            self.index += 1
            return value

        def parse_expression(self):
            kind, value, position = self.current()

            if kind == "WORD":
                self.index += 1
                return ("word", value)

            if kind == "LEFT_PAREN":
                self.index += 1
                left = self.parse_expression()
                self.consume("COMMA")
                right = self.parse_expression()
                self.consume("RIGHT_PAREN")
                return ("pair", left, right)

            raise SyntaxError(
                f"expected an expression at position {position}, got {kind}"
            )


    def parse(text):
        parser = Parser(tokenize(text))
        tree = parser.parse_expression()
        parser.consume("END")
        return tree


    if __name__ == "__main__":
        expression = "(cat,(dog,bird))"
        tree = parse(expression)

        assert tree[0] == "pair"
        assert tree[2][1] == ("word", "dog")
        print(tree)
    ```

    Sample output:

    ```text
    ('pair', ('word', 'cat'), ('pair', ('word', 'dog'), ('word', 'bird')))
    ```

    Tokenization and parsing are both `O(n)`. The call stack is
    `O(nesting depth)`. For untrusted input, limit both input length and
    nesting depth or replace recursion with an explicit stack.

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

??? note "Reference solution"

    Treat each grant and spend as an immutable ledger event. To answer a
    historical query, replay spends in time order and take credits from the
    active grant that expires first. The end of a grant window is exclusive:
    a grant with `end=20` cannot be used at time `20`.

    ```python
    from dataclasses import dataclass


    @dataclass(frozen=True)
    class Grant:
        grant_id: str
        amount: int
        start: int
        end: int


    @dataclass(frozen=True)
    class Spend:
        spend_id: str
        amount: int
        time: int


    class CreditLedger:
        def __init__(self):
            self.grants = {}
            self.spends = {}

        def grant(self, grant_id, amount, start, end):
            if amount <= 0 or start >= end:
                raise ValueError("grant amount and time window are invalid")

            event = Grant(grant_id, amount, start, end)
            previous = self.grants.get(grant_id)
            if previous is not None and previous != event:
                raise ValueError("grant ID was already used")
            self.grants[grant_id] = event

        def spend(self, spend_id, amount, time):
            if amount <= 0:
                raise ValueError("spend amount must be positive")

            event = Spend(spend_id, amount, time)
            previous = self.spends.get(spend_id)
            if previous is not None:
                if previous != event:
                    raise ValueError("spend ID was already used")
                return

            self.spends[spend_id] = event
            try:
                self._replay(float("inf"))
            except ValueError:
                del self.spends[spend_id]
                raise

        def balance_at(self, time):
            remaining = self._replay(time)
            return sum(
                remaining[grant.grant_id]
                for grant in self.grants.values()
                if grant.start <= time < grant.end
            )

        def _replay(self, until):
            remaining = {
                grant_id: grant.amount
                for grant_id, grant in self.grants.items()
            }
            events = sorted(
                (
                    spend for spend in self.spends.values()
                    if spend.time <= until
                ),
                key=lambda spend: (spend.time, spend.spend_id),
            )

            for spend in events:
                amount_left = spend.amount
                usable = sorted(
                    (
                        grant for grant in self.grants.values()
                        if grant.start <= spend.time < grant.end
                        and remaining[grant.grant_id] > 0
                    ),
                    key=lambda grant: (
                        grant.end,
                        grant.start,
                        grant.grant_id,
                    ),
                )

                for grant in usable:
                    taken = min(
                        amount_left,
                        remaining[grant.grant_id],
                    )
                    remaining[grant.grant_id] -= taken
                    amount_left -= taken
                    if amount_left == 0:
                        break

                if amount_left:
                    raise ValueError(
                        f"insufficient credits for spend {spend.spend_id}"
                    )

            return remaining


    if __name__ == "__main__":
        ledger = CreditLedger()
        ledger.grant("A", amount=10, start=0, end=20)
        ledger.grant("B", amount=10, start=0, end=30)
        ledger.spend("job-1", amount=12, time=15)

        at_15 = ledger.balance_at(15)
        at_25 = ledger.balance_at(25)

        assert at_15 == 8
        assert at_25 == 8
        print("Balance at time 15:", at_15)
        print("Balance at time 25:", at_25)
    ```

    Sample output:

    ```text
    Balance at time 15: 8
    Balance at time 25: 8
    ```

    Example: grant A has 10 credits and expires at 20; grant B has 10 and
    expires at 30. A spend of 12 at time 15 consumes all 10 from A and 2 from
    B. This replay is deliberately simple and can be slow as the ledger grows.
    A production service would use database transactions, unique request IDs,
    an append-only ledger, and materialized balances or periodic snapshots.

### C5. Rectangle board

**Problem:** Build a board that can add, move, remove, and list
axis-aligned rectangles. Rectangles may not overlap or leave the board.

**Walkthrough:** Give each rectangle an ID and four coordinates. Centralize
boundary and overlap checks. A move first validates the proposed coordinates,
then changes the stored rectangle. Start with an `O(n)` overlap scan; introduce
a spatial index only if the constraints require it.

**Test:** touching edges, complete containment, identical rectangles, moving a
rectangle over itself, and coordinates on the board boundary.

??? note "Reference solution"

    Put all geometry rules in one validation path so `add` and `move` cannot
    disagree. Use strict `<` in the overlap test: rectangles that only touch
    at an edge do not overlap.

    ```python
    from dataclasses import dataclass, replace


    @dataclass(frozen=True)
    class Rectangle:
        rectangle_id: str
        x: int
        y: int
        width: int
        height: int


    class Board:
        def __init__(self, width, height):
            if width <= 0 or height <= 0:
                raise ValueError("board dimensions must be positive")
            self.width = width
            self.height = height
            self.rectangles = {}

        @staticmethod
        def overlaps(first, second):
            return (
                first.x < second.x + second.width
                and second.x < first.x + first.width
                and first.y < second.y + second.height
                and second.y < first.y + first.height
            )

        def _validate(self, candidate, ignore_id=None):
            if candidate.width <= 0 or candidate.height <= 0:
                raise ValueError("rectangle dimensions must be positive")

            inside = (
                candidate.x >= 0
                and candidate.y >= 0
                and candidate.x + candidate.width <= self.width
                and candidate.y + candidate.height <= self.height
            )
            if not inside:
                raise ValueError("rectangle is outside the board")

            for rectangle_id, rectangle in self.rectangles.items():
                if rectangle_id == ignore_id:
                    continue
                if self.overlaps(candidate, rectangle):
                    raise ValueError(
                        f"rectangle overlaps {rectangle_id}"
                    )

        def add(self, rectangle):
            if rectangle.rectangle_id in self.rectangles:
                raise ValueError("rectangle ID already exists")
            self._validate(rectangle)
            self.rectangles[rectangle.rectangle_id] = rectangle

        def move(self, rectangle_id, x, y):
            current = self.rectangles[rectangle_id]
            candidate = replace(current, x=x, y=y)
            self._validate(candidate, ignore_id=rectangle_id)
            self.rectangles[rectangle_id] = candidate

        def remove(self, rectangle_id):
            return self.rectangles.pop(rectangle_id, None) is not None

        def list(self):
            return [
                self.rectangles[rectangle_id]
                for rectangle_id in sorted(self.rectangles)
            ]


    if __name__ == "__main__":
        board = Board(width=10, height=10)
        board.add(Rectangle("A", x=0, y=0, width=2, height=2))
        board.add(Rectangle("B", x=2, y=0, width=3, height=2))
        board.move("B", x=3, y=1)

        rectangles = board.list()
        assert rectangles[1].x == 3
        assert rectangles[1].y == 1
        for rectangle in rectangles:
            print(rectangle)
    ```

    Sample output:

    ```text
    Rectangle(rectangle_id='A', x=0, y=0, width=2, height=2)
    Rectangle(rectangle_id='B', x=3, y=1, width=3, height=2)
    ```

    `add` and `move` are `O(n)` because they compare with every rectangle.
    `remove` is `O(1)`. If the board contains many rectangles, keep this API
    and replace the scan with a spatial index such as an R-tree.

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

??? note "Reference solution"

    Binary search still controls the algorithm. Timeouts and retries wrap one
    comparison; they do not change which guess comes next. Use a monotonic
    clock so changes to the computer's wall clock do not affect the deadline.

    ```python
    import asyncio


    class TransientComparisonError(Exception):
        pass


    async def find_hidden_number(
        compare,
        low,
        high,
        *,
        per_call_timeout=1.0,
        total_timeout=10.0,
        max_retries=2,
    ):
        if low > high:
            raise ValueError("low must not be greater than high")

        loop = asyncio.get_running_loop()
        deadline = loop.time() + total_timeout

        while low <= high:
            guess = low + (high - low) // 2
            result = None

            for attempt in range(max_retries + 1):
                remaining = deadline - loop.time()
                if remaining <= 0:
                    raise TimeoutError("total deadline exceeded")

                try:
                    result = await asyncio.wait_for(
                        compare(guess),
                        timeout=min(per_call_timeout, remaining),
                    )
                    break
                except (TimeoutError, TransientComparisonError) as error:
                    if attempt == max_retries:
                        raise RuntimeError(
                            f"comparison failed for guess {guess}"
                        ) from error

            if result == "correct":
                return guess
            if result == "too_low":
                low = guess + 1
            elif result == "too_high":
                high = guess - 1
            else:
                raise ValueError(
                    f"comparison returned an invalid result: {result!r}"
                )

        raise LookupError("the hidden number is not in the given range")


    async def comparison_demo():
        hidden = 73

        async def compare(guess):
            if guess == hidden:
                return "correct"
            return "too_low" if guess < hidden else "too_high"

        result = await find_hidden_number(compare, low=1, high=100)
        assert result == hidden
        print("Hidden number:", result)


    if __name__ == "__main__":
        asyncio.run(comparison_demo())
    ```

    Sample output:

    ```text
    Hidden number: 73
    ```

    With reliable calls, the number of comparisons is
    `O(log(high - low + 1))`. Retries increase requests but not search depth.
    `asyncio.wait_for` cancels a timed-out coroutine. The remote service may
    still finish its work, so comparison requests should also carry a request
    ID when duplicate work matters.

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

??? note "Reference solution"

    Make ordering and delete behavior explicit. This solution assumes each
    message has a unique, increasing version and that deletion is final.
    Arrival time is not used to rebuild state.

    ```python
    from collections import Counter


    def aggregate_chat_events(events):
        unique = {}
        for event in events:
            unique.setdefault(event["event_id"], event)

        ordered = sorted(
            unique.values(),
            key=lambda event: (
                event["message_id"],
                event["version"],
                event["event_id"],
            ),
        )
        messages = {}

        for event in ordered:
            message_id = event["message_id"]
            event_type = event["type"]

            if event_type == "message_created":
                messages.setdefault(
                    message_id,
                    {
                        "message_id": message_id,
                        "author": event["author"],
                        "content": event["content"],
                        "deleted": False,
                        "reactions": {},
                    },
                )
                continue

            message = messages.get(message_id)
            if message is None or message["deleted"]:
                continue

            if event_type == "message_edited":
                message["content"] = event["content"]
            elif event_type == "message_deleted":
                message["deleted"] = True
            elif event_type == "reaction_added":
                message["reactions"].setdefault(
                    event["reaction_id"],
                    event["emoji"],
                )
            elif event_type == "reaction_removed":
                message["reactions"].pop(event["reaction_id"], None)
            else:
                raise ValueError(f"unknown event type: {event_type}")

        view = []
        for message_id in sorted(messages):
            message = messages[message_id]
            if message["deleted"]:
                continue
            view.append(
                {
                    "message_id": message_id,
                    "author": message["author"],
                    "content": message["content"],
                    "reaction_counts": dict(
                        Counter(message["reactions"].values())
                    ),
                }
            )
        return view


    if __name__ == "__main__":
        sample_events = [
            {
                "event_id": "event-3",
                "message_id": "message-1",
                "version": 3,
                "type": "reaction_added",
                "reaction_id": "reaction-1",
                "emoji": "👍",
            },
            {
                "event_id": "event-1",
                "message_id": "message-1",
                "version": 1,
                "type": "message_created",
                "author": "Mina",
                "content": "Draft",
            },
            {
                "event_id": "event-2",
                "message_id": "message-1",
                "version": 2,
                "type": "message_edited",
                "content": "Ready",
            },
        ]

        current_view = aggregate_chat_events(sample_events)
        assert current_view[0]["content"] == "Ready"
        assert current_view[0]["reaction_counts"] == {"👍": 1}
        message = current_view[0]
        print("Message:", message["message_id"])
        print("Content:", message["content"])
        print("Reactions:", message["reaction_counts"])
    ```

    Sample output:

    ```text
    Message: message-1
    Content: Ready
    Reactions: {'👍': 1}
    ```

    Sorting makes this `O(e log e)` for `e` events. A production event stream
    would usually partition by message or conversation, reject conflicting
    versions, and store snapshots so it does not replay all history for every
    read. If restoring a deleted message is allowed, model it as an explicit
    event rather than letting an edit silently restore it.

### C8. Data-labeling task scheduler

**Problem:** Assign labeling tasks to workers. Tasks have a priority and
required skill. A task must not be assigned to two workers at once. Abandoned
tasks should become available again.

**Walkthrough:** Keep a priority queue per skill. Claim work with a lease and a
conditional state change. Renew long leases. On expiry, return the task to its
queue. Track attempts and move repeatedly failing tasks to review.

**What to discuss:** fairness, worker quality, agreement between labelers,
gold-standard check tasks, and separating scheduling from label quality.

??? note "Reference solution"

    This uses one priority heap per skill and the same lease idea as C1.
    Higher numeric priority runs first; a sequence number keeps equal-priority
    work in arrival order. A lease token prevents a late worker from
    completing a task that has already been reassigned.

    ```python
    from collections import defaultdict
    from dataclasses import dataclass
    from heapq import heappop, heappush
    from itertools import count
    from uuid import uuid4


    @dataclass
    class LabelingTask:
        task_id: str
        payload: dict
        skill: str
        priority: int
        attempts: int = 0
        state: str = "pending"
        worker_id: str | None = None
        lease_token: str | None = None
        lease_deadline: float | None = None
        label: object | None = None


    class TaskScheduler:
        def __init__(self, max_attempts=3):
            self.tasks = {}
            self.queues = defaultdict(list)
            self.expiring = []
            self.sequence = count()
            self.max_attempts = max_attempts

        def _enqueue(self, task):
            heappush(
                self.queues[task.skill],
                (-task.priority, next(self.sequence), task.task_id),
            )

        def add(self, task):
            if task.task_id in self.tasks:
                return
            self.tasks[task.task_id] = task
            self._enqueue(task)

        def _expire(self, now):
            while self.expiring and self.expiring[0][0] <= now:
                deadline, task_id, token = heappop(self.expiring)
                task = self.tasks[task_id]
                active_lease = (
                    task.state == "leased"
                    and task.lease_token == token
                    and task.lease_deadline == deadline
                )
                if not active_lease:
                    continue

                task.worker_id = None
                task.lease_token = None
                task.lease_deadline = None
                if task.attempts >= self.max_attempts:
                    task.state = "review"
                else:
                    task.state = "pending"
                    self._enqueue(task)

        def claim(self, skill, worker_id, now, lease_seconds):
            self._expire(now)
            queue = self.queues[skill]

            while queue:
                _, _, task_id = heappop(queue)
                task = self.tasks[task_id]
                if task.state != "pending":
                    continue

                token = str(uuid4())
                task.state = "leased"
                task.attempts += 1
                task.worker_id = worker_id
                task.lease_token = token
                task.lease_deadline = now + lease_seconds
                heappush(
                    self.expiring,
                    (task.lease_deadline, task.task_id, token),
                )
                return task.task_id, task.payload, token

            return None

        def renew(self, task_id, token, now, lease_seconds):
            task = self.tasks[task_id]
            if task.state != "leased" or task.lease_token != token:
                return False
            task.lease_deadline = now + lease_seconds
            heappush(
                self.expiring,
                (task.lease_deadline, task.task_id, token),
            )
            return True

        def complete(self, task_id, token, label):
            task = self.tasks[task_id]
            if task.state != "leased" or task.lease_token != token:
                return False
            task.state = "done"
            task.label = label
            task.lease_token = None
            task.lease_deadline = None
            return True


    if __name__ == "__main__":
        scheduler = TaskScheduler()
        scheduler.add(
            LabelingTask(
                "normal-task",
                {"text": "Good morning"},
                skill="English",
                priority=10,
            )
        )
        scheduler.add(
            LabelingTask(
                "urgent-task",
                {"text": "Account locked"},
                skill="English",
                priority=100,
            )
        )

        claim = scheduler.claim(
            skill="English",
            worker_id="worker-7",
            now=0,
            lease_seconds=30,
        )
        completed = scheduler.complete(
            claim[0],
            claim[2],
            label="support_request",
        )

        assert claim[0] == "urgent-task"
        assert completed is True
        print("Claimed first:", claim[0])
        print("Final state:", scheduler.tasks[claim[0]].state)
    ```

    Sample output:

    ```text
    Claimed first: urgent-task
    Final state: done
    ```

    Heap operations are `O(log n)`. These methods need one lock in memory or
    one conditional transaction in a database. Scheduling decides who gets
    work; a separate quality pipeline should measure agreement, route
    uncertain labels to review, and monitor quality by worker and task type.

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

??? note "Reference solution"

    Treat model output as untrusted input. The application owns tool names,
    argument validation, approval, timeouts, and budgets. This example assumes
    every registered tool and the approval function are asynchronous.

    ```python
    import asyncio
    from dataclasses import dataclass


    @dataclass(frozen=True)
    class Tool:
        run: object
        validate: object
        has_side_effect: bool = False


    async def run_tool_loop(
        model,
        tools,
        messages,
        approve,
        *,
        max_turns=8,
        max_tool_calls=20,
        total_timeout=30.0,
        per_tool_timeout=10.0,
    ):
        loop = asyncio.get_running_loop()
        deadline = loop.time() + total_timeout
        tool_call_count = 0

        def time_left():
            return deadline - loop.time()

        async def execute(plan):
            call, tool, arguments = plan
            try:
                if tool.has_side_effect:
                    approval_timeout = time_left()
                    if approval_timeout <= 0:
                        raise TimeoutError("total deadline exceeded")
                    allowed = await asyncio.wait_for(
                        approve(call["name"], arguments),
                        timeout=approval_timeout,
                    )
                    if not allowed:
                        return {
                            "call_id": call["id"],
                            "ok": False,
                            "error": "approval denied",
                        }

                timeout = min(per_tool_timeout, time_left())
                if timeout <= 0:
                    raise TimeoutError("total deadline exceeded")
                value = await asyncio.wait_for(
                    tool.run(**arguments),
                    timeout=timeout,
                )
                return {
                    "call_id": call["id"],
                    "ok": True,
                    "value": value,
                }
            except Exception as error:
                return {
                    "call_id": call["id"],
                    "ok": False,
                    "error": f"{type(error).__name__}: {error}",
                }

        for _ in range(max_turns):
            if time_left() <= 0:
                raise TimeoutError("tool loop exceeded its total deadline")

            response = await asyncio.wait_for(
                model(messages),
                timeout=time_left(),
            )
            if response["type"] == "final":
                return response["text"]
            if response["type"] != "tool_calls":
                raise ValueError("model returned an unknown response type")

            calls = response["calls"]
            tool_call_count += len(calls)
            if tool_call_count > max_tool_calls:
                raise RuntimeError("tool-call budget exceeded")

            results = {}
            read_plans = []
            write_plans = []

            for call in calls:
                tool = tools.get(call["name"])
                if tool is None:
                    results[call["id"]] = {
                        "call_id": call["id"],
                        "ok": False,
                        "error": "unknown tool",
                    }
                    continue

                try:
                    arguments = tool.validate(call.get("arguments", {}))
                except Exception as error:
                    results[call["id"]] = {
                        "call_id": call["id"],
                        "ok": False,
                        "error": f"invalid arguments: {error}",
                    }
                    continue

                plan = (call, tool, arguments)
                if tool.has_side_effect:
                    write_plans.append(plan)
                else:
                    read_plans.append(plan)

            read_results = await asyncio.gather(
                *(execute(plan) for plan in read_plans)
            )
            for result in read_results:
                results[result["call_id"]] = result

            for plan in write_plans:
                result = await execute(plan)
                results[result["call_id"]] = result

            messages = [
                *messages,
                {"role": "assistant", "content": response},
                {
                    "role": "tool",
                    "content": [
                        results[call["id"]]
                        for call in calls
                    ],
                },
            ]

        raise RuntimeError("model did not finish within the turn budget")


    class DemoModel:
        async def __call__(self, messages):
            if len(messages) == 1:
                return {
                    "type": "tool_calls",
                    "calls": [
                        {
                            "id": "call-1",
                            "name": "read_profile",
                            "arguments": {"user_id": 42},
                        }
                    ],
                }

            tool_result = messages[-1]["content"][0]["value"]
            return {
                "type": "final",
                "text": f"Profile status: {tool_result['status']}",
            }


    async def read_profile(user_id):
        return {"user_id": user_id, "status": "active"}


    def validate_profile_arguments(arguments):
        if set(arguments) != {"user_id"}:
            raise ValueError("user_id is required")
        return {"user_id": int(arguments["user_id"])}


    async def approve(_tool_name, _arguments):
        return True


    async def tool_loop_demo():
        tools = {
            "read_profile": Tool(
                run=read_profile,
                validate=validate_profile_arguments,
            )
        }
        answer = await run_tool_loop(
            model=DemoModel(),
            tools=tools,
            messages=[
                {
                    "role": "user",
                    "content": "Is user 42 active?",
                }
            ],
            approve=approve,
        )

        assert answer == "Profile status: active"
        print(answer)


    if __name__ == "__main__":
        asyncio.run(tool_loop_demo())
    ```

    Sample output:

    ```text
    Profile status: active
    ```

    Only independent reads should appear in the same parallel batch.
    Side-effecting tools run one at a time and require approval. A timeout does
    not prove that a remote write did not happen, so write calls need an
    idempotency key and backend support for deduplication. In production, also
    limit model tokens or cost, cap result size, redact secrets, trace each
    call, and use typed message objects instead of raw dictionaries.

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

??? note "Reference solution"

    Score the baseline and candidate on exactly the same cases. Keep
    per-category scores and release-blocking cases; an overall average alone
    can hide a severe safety or correctness failure.

    ```python
    from collections import defaultdict
    from dataclasses import dataclass
    from statistics import mean


    @dataclass(frozen=True)
    class EvaluationCase:
        case_id: str
        input: object
        expected: object
        category: str
        grader: object
        blocking: bool = False


    def run_suite(system, cases, blocking_score=1.0):
        results = []

        for case in cases:
            try:
                output = system(case.input)
                score, reason = case.grader(output, case.expected)
                if not 0.0 <= score <= 1.0:
                    raise ValueError("grader score must be between 0 and 1")
            except Exception as error:
                output = None
                score = 0.0
                reason = f"{type(error).__name__}: {error}"

            results.append(
                {
                    "case_id": case.case_id,
                    "category": case.category,
                    "blocking": case.blocking,
                    "output": output,
                    "score": score,
                    "reason": reason,
                }
            )

        if not results:
            raise ValueError("evaluation suite must not be empty")

        by_category = defaultdict(list)
        for result in results:
            by_category[result["category"]].append(result["score"])

        return {
            "overall": mean(result["score"] for result in results),
            "categories": {
                category: mean(scores)
                for category, scores in by_category.items()
            },
            "blocking_failures": [
                result["case_id"]
                for result in results
                if result["blocking"]
                and result["score"] < blocking_score
            ],
            "results": results,
        }


    def evaluate_release(
        baseline,
        candidate,
        cases,
        *,
        max_overall_drop=0.01,
        max_category_drop=0.05,
    ):
        baseline_report = run_suite(baseline, cases)
        candidate_report = run_suite(candidate, cases)
        reasons = []

        overall_drop = (
            baseline_report["overall"] - candidate_report["overall"]
        )
        if overall_drop > max_overall_drop:
            reasons.append(f"overall score dropped by {overall_drop:.3f}")

        for category, baseline_score in baseline_report[
            "categories"
        ].items():
            candidate_score = candidate_report["categories"][category]
            drop = baseline_score - candidate_score
            if drop > max_category_drop:
                reasons.append(
                    f"{category} score dropped by {drop:.3f}"
                )

        if candidate_report["blocking_failures"]:
            reasons.append(
                "blocking cases failed: "
                + ", ".join(candidate_report["blocking_failures"])
            )

        return {
            "ship": not reasons,
            "reasons": reasons,
            "baseline": baseline_report,
            "candidate": candidate_report,
        }


    def exact_match(output, expected):
        if output == expected:
            return 1.0, "matched"
        return 0.0, "did not match"


    if __name__ == "__main__":
        cases = [
            EvaluationCase(
                case_id="uppercase",
                input="hello",
                expected="HELLO",
                category="correctness",
                grader=exact_match,
                blocking=True,
            )
        ]
        report = evaluate_release(
            baseline=lambda value: value.upper(),
            candidate=lambda value: value.lower(),
            cases=cases,
        )

        assert report["ship"] is False
        print("Ship candidate:", report["ship"])
        print(
            "Candidate score:",
            report["candidate"]["overall"],
        )
        print(
            "Blocking failures:",
            report["candidate"]["blocking_failures"],
        )
    ```

    Sample output:

    ```text
    Ship candidate: False
    Candidate score: 0.0
    Blocking failures: ['uppercase']
    ```

    Real AI evaluations often combine deterministic checks, human review, and
    model graders. Calibrate model graders against people, retain the raw
    outputs, and inspect close results rather than treating a small score
    difference as certain.

<!-- markdownlint-enable MD046 -->

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
