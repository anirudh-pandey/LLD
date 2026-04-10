# Async/Await

## In One Line
Async/await lets your code release the thread while waiting for slow operations (I/O), so the system can do more work with fewer threads.

## Core Idea
When code calls something slow — a database, an API, a file — the thread normally sits idle, blocked, doing nothing.

Async/await frees that thread. It says "go serve someone else, I'll call you back when my result is ready." The compiler rewrites your method into a state machine that saves its position at each `await`, returns, and resumes later when the operation completes.

No extra threads are created. The I/O operation is handled by the OS/hardware. The thread is simply released.

## Why Do We Need It?
In a web server, each request uses a ThreadPool thread. If 500 requests are all blocked waiting on a database, you need 500 threads doing nothing. You run out of threads, new requests queue up, throughput drops.

With async, those 500 threads are released back to the pool while waiting. A handful of threads can now serve thousands of concurrent requests.

In UI apps (WPF/WinForms), the single UI thread must stay free to render and handle user input. Without async, a slow HTTP call freezes the entire window.

## What Goes Wrong Without It?
The server runs out of ThreadPool threads under load. Requests start queueing. Response times spike. Users see timeouts.

In UI apps, the window becomes unresponsive. The user sees "Not Responding" because the UI thread is stuck waiting on I/O.

Developers try to fix this with `.Result` or `.Wait()`, which blocks the thread and can cause deadlocks in environments with a SynchronizationContext.

## Simple Example
Bad design — blocking the thread:

```csharp
public string GetData()
{
    var result = httpClient.GetStringAsync(url).Result; // thread blocked, deadlock risk
    return result;
}
```

Better design — async all the way:

```csharp
public async Task<string> GetDataAsync()
{
    var result = await httpClient.GetStringAsync(url); // thread released during wait
    return result;
}
```

The thread is freed during the HTTP call. When the response arrives, execution resumes.

## Key Rules
`async` marks a method for the compiler to build a state machine. It does not create a thread.

`await` pauses execution and releases the thread. It does not block.

`Task` / `Task<T>` is a promise that work will complete in the future.

`async void` is only for UI event handlers. Everywhere else use `async Task`. An `async void` method swallows exceptions and cannot be awaited.

Always `await` a returned Task. A missing `await` means the operation runs unsupervised — exceptions are silently lost, and code continues before the operation finishes.

## Concurrent vs Parallel
Concurrent: one thread, many I/O operations in-flight. Like one cook managing multiple dishes during wait times. Use `Task.WhenAll`.

Parallel: multiple threads doing CPU work at the same time. Like four cooks each chopping vegetables simultaneously. Use `Task.Run` or `Parallel.For`.

`Task.Run` is for CPU-bound work. Using it for I/O-bound work wastes a thread that does nothing but wait.

## SynchronizationContext and ConfigureAwait
UI frameworks (WPF/WinForms) have a SynchronizationContext that makes continuations resume on the UI thread, because UI controls are not thread-safe — only the creating thread can touch them.

This "come back to original context" has a cost: capture context, queue to the dispatcher, wait for the UI thread to be free. `ConfigureAwait(false)` skips this — the continuation runs on any available thread. Use it in library code.

ASP.NET Core has no SynchronizationContext. Continuations resume on any ThreadPool thread. Deadlocks from `.Result`/`.Wait()` don't happen, but blocking still wastes threads and hurts throughput.

## CancellationToken
A shared signal that says "please stop." `CancellationTokenSource` is the remote control, `CancellationToken` is the read-only flag passed to async methods.

When cancelled, methods throw `OperationCanceledException`. In ASP.NET Core, the framework provides a token that fires when the client disconnects — pass it through to avoid wasted work.

## ValueTask
`Task<T>` allocates on the heap every time. `ValueTask<T>` avoids allocation when the result is already available (e.g., cache hit). Use in hot paths. Can only be awaited once.

## Real Meaning
Async/await does not make code faster. It makes code more scalable. A single operation takes the same time. But the system handles more concurrent work because threads are not wasted waiting.

It is not multithreading. No new threads are created for I/O. The OS and hardware handle the waiting.

## Strong Mental Model
Ask: is this thread doing useful CPU work right now, or is it just sitting idle waiting for something external?

If it is sitting idle, it should be released with `await`.

## Interview-Ready Definition
Async/await is a language feature that lets methods release the thread while waiting for I/O operations. The compiler transforms the method into a state machine that saves its position at each await point. When the operation completes, a continuation resumes execution. This means fewer threads can handle more concurrent work, improving scalability in servers and responsiveness in UI apps.

## Interview-Ready Why
We need async/await because blocking threads during I/O is wasteful. In web servers, blocked threads reduce throughput under load. In UI apps, blocking the UI thread freezes the interface. Async frees the thread during the wait so it can serve other requests or keep the UI responsive.

## Common Interview Questions
**Does async/await create new threads?**
No. It releases the current thread during I/O. The operation is handled by OS/hardware. No thread is consumed while waiting.

**What is the danger of `.Result` or `.Wait()`?**
They block the thread. In environments with a SynchronizationContext (WPF, old ASP.NET), this causes deadlocks. In ASP.NET Core, no deadlock but you waste a thread.

**When should you use `Task.Run`?**
Only for CPU-bound work you want off the current thread (e.g., off the UI thread). Never for I/O — just await the async method directly.

**Why avoid `async void`?**
It cannot be awaited, so the caller cannot know when it finishes. Exceptions crash the process instead of being catchable. Only valid for UI event handlers.

**What is `ConfigureAwait(false)`?**
It tells the continuation not to marshal back to the original SynchronizationContext. Use in library code. Irrelevant in ASP.NET Core (no SyncContext).

## Deadlock Deep-Dive: `.Result` on an Async Method

### The Classic Deadlock Pattern
```csharp
public IActionResult GetData()
{
    var result = GetDataFromService().Result; // blocking call
    return Ok(result);
}

public async Task<string> GetDataFromService()
{
    await Task.Delay(2000); // async wait
    return "Done.";
}
```

### Step-by-Step: .NET Framework (ASP.NET) — DEADLOCK

1. HTTP request arrives. ASP.NET assigns **Thread A** and its `AspNetSynchronizationContext` to handle the request.
2. `GetData()` runs on Thread A.
3. `GetDataFromService()` is called. Starts running on Thread A.
4. `await Task.Delay(2000)` is hit. The `await` **captures the current SynchronizationContext** (the one tied to this request). `Task.Delay` sets a timer and returns an incomplete Task.
5. Execution returns to `GetData()`. `.Result` is called — **Thread A is now blocked**, sitting idle, waiting for the Task to complete.
6. 2000ms later, the timer fires. The continuation (`return "Done."`) is ready to run.
7. The continuation checks: "where should I resume?" — it captured the `AspNetSynchronizationContext`, which says: **"resume on Thread A, in the context of this request."**
8. But Thread A is **blocked** waiting for `.Result`. The continuation cannot be scheduled. Thread A cannot unblock until the continuation runs. **DEADLOCK.**

The continuation needs Thread A. Thread A is waiting for the continuation. Neither can proceed.

### Step-by-Step: .NET Core (ASP.NET Core) — NO DEADLOCK

1. HTTP request arrives. ASP.NET Core assigns **Thread A** (from the ThreadPool) to handle it.
2. `GetData()` runs on Thread A.
3. `GetDataFromService()` is called. Starts running on Thread A.
4. `await Task.Delay(2000)` is hit. In ASP.NET Core, **there is no SynchronizationContext** — nothing is captured. `Task.Delay` sets a timer and returns an incomplete Task.
5. Execution returns to `GetData()`. `.Result` is called — **Thread A is now blocked**, sitting idle.
6. 2000ms later, the timer fires. The continuation (`return "Done."`) is ready to run.
7. The continuation checks: "where should I resume?" — no context was captured, so it just needs **any free ThreadPool thread**.
8. **Thread B** (any available thread) picks up the continuation and executes `return "Done."`. The Task completes with "Done.".
9. Thread A (blocked on `.Result`) sees the Task is complete. It unblocks and receives "Done.".
10. `GetData()` returns `Ok("Done.")`. **No deadlock** — just Thread A was wasted/blocked for 2 seconds.

### Why the Difference?
| | .NET Framework | .NET Core |
|---|---|---|
| SynchronizationContext | `AspNetSynchronizationContext` (per-request, single-threaded) | None |
| `await` captures context? | Yes — continuation must return to the **same context thread** | No — continuation runs on **any ThreadPool thread** |
| `.Result` blocks context thread? | Yes → continuation can never schedule → **DEADLOCK** | Thread blocked, but continuation uses a different thread → **no deadlock** |

### The Root Cause
The deadlock is not caused by `.Result` alone. It is caused by the combination of:
- `.Result` blocking a thread that **owns a single-threaded context**, and
- `await` capturing that context so the continuation **demands that exact thread back**

Both conditions must be true for deadlock. .NET Core breaks the second condition.

### Fixes (best to worst)

**Best: async all the way**
```csharp
public async Task<IActionResult> GetData()
{
    var result = await GetDataFromService(); // no blocking, no wasted thread
    return Ok(result);
}
```

**Acceptable: ConfigureAwait(false) (useful in library code)**
```csharp
public async Task<string> GetDataFromService()
{
    await Task.Delay(2000).ConfigureAwait(false); // don't capture context
    return "Done.";
}
```
Continuation will not try to resume on the original context. Breaks the second deadlock condition. Fixes .NET Framework deadlock.

**Avoid: `.GetAwaiter().GetResult()`**
Same blocking semantics as `.Result`. Still deadlocks in .NET Framework. Only difference: unwraps `AggregateException` so you get the original exception directly.

---

## Fast Recall
Think: thread waiting idle → release it with await.
Think: `async void` = fire and forget with no safety net. Use `async Task`.
Think: `Task.Run` = CPU work. `await` = I/O work. Don't mix them.
Think: `.Result` + SynchronizationContext = deadlock trap. Async all the way = safe.

## Questions To Ask In Design
Is this operation I/O-bound or CPU-bound?
Can I fire multiple independent async calls concurrently with Task.WhenAll?
Am I passing CancellationToken through the entire call chain?
Am I blocking anywhere with .Result or .Wait() instead of awaiting?
