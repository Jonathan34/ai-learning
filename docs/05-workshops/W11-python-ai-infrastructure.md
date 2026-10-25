# Workshop W11 — Python for AI Infrastructure

**Goal:** Build a small command-line report that turns raw GPU-cluster outputs into useful health findings.

**Time:** 3-4 hours

**Prerequisites:** Python, basic testing, and familiarity with distributed training, cluster networking, and reliability

---

## What you're building

Your tool will read three files:

- `nccl.txt`: output from `all_reduce_perf`
- `steps.json`: per-rank training step times
- `gpus.csv`: selected `nvidia-smi` fields

It will print a JSON report with:

- Peak collective bus bandwidth
- Large-message tests below a supplied baseline
- Suspected straggler ranks
- GPUs with high temperature or active throttling
- A final healthy or unhealthy result

Use captured or synthetic data. No GPU is required.

## Part 1: Parse collective results

Start with a real sample format from `nccl-tests`. Do not split on fixed character positions; columns are separated by variable whitespace.

Represent each parsed row as a dataclass:

```python
@dataclass(frozen=True)
class NcclResult:
    size_bytes: int
    algorithm_bandwidth: float
    bus_bandwidth: float
    errors: int
```

Write:

```python
def parse_nccl(text: str) -> list[NcclResult]:
    ...
```

Tests should cover:

- Header and comment lines
- Empty input
- Malformed rows
- In-place and out-of-place columns
- A large message below the baseline

Decide whether malformed data should fail the command or appear as a warning. Document the choice.

## Part 2: Find stragglers

Average and standard deviation are sensitive to large outliers. Use the median and median absolute deviation:

```text
MAD = median(abs(value - median(values)))
```

Flag ranks whose step time is more than a configurable number of MADs above the median.

Handle the case where all ranks have the same time and MAD is zero.

Your output should explain the threshold:

```json
{
  "median_step_ms": 412.8,
  "mad_step_ms": 3.1,
  "suspected_stragglers": [17]
}
```

## Part 3: Parse GPU health

Capture a stable CSV shape:

```bash
nvidia-smi \
  --query-gpu=index,uuid,temperature.gpu,power.draw,utilization.gpu,memory.used \
  --format=csv,noheader,nounits
```

Write a parser and rules for:

- Temperature above a supplied threshold
- Missing values
- Duplicate GPU IDs
- Zero utilization as information, not automatic failure

Throttle reasons and XIDs vary by driver and source. Add them only if your sample data has a well-defined format.

## Part 4: Add concurrent collection

Pretend each node exposes these files through an API.

Use `asyncio` to fetch up to N nodes concurrently, with:

- A semaphore
- Per-request timeout
- Bounded retries
- Exponential backoff with jitter
- Partial results when one node fails

Do not retry every exception. Invalid authentication and malformed responses usually need a different action than a timeout.

## Part 5: Produce one report

The command should support:

```text
cluster-health \
  --nccl nccl.txt \
  --steps steps.json \
  --gpus gpus.csv \
  --min-busbw 320 \
  --max-temp 85
```

Return a non-zero exit code when a required health check fails. That makes the tool usable in CI and cluster admission workflows.

## Part 6: Review it as production code

Check:

- Type hints
- Clear error messages
- Unit tests
- No shell injection
- Bounded memory use
- Stable JSON output
- Logging separate from machine-readable output
- No secret values in logs

## What you should have

- A Python package or single-purpose CLI
- At least ten tests
- Three sample input files
- One healthy and one unhealthy report
- A short README explaining thresholds and limits

## Go deeper

- [Python `asyncio` documentation](https://docs.python.org/3/library/asyncio.html)
- [NCCL tests](https://github.com/NVIDIA/nccl-tests)
- [Chapter 31 — Reliability and Performance](../06-ai-infrastructure/31-reliability-and-performance.md)
