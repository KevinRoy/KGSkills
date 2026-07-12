# Root Cause Playbook

Use this reference to convert ANR evidence into fixes and validation.

## Diagnosis Patterns

Main-thread disk or DB:

- Evidence: main stack in SQLite, Room, file APIs, SharedPreferences, asset read, migration, or serialization.
- Fix: move work off main, prefetch, cache, split migration, use async APIs, or defer until after first draw.
- Validate: StrictMode, regression trace, startup benchmark, interaction latency test.

Main-thread network or DNS:

- Evidence: socket, DNS, HTTP client, WebView/network bridge, or blocking SDK init on main.
- Fix: remove synchronous network from lifecycle/input paths; use async request with timeout and cancellation.
- Validate: network-offline test, slow DNS simulation, ANR watchdog, UI automation under poor network.

Synchronous Binder or ContentProvider:

- Evidence: `BinderProxy.transact`, `ContentResolver`, `PackageManager`, media/camera/location/settings, or app AIDL on main.
- Fix: move IPC off main, cache results, add timeout/fallback, avoid nested Binder calls while holding locks.
- Validate: fake slow provider/service, Perfetto Binder slices, integration test with delayed remote process.

Lock contention:

- Evidence: main thread `BLOCKED` or waiting for monitor/lock, owner thread visible doing slow work or waiting back.
- Fix: reduce lock scope, avoid holding locks across I/O/Binder/callbacks, use immutable snapshots, reorder locks consistently.
- Validate: stress test concurrent path, add lock timing logs, thread dump under load.

Deadlock:

- Evidence: cycle between main thread and one or more threads/locks/Binder callbacks.
- Fix: break synchronous callback cycle, avoid main-thread waits, define lock ordering, replace blocking waits with async continuation.
- Validate: reproduce with deterministic scheduling or stress loop; ensure no cycle in fresh trace.

CPU starvation:

- Evidence: high runnable load, main runnable but delayed, many busy workers, Perfetto scheduler delay, high process/device CPU.
- Fix: cap thread pools, remove busy loops, add backoff, reduce priority, chunk CPU work, use WorkManager/job constraints.
- Validate: profiler/Perfetto before and after, interaction test under load.

Startup ANR:

- Evidence: Application/Activity/provider init path, DI graph, SDK initialization, class loading, DB migration, first render blocked.
- Fix: lazy initialize, defer noncritical SDKs, move disk/DB off main, warm only critical path, reduce provider side effects.
- Validate: cold-start benchmark, first-frame timing, ANR trace on low-end device.

Broadcast/service ANR:

- Evidence: receiver/service lifecycle stack or log reason.
- Fix: keep lifecycle callback short; schedule WorkManager/job/service worker; call `goAsync().finish()`; avoid main-thread blocking.
- Validate: test receiver/service under slow dependencies and process cold start.

## Fix Review Checklist

- Remove or bound the main-thread wait.
- Avoid moving the same blocking work to another path that still blocks UI.
- Add timeout, cancellation, and fallback for remote or slow dependencies.
- Avoid holding locks while calling into app callbacks, Binder, I/O, or code that can re-enter.
- Add observability: operation timing, ANR breadcrumbs, StrictMode in debug, and sampled traces for risky paths.
- Validate on low-end hardware or constrained CPU/I/O where possible.

## Response Guidance

When evidence is incomplete, say what is known and unknown:

- Known: exact main-thread state and stack.
- Probable: most likely blocking operation.
- Unknown: lock owner, Binder peer, CPU load, or source mapping.

Recommend the smallest next artifact that would raise confidence, such as:

- full `/data/anr` trace
- 30 seconds of logcat around the event
- bugreport section with ActivityManager CPU summary
- Perfetto trace for repeated runnable stalls
- source for the blocking method
