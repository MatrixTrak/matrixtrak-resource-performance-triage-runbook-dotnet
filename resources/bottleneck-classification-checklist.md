# Bottleneck Classification Checklist

## Purpose

Decide whether a bottleneck is a quick-win or structural fix before assigning work.

---

## Quick Win Checklist

A bottleneck is a **quick win** if ANY of these apply:

### Configuration Changes
- [ ] Missing timeout (add explicit timeout)
- [ ] Connection pool too small (increase capacity)
- [ ] Thread pool min threads too low (set minWorkerThreads)
- [ ] GC mode incorrect (use Server GC for server apps)
- [ ] Missing indexes in database (add index)

### Code Changes (< 1 day effort)
- [ ] Sync over async (.Result, .Wait, .GetAwaiter().GetResult())
- [ ] N+1 query pattern (batch into single query)
- [ ] Excessive logging in hot path (reduce log level or remove)
- [ ] Missing cache for repeated identical calls (add in-memory cache)
- [ ] String concatenation in loop (use StringBuilder)
- [ ] LINQ .ToList() when only .Any() needed

### Add Infrastructure (< 2 days effort)
- [ ] Add timing logs to identify time sinks
- [ ] Add bulkhead around slow dependency
- [ ] Add retry with backoff to flaky calls
- [ ] Add circuit breaker to failing dependency

---

## Structural Fix Checklist

A bottleneck is **structural** if ANY of these apply:

### Requires Redesign
- [ ] Data access layer needs rewrite
- [ ] Missing caching layer (needs Redis/distributed cache)
- [ ] Tight coupling between components
- [ ] Missing async throughout call chain (can't add async at one layer)
- [ ] Memory leak from architectural issue

### Requires Significant Infrastructure
- [ ] Need to split monolith into services
- [ ] Need to add message queue for decoupling
- [ ] Need to migrate to newer .NET version
- [ ] Need to add distributed tracing infrastructure

### Requires Multi-Sprint Work
- [ ] Estimated effort > 1 week
- [ ] Multiple teams need to coordinate
- [ ] Breaking API changes required
- [ ] Database schema migration required

---

## Decision Tree

```
Is the fix a config change?
├── YES → Quick Win (do immediately)
└── NO → Is the code change < 1 day effort?
    ├── YES → Quick Win (current sprint)
    └── NO → Is the change isolated to one component?
        ├── YES → Structural (schedule next sprint)
        └── NO → Structural (plan multi-sprint work)
```

---

## Priority Matrix

| Impact | Quick Win | Structural |
|--------|-----------|------------|
| High (> 500ms p95 saved) | Do first | Plan for next sprint |
| Medium (100-500ms) | Do second | Defer or skip |
| Low (< 100ms) | If time permits | Skip |

---

## Warning Signs: Wrong Classification

### Treating Structural as Quick Win
- "Let's just add async here" — but the entire call chain is sync
- "Let's just cache this" — but no cache infrastructure exists
- "Let's just split this out" — but data coupling is deep

### Treating Quick Win as Structural
- "We need to refactor the whole layer" — for what is really one method fix
- "We need a project to address this" — for a configuration change
- "This is too risky to touch" — for isolated, well-tested code

---

## Documentation Template

For each bottleneck, document:

```markdown
## Bottleneck: [Name]

**Location:** [File/Class/Method]
**Impact:** [Estimated p95 ms saved]
**Classification:** Quick Win / Structural
**Effort:** [Hours/Days/Weeks]

**Evidence:**
- [What measurement shows this is a bottleneck]

**Fix:**
- [Specific change required]

**Risks:**
- [What could go wrong]

**Validation:**
- [How to verify the fix worked]
```
