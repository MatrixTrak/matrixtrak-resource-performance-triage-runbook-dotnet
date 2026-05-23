# Performance Triage Runbook

## Purpose

Find the top 3 performance bottlenecks in a legacy .NET system in 2-4 hours.

---

## Step 1: Baseline the Four Pillars (30 minutes)

### What to Measure

| Pillar | Tool | Key Metrics | Abnormal Threshold |
|--------|------|-------------|-------------------|
| CPU | dotnet-counters | `cpu-usage` | > 70-80% sustained |
| Memory/GC | dotnet-counters | `gc-heap-size`, `time-in-gc` | GC > 15%, heap growing |
| Thread Pool | dotnet-counters | `threadpool-queue-length` | > 0 sustained, > 10 critical |
| External I/O | APM/Logs | Dependency call duration | > 50% of request time |

### Commands

```bash
# Monitor all key metrics for 15-30 minutes
dotnet-counters monitor --process-id <pid> --counters System.Runtime

# Key counters to watch:
# - cpu-usage
# - gc-heap-size
# - gen-0-gc-count, gen-1-gc-count, gen-2-gc-count
# - time-in-gc
# - threadpool-queue-length
# - threadpool-thread-count
# - exception-count
```

### Record Your Findings

| Pillar | Observed Value | Normal/Abnormal | Notes |
|--------|----------------|-----------------|-------|
| CPU | ___% | [ ] Normal [ ] Abnormal | |
| GC Time | ___% | [ ] Normal [ ] Abnormal | |
| Heap Size | ___MB | [ ] Normal [ ] Abnormal | |
| Thread Pool Queue | ___ | [ ] Normal [ ] Abnormal | |

---

## Step 2: Find the Slowest Endpoints (30 minutes)

### Query Your APM

**Query 1: Top 10 by p95 latency**
- Application Insights: `requests | summarize percentile(duration, 95) by name | top 10 by percentile_duration_95 desc`
- Datadog: Filter by service, sort by p95
- New Relic: Transactions, sort by slowest

**Query 2: Top 10 by total time consumed**
- Calculate: `p95_latency × request_count`
- This shows where the system spends the most total time

### Record Your Findings

| Rank | Endpoint | p95 Latency | Request Count | Total Time |
|------|----------|-------------|---------------|------------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

### Focus List

The intersection of high p95 AND high total time is where to focus first.

Endpoints to investigate:
1. ___________
2. ___________
3. ___________

---

## Step 3: Trace One Slow Request End-to-End (60 minutes)

### Add Timing Logs

For the slowest high-traffic endpoint, add timing around each stage:

```csharp
// Example timing pattern
var requestStart = Stopwatch.StartNew();

// Stage 1: Validation
var validationStart = Stopwatch.StartNew();
// ... validation logic ...
var validationMs = validationStart.ElapsedMilliseconds;

// Stage 2: Database
var dbStart = Stopwatch.StartNew();
// ... database calls ...
var dbMs = dbStart.ElapsedMilliseconds;

// Stage 3: External HTTP
var httpStart = Stopwatch.StartNew();
// ... HTTP calls ...
var httpMs = httpStart.ElapsedMilliseconds;

// Stage 4: Business Logic
var logicStart = Stopwatch.StartNew();
// ... processing ...
var logicMs = logicStart.ElapsedMilliseconds;

var totalMs = requestStart.ElapsedMilliseconds;

_logger.LogInformation(
    "Request timing: {Endpoint} total={TotalMs}ms validation={ValidationMs}ms db={DbMs}ms http={HttpMs}ms logic={LogicMs}ms",
    endpoint, totalMs, validationMs, dbMs, httpMs, logicMs);
```

### Time Breakdown

| Stage | Elapsed (ms) | % of Total | Notes |
|-------|--------------|------------|-------|
| Validation | | | |
| Database | | | |
| External HTTP | | | |
| Business Logic | | | |
| Other/Unaccounted | | | |
| **Total** | | 100% | |

### Where Is Time Going?

- [ ] Database calls (> 50% of time)
- [ ] External HTTP calls (> 30% of time)
- [ ] Business logic / computation (> 30% of time)
- [ ] Unaccounted (need finer logging)

---

## Step 4: Rank and Classify Bottlenecks (30 minutes)

### Bottleneck List

| # | Bottleneck | Impact (p95 ms saved) | Type | Effort | Priority |
|---|------------|----------------------|------|--------|----------|
| 1 | | | [ ] Quick [ ] Structural | | |
| 2 | | | [ ] Quick [ ] Structural | | |
| 3 | | | [ ] Quick [ ] Structural | | |
| 4 | | | [ ] Quick [ ] Structural | | |
| 5 | | | [ ] Quick [ ] Structural | | |

### Classification Criteria

**Quick Win (do first):**
- [ ] Configuration change (timeouts, pool sizes)
- [ ] Replace sync with async
- [ ] Batch N+1 queries
- [ ] Add simple cache
- [ ] Remove excessive logging
- [ ] Add missing indexes

**Structural (do after quick wins):**
- [ ] Redesign data access layer
- [ ] Add distributed caching
- [ ] Split monolith
- [ ] Upgrade .NET version
- [ ] Rewrite component

---

## Step 5: Fix and Measure

### Fix Priority Order

1. __________ (impact: ___ms, effort: ___ days)
2. __________ (impact: ___ms, effort: ___ days)
3. __________ (impact: ___ms, effort: ___ days)

### Measurement Plan

| Fix | Before p95 | After p95 | Improvement | Date |
|-----|------------|-----------|-------------|------|
| Fix 1 | | | | |
| Fix 2 | | | | |
| Fix 3 | | | | |

### Success Criteria

- [ ] p95 latency reduced by ___ms (target)
- [ ] No regressions in other endpoints
- [ ] Alerting configured for p95 threshold
- [ ] Findings documented for future reference

---

## Quick Reference: Common Bottlenecks

| Pattern | Symptom | Likely Cause | Quick Fix |
|---------|---------|--------------|-----------|
| CPU low, latency high | Thread pool queue > 0 | Sync over async | Remove .Result/.Wait |
| CPU high, latency high | GC > 15% | Excessive allocations | Reduce string/object creation |
| One endpoint slow | Database time > 50% | N+1 queries | Batch queries |
| All endpoints slow | No single cause | Multiple issues | Triage and fix top 3 |
| Slow under load only | Connection exhaustion | Pool too small | Increase pool capacity |
