# dotnet-counters Cheatsheet

## Quick Reference

| Metric | Normal | Warning | Critical | What It Means |
|--------|--------|---------|----------|---------------|
| cpu-usage | < 60% | 60-80% | > 80% | Process compute load |
| gen-0-gc-count | varies | > 10/sec | > 50/sec | Frequent small allocations |
| gen-2-gc-count | rare | > 1/min | > 5/min | Large/long-lived allocations |
| time-in-gc | < 5% | 5-15% | > 15% | GC blocking work |
| gc-heap-size | stable | growing | growing fast | Memory pressure |
| threadpool-queue-length | 0 | 1-10 | > 10 | Thread pool starvation |
| threadpool-thread-count | stable | growing | at max | Concurrency demand |
| exception-count | low | rising | spiking | Error rate |
| lock-contention-count | low | rising | spiking | Thread contention |

---

## Commands

### Basic Monitoring

```bash
# List running .NET processes
dotnet-counters ps

# Monitor default counters
dotnet-counters monitor --process-id <pid>

# Monitor specific counter set
dotnet-counters monitor --process-id <pid> --counters System.Runtime
```

### Advanced Monitoring

```bash
# Monitor with refresh interval (1 second)
dotnet-counters monitor --process-id <pid> --refresh-interval 1

# Monitor multiple counter providers
dotnet-counters monitor --process-id <pid> \
  --counters System.Runtime,Microsoft.AspNetCore.Hosting

# Collect counters to file (for later analysis)
dotnet-counters collect --process-id <pid> --output counters.csv
```

### ASP.NET Core Specific

```bash
# Monitor ASP.NET Core hosting counters
dotnet-counters monitor --process-id <pid> \
  --counters Microsoft.AspNetCore.Hosting

# Key ASP.NET metrics:
# - requests-per-second
# - total-requests
# - current-requests
# - failed-requests
```

---

## Key Metrics Explained

### CPU Usage (cpu-usage)

- **What it measures:** Percentage of CPU time used by the process
- **Normal:** < 60%
- **High means:** Your code is compute-bound
- **Low + high latency means:** Thread pool starvation or I/O waits

```bash
# Check CPU
dotnet-counters monitor --process-id <pid> --counters System.Runtime[cpu-usage]
```

### Garbage Collection (GC)

| Counter | What It Means |
|---------|---------------|
| gen-0-gc-count | Collections of short-lived objects (fast, frequent) |
| gen-1-gc-count | Collections of medium-lived objects |
| gen-2-gc-count | Collections of long-lived objects (slow, should be rare) |
| gc-heap-size | Total managed heap size in bytes |
| time-in-gc | Percentage of time spent in GC (blocking work) |

```bash
# Check GC health
dotnet-counters monitor --process-id <pid> --counters \
  System.Runtime[gc-heap-size,gen-0-gc-count,gen-1-gc-count,gen-2-gc-count,time-in-gc]
```

**Warning signs:**
- gen-2-gc-count > 1/minute: Large allocations or memory leak
- time-in-gc > 15%: GC is starving your application
- gc-heap-size growing: Memory leak or unbounded cache

### Thread Pool

| Counter | What It Means |
|---------|---------------|
| threadpool-thread-count | Current number of thread pool threads |
| threadpool-queue-length | Work items waiting for a thread |

```bash
# Check thread pool health
dotnet-counters monitor --process-id <pid> --counters \
  System.Runtime[threadpool-thread-count,threadpool-queue-length]
```

**Warning signs:**
- threadpool-queue-length > 0 under normal load: Starvation starting
- threadpool-queue-length > 10: Active starvation, requests queuing
- threadpool-thread-count at max (500+): Pool exhausted

### Exceptions

```bash
# Check exception rate
dotnet-counters monitor --process-id <pid> --counters \
  System.Runtime[exception-count]
```

**Warning signs:**
- exception-count rising: Error rate increasing
- Exceptions in hot path: Performance impact (exceptions are slow)

---

## Interpretation Guide

### Pattern: CPU Low, Latency High

```
cpu-usage: 20%
threadpool-queue-length: 15
```

**Diagnosis:** Thread pool starvation. Threads are blocked waiting, not computing.
**Fix:** Remove sync-over-async, add timeouts, reduce blocking calls.

### Pattern: CPU High, GC High

```
cpu-usage: 85%
time-in-gc: 25%
gen-0-gc-count: 50/sec
```

**Diagnosis:** Excessive allocations causing GC pressure.
**Fix:** Profile allocations with PerfView, reduce object creation in hot paths.

### Pattern: Memory Growing

```
gc-heap-size: 500MB → 800MB → 1.2GB (over 1 hour)
gen-2-gc-count: increasing
```

**Diagnosis:** Memory leak or unbounded cache.
**Fix:** Take heap dump, find retention path, fix leak or add cache eviction.

### Pattern: Thread Count Growing

```
threadpool-thread-count: 50 → 100 → 200 → 300
threadpool-queue-length: stable at 0
```

**Diagnosis:** Async work completing slowly, pool expanding to compensate.
**Fix:** May be OK if it stabilizes. If growing unbounded, check for I/O waits.

---

## Output to CSV for Analysis

```bash
# Collect counters for 5 minutes
dotnet-counters collect --process-id <pid> --output counters.csv --duration 00:05:00

# Collect with specific counters
dotnet-counters collect --process-id <pid> --output counters.csv \
  --counters System.Runtime,Microsoft.AspNetCore.Hosting
```

The CSV can be imported into Excel or a time-series tool for graphing.

---

## Alternatives for Older .NET Versions

### .NET Framework (no dotnet-counters)

Use Performance Monitor (perfmon) with these counters:
- .NET CLR Memory: % Time in GC, Gen 0/1/2 Collections
- .NET CLR LocksAndThreads: Contention Rate/sec
- ASP.NET Applications: Requests/Sec, Request Execution Time

### Windows

```powershell
# Get performance counters
Get-Counter '\Process(your-app)\% Processor Time'
Get-Counter '\.NET CLR Memory(your-app)\% Time in GC'
```

---

## Quick Diagnosis Script

Run this for a quick health check:

```bash
# Save as triage.sh and run with: ./triage.sh <pid>
dotnet-counters monitor --process-id $1 --counters \
  System.Runtime[cpu-usage,gc-heap-size,time-in-gc,threadpool-queue-length,exception-count] \
  --refresh-interval 2
```

Watch for 2-3 minutes under normal load. Any abnormal values indicate where to dig deeper.
