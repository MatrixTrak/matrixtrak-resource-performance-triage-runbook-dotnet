# Performance Triage Runbook for Legacy .NET

A step-by-step method to identify the top 3 performance bottlenecks in a legacy .NET system in hours, not weeks.

## What's Inside

| File | Purpose |
|------|---------|
| `performance-triage-runbook.md` | Step-by-step method with timing guidance |
| `bottleneck-classification-checklist.md` | Quick-win vs structural decision framework |
| `dotnet-counters-cheatsheet.md` | Commands and thresholds for each metric |
| `README.md` | This file |

## Quick Start

1. **Baseline the four pillars** using `dotnet-counters` (30 minutes)
2. **Find the slowest endpoints** by p95 in your APM (30 minutes)
3. **Trace one slow request** end-to-end with timing logs (60 minutes)
4. **Rank and classify** bottlenecks by impact and effort (30 minutes)
5. **Fix top 3 quick wins** and measure after each

## How to Use

### On Call

- Check the four pillars first: CPU, memory, thread pool, I/O
- If CPU is low but latency is high: likely thread pool starvation or I/O waits
- If CPU is high: profile with PerfView to find the hot path

### Tech Lead

- Run the triage method to identify top 3 bottlenecks
- Classify each as quick-win or structural
- Prioritize quick wins in the current sprint
- Schedule structural fixes for future sprints

### CTO

- The triage delivers measurable results: p95 reduction in ms
- Each bottleneck has an impact estimate before work starts
- Quick wins typically deliver 50-70% of improvement
- Risk is bounded: each fix is independent and measurable

## Prerequisites

- .NET 6+ (for dotnet-counters; alternatives exist for older versions)
- Production access or staging environment under realistic load
- APM tool (Application Insights, Datadog, New Relic) or structured logging

## Related Resources

- [Performance triage blog post](/blog/performance-triage-legacy-dotnet-find-top-3-bottlenecks-fast)
- [Thread pool starvation: the silent killer](/blog/thread-pool-starvation-silent-killer-aspnet-performance)
- [dotnet-counters documentation](https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-counters)

## Support

If you need hands-on help with performance triage or optimization, see [MatrixTrak services](/services).
