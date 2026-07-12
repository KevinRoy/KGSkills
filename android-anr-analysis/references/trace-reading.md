# ANR Trace Reading

Use this reference to inspect `/data/anr` traces, Play Console ANR stacks, bugreport ANR sections, or pasted thread dumps.

## Fast Path

1. Match package, pid, and timestamp to the reported ANR.
2. Find `"main"` or `main` thread in the target process.
3. Record:
   - Thread state: `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, native wait, sleeping, or Binder wait.
   - Top app-owned frames.
   - Any `waiting to lock`, `locked`, `parking to wait`, `Object.wait`, `Condition.await`, `CountDownLatch.await`, `Future.get`, `join`, `runBlocking`, or Binder transaction frame.
4. Scan non-main threads for:
   - Lock owners.
   - Busy CPU loops.
   - Binder pool exhaustion.
   - DB, disk, network, class loading, image decode, or startup work.
5. Build a blocking chain from main thread to owner/remote work. Stop when evidence runs out.

## Thread State Interpretation

`BLOCKED`:

- Usually waiting to enter a Java monitor.
- Search the trace for the monitor address or matching `locked` line.
- If the owner is waiting for the main thread or another lock in the chain, suspect deadlock.

`WAITING` or `TIMED_WAITING`:

- Look for `Object.wait`, `LockSupport.park`, `Condition.await`, `CountDownLatch.await`, `Semaphore.acquire`, `Thread.join`, `FutureTask.get`, coroutine blocking, or message queue waits.
- Main thread waiting on app-controlled async work is usually an app bug, even if the worker is slow rather than deadlocked.

`RUNNABLE`:

- Could be executing CPU-heavy work, stuck in a native call, or simply sampled while runnable.
- Correlate with CPU usage, repeated samples, Perfetto, or other busy threads before declaring CPU starvation.

Native or Binder wait:

- Frames such as `BinderProxy.transact`, `IPCThreadState`, `ContentResolver`, `PackageManager`, `Settings`, media/camera/location services, or app-defined AIDL suggest synchronous IPC.
- Inspect Binder pool threads and logcat for slow remote service, provider, or system_server issues.

Message queue idle:

- If main thread appears idle, verify the trace matches the ANR process and timestamp.
- For input/no focused window ANRs, the issue can be window focus, launch, rendering, or system_server state rather than an obvious app stack.

## Common Main Thread Culprits

- Disk or database operations: SQLite query, Room migration, SharedPreferences commit/read, file read/write, asset loading.
- Network or IPC: blocking HTTP, DNS, ContentProvider query, PackageManager call, synchronous Binder to app or system service.
- Startup and lifecycle: Application/Activity initialization, dependency injection graph creation, SDK initialization, dynamic feature/class loading.
- UI work: RecyclerView diff/layout on main, bitmap decode/resize, WebView initialization, large JSON/protobuf parsing, encryption/compression.
- Blocking async bridges: `runBlocking`, `blockingGet`, `Future.get`, `await`, `join`, `CountDownLatch.await`.
- Locks: synchronized singleton init, cache locks, DB locks, logging locks, analytics SDK locks.

## Evidence Quality

High confidence:

- Main thread blocked in app code and owner thread or operation is visible.
- Logs show the slow operation starting before ANR and ending after it.
- Multiple traces or Perfetto confirm the same blocking path.

Medium confidence:

- Main thread stack clearly shows risky synchronous work, but owner/timing is incomplete.
- Logs show related warnings without complete timing.

Low confidence:

- Only one truncated Play Console stack is available.
- Main thread looks idle or generic without matching logs.
- ANR reason, pid, or timestamp is uncertain.
