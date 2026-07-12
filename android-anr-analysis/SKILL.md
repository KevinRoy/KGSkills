---
name: android-anr-analysis
description: Diagnose Android Application Not Responding issues from ANR traces, tombstones, bugreports, DropBox entries, logcat, Perfetto/Systrace notes, StrictMode logs, and source code. Use when Codex is asked to analyze, triage, summarize, or fix Android ANRs involving input dispatch timeouts, broadcast/service/content-provider timeouts, main-thread stalls, Binder waits, deadlocks, lock contention, slow I/O, CPU starvation, GC pauses, or app startup hangs.
---

# Android ANR Analysis

## Purpose

Analyze Android ANR evidence and produce a concise root-cause narrative, confidence level, supporting signals, and concrete fix plan.

Optimize for practical debugging: identify why the main thread could not respond in time, then map the blocking path back to code or system conditions.

## Intake Checklist

Ask for or inspect these artifacts when available:

- ANR trace: `/data/anr/anr_*`, bugreport `FS/data/anr/`, Play Console ANR stack, or OEM ANR attachment.
- Logcat around the ANR window, especially `ActivityManager`, `InputDispatcher`, `WindowManager`, `BroadcastQueue`, `system_server`, app logs, GC, StrictMode, Binder, and watchdog messages.
- App version, device model, Android version, foreground/background state, and user action.
- Relevant source code around the main-thread stack, locks, Binder calls, lifecycle callbacks, broadcast receivers, services, providers, and startup path.
- Perfetto/Systrace/CPU profiler data if the stack alone does not explain the stall.

Do not assume the first stack frame is the root cause. Treat the main-thread stack as the symptom location until cross-checked with held locks, Binder peers, CPU saturation, and surrounding logs.

## Workflow

1. Identify ANR type and deadline.
   - Input dispatch: user input not handled, commonly around 5 seconds.
   - BroadcastReceiver: `onReceive` exceeded its allowed window.
   - Service: foreground/background service lifecycle work exceeded system timeout.
   - ContentProvider: provider publish/query/init blocked.
   - No focused window or startup: app did not draw/respond during launch or focus transition.
2. Locate the app process and main thread in the trace.
   - Prefer the trace section matching package, pid, timestamp, and ANR reason.
   - Read the main thread state, top frames, monitor locks, Binder waits, native waits, and message loop position.
3. Classify the stall.
   - Main-thread long work: I/O, DB, JSON/protobuf parsing, bitmap work, crypto, compression, network, class loading, heavy initialization.
   - Lock contention or deadlock: main thread waiting for a monitor, mutex, condition, or coroutine/job while another thread waits back.
   - Binder or system call wait: main thread blocked on remote service, provider, package manager, content resolver, camera/media/location, or app-owned Binder.
   - CPU starvation: runnable main thread but little CPU due to saturation, priority inversion, thermal throttling, or runaway worker threads.
   - Runtime pressure: GC, finalizers, memory churn, JIT/class loading, or low-memory side effects.
4. Correlate with surrounding evidence.
   - Use logcat for ANR reason, timing, preceding warnings, slow operations, lifecycle events, and crashes.
   - Use other threads in the trace to find owners of locks, Binder endpoints, thread-pool exhaustion, or busy loops.
   - Use source code to map stack frames to risky synchronous operations.
5. Produce the diagnosis.
   - State root cause, confidence, exact blocking path, evidence, and missing data.
   - Separate immediate mitigation from durable fix.
   - Include validation steps to prove the fix removes the ANR path.

## Reference Routing

Read `references/trace-reading.md` when analyzing an ANR trace, Play Console stack, tombstone-like stack dump, or thread-state output.

Read `references/logcat-signals.md` when logcat, bugreport text, DropBox ANR metadata, system_server logs, or event logs are available.

Read `references/root-cause-playbook.md` when turning evidence into fixes, mitigations, regression tests, or code review guidance.

## Output Format

Prefer this structure:

```markdown
**Summary**
One-sentence diagnosis.

**Confidence**
High/Medium/Low, with why.

**Evidence**
- Timestamp, pid/package, ANR reason.
- Main thread state and key frames.
- Cross-thread, logcat, Binder, CPU, or lock evidence.

**Likely Root Cause**
Explain the blocking chain in order.

**Fix Plan**
- Immediate mitigation.
- Durable code change.
- Validation steps.

**Missing Data**
List only data that would materially change confidence.
```

## Guardrails

- Do not claim a deadlock unless there is a cycle or strong lock-owner evidence.
- Do not claim CPU starvation from a blocked stack alone; require runnable threads, high load, scheduler/perf evidence, or logs.
- Do not blame Android framework frames when app code synchronously invoked them from the main thread.
- Do not treat coroutine, Rx, Future, CountDownLatch, or `runBlocking` waits on the main thread as harmless.
- Prefer file, class, method, and line references when source code is available.
