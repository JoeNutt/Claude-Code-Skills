# Hard Debugging Cases

The five-step method still applies. What changes is how you achieve step 1 — stabilization —
because in every case below the defect actively resists reproduction.

## It will not reproduce

Work through the variables that differ between the failing and passing runs:

| Suspect | How it hides |
|---|---|
| **Timing / ordering** | Passes alone, fails in a suite; passes under a debugger |
| **Uninitialized state** | Depends on whatever the previous test or allocation left behind |
| **Test pollution** | Shared fixture, module-level cache, a global mutated by an earlier case |
| **Environment** | Version, locale, timezone, filesystem case-sensitivity, path separator, env var |
| **Clock** | Month ends, DST transitions, leap days, timezone boundaries, expiring tokens |
| **Randomness** | Unseeded generator, hash iteration order, UUID collisions in a small space |
| **Data** | Only a specific record triggers it — null field, unicode, empty collection, huge value |
| **Caching** | Stale build artifact, module cache, HTTP or DNS cache, memoized value |
| **Resources** | Only under memory pressure, disk full, connection pool exhausted |

Run the failing case in isolation and then in its suite. If the result differs, the defect is
in the shared state, not in the code under test.

Pin down everything you can control — seed the RNG, freeze the clock, fix the ordering, use a
clean fixture — until it fails every time. Then remove the pins one at a time; the one that
restores intermittency names the cause.

## Heisenbugs: it disappears when observed

Adding a print or attaching a debugger changes timing, allocation, and optimization. When
the act of looking hides the defect:

- Log to a ring buffer in memory and dump it *after* the failure, rather than writing during.
- Record rather than watch: capture inputs, thread interleavings, and state transitions and
  analyze them afterwards.
- Reduce observer cost — one counter is cheaper than a formatted string.
- Suspect the categories that are timing-sensitive by nature: uninitialized memory, data
  races, and code the optimizer is allowed to reorder.

## Concurrency

The hardest class, because the failure is in an *interleaving* rather than in any line.

- Look for shared mutable state first. Every concurrency defect needs it; remove the sharing
  and the defect cannot exist.
- Enumerate what must be atomic and check it actually is. Read-modify-write across two
  statements is the classic gap.
- Check lock ordering across all call paths — inconsistent order is deadlock waiting for load.
- Use the platform's race detector or thread sanitizer. This is one of the few places where
  a tool genuinely outperforms reasoning.
- Increase contention deliberately to make it fail on demand: more threads, artificial
  delays at suspected windows, single-CPU affinity.
- Be sceptical of a fix that only reduces the failure rate. A narrowed window is not a
  closed one, and it will reopen on faster hardware or under real load.

## Only in production

- Diff the environments systematically: versions, configuration, data volume, concurrency,
  network topology, resource limits, feature flags.
- Production data is usually the difference. Scale, unicode, nulls, legacy rows written by
  an older version of your own code, and values no test fixture would contain.
- Improve observability *before* theorizing. A correlation id through the request path, and
  structured logs at each boundary, turn an unreproducible report into evidence.
- Reproduce against a sanitized copy of real data rather than synthetic fixtures.
- Check what else was deployed. The defect may be in an interaction, not in your change.

## Performance

Never guess at a hot path — the intuition is wrong often enough that acting on it wastes the
effort it was meant to save.

- Measure first, with a profiler, under a workload that resembles production.
- Find the actual bottleneck rather than the suspicious-looking code. Most time is usually in
  a small fraction of the code, and rarely the fraction you expected.
- Check the algorithm before the constant factor: an accidental O(n²), an N+1 query, work
  repeated inside a loop that could be hoisted.
- Establish a baseline measurement, change one thing, measure again. Unmeasured optimization
  is a readability cost with unknown benefit.
- Stop when it is fast enough. "Faster" is not a requirement.

## Memory and resources

- Growth over time means a leak: an unbounded cache, a listener never removed, a connection
  or file handle never closed, an ever-growing collection.
- Reproduce by accelerating — run the suspect cycle thousands of times and watch the trend.
- Use the platform's heap profiler to find what is retained and, more usefully, *what is
  retaining it*.
- Check that every acquisition has a release on **every** path, including the error paths.
  Prefer the language's scoped construct so the release cannot be skipped.

## When genuinely stuck

- Re-read the error message and the stack trace. Properly, from the top, out loud.
- State the problem to someone else — the explanation frequently produces the answer before
  the listener responds.
- Question an assumption you have not tested. Write the list; the cause is usually on it,
  marked "obviously fine".
- Verify you are debugging what you think you are: the right build, the right branch, the
  right process, the right file, actually rebuilt.
- Take a break. Fatigue reliably produces tunnel vision on a discarded hypothesis.
- Return to brute force: bisect the change history, or delete code until it works and add it
  back until it breaks.
