# Logcat Signals

Use this reference to correlate ANR traces with logcat, bugreports, DropBox metadata, and system logs.

## Search Terms

Use targeted searches around the ANR timestamp:

```text
ANR in
Application Not Responding
Input dispatching timed out
BroadcastQueue
Timeout executing service
ContentProvider not responding
am_anr
ActivityManager
InputDispatcher
WindowManager
system_server
StrictMode
Skipped frames
Choreographer
GC
WaitForGcToComplete
Binder
slow
timeout
```

## Key Lines to Extract

- ANR reason and package/process.
- `Load:` and CPU usage before/during/after ANR when present.
- `Reason:` lines such as input dispatch timeout, broadcast timeout, service timeout, or provider timeout.
- `Parent:` / `Window:` / focused activity or window information for input ANRs.
- `Broadcast of Intent` or receiver class for broadcast ANRs.
- Service class and lifecycle callback for service ANRs.
- Provider authority/class for provider ANRs.
- StrictMode violations for disk/network on main thread.
- GC pauses, memory pressure, low memory killer, or repeated allocation churn.
- App logs bracketing suspicious operations.

## Interpreting ANR Types

Input dispatch timeout:

- Check whether main thread is blocked, window focus is missing, or system_server/input pipeline is delayed.
- Look for preceding tap, launch, dialog, activity transition, or surface/window logs.

Broadcast timeout:

- Identify receiver and whether work was done directly in `onReceive`.
- Watch for synchronous network, DB, Binder, or long initialization.
- Confirm whether `goAsync()` was used correctly and finished.

Service timeout:

- Identify `onCreate`, `onStartCommand`, `onBind`, foreground service start, or job/service bridge.
- Check slow startup, notification setup, Binder calls, or initialization on main.

ContentProvider timeout:

- Check provider `onCreate`, query/insert/update/delete, DB migrations, cross-process provider calls, and app startup side effects.

No focused window:

- Correlate activity launch, window attach, rendering, splash, and focus transitions.
- Main thread may be busy before first draw, or the issue may be system/window state.

## CPU and Scheduler Clues

High CPU in app:

- Look for worker pools, render threads, native loops, image/video processing, compression, crypto, or retry loops.
- If main thread is `RUNNABLE`, Perfetto or repeated samples are stronger evidence than one trace.

High CPU outside app:

- Device-wide load can delay app response; still identify whether app code made the ANR likely.
- Mention environmental contribution separately from code root cause.

Low CPU with blocked main thread:

- More consistent with lock, I/O wait, Binder wait, or sleeping/waiting on async work.

## Bugreport Notes

Bugreports can contain multiple ANRs. Always match:

- package/process name
- pid
- timestamp
- ANR reason
- trace file section

Avoid mixing evidence from different ANR events unless they show the same repeated pattern.
