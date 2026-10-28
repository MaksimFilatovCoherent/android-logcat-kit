---
name: logcat-reading
description: Use when reading Android logcat output or a pasted stack trace - finding the root cause of a FATAL EXCEPTION, ANR, native crash, StrictMode violation or Choreographer "Skipped frames" warning, with Kotlin coroutines, Jetpack Compose and Hilt specific signatures.
---

# Reading Android logcat

## Getting the right slice

| Goal | Command |
|---|---|
| Dump and exit (never stream in an agent) | `adb logcat -d -v threadtime -t 1500` |
| Only the app's process | `adb logcat -d -v threadtime --pid=$(adb shell pidof -s <applicationId>)` |
| Only crashes | `adb logcat -d -s AndroidRuntime:E` |
| One specific device | `adb -s <serial> logcat -d ...` |
| Crash buffer (survives app restarts) | `adb logcat -d -b crash` |

`-v threadtime` prints date, time, pid, tid, level and tag, so the thread of a failure is visible.

## Signatures and what they mean

| Log pattern | Meaning and where to look |
|---|---|
| `FATAL EXCEPTION: main` + `Process: <pkg>, PID: n` | Uncaught exception on the main thread. Read the **last** `Caused by:`, then the first frame in your package. |
| `FATAL EXCEPTION: DefaultDispatcher-worker-N` | Uncaught exception in a coroutine on `Dispatchers.Default/IO` with no `CoroutineExceptionHandler`. Check `launch` without try/catch in `viewModelScope`. |
| `kotlinx.coroutines.JobCancellationException` | Usually not a bug: a cancelled scope. It is a bug when it is caught and swallowed by a generic `catch (e: Exception)` - rethrow `CancellationException`. |
| `UninitializedPropertyAccessException: lateinit property` | Access before initialisation; common with Hilt field injection used before `super.onCreate()` or in a non-`@AndroidEntryPoint` class. |
| `java.lang.RuntimeException: Cannot create an instance of class ...ViewModel` / `does not have a zero-argument constructor` | Missing `@HiltViewModel` or `@AndroidEntryPoint` on the host Activity. |
| `IllegalStateException: Reading a state that was created after the snapshot was taken` | Compose state mutated or read from a background thread outside a snapshot; move the write to the main dispatcher or use `StateFlow`. |
| `IllegalStateException: CompositionLocal ... not present` | A composable runs outside the provider (preview or test without the theme / `CompositionLocalProvider`). |
| `NetworkOnMainThreadException` | Retrofit/OkHttp call on the main thread; use a suspend function on `Dispatchers.IO`. |
| `ANR in <pkg> (<pkg>/.Activity)` + `Reason: Input dispatching timed out` | Main thread blocked > 5 s. Look at the `main` thread stack in the surrounding `Cmd line` / `"main" tid=1` dump for the blocking call (disk, network, lock, `runBlocking`). |
| `StrictMode policy violation; ~duration=N ms: ... DiskReadViolation` | Disk I/O on the main thread (SharedPreferences, Room without a background executor). |
| `Choreographer: Skipped N frames! The application may be doing too much work on its main thread.` | N > 30 is visible jank. Look for heavy work in composition, `remember` missing around expensive calls, large unkeyed `LazyColumn` items. |
| `*** *** ***` / `signal 11 (SIGSEGV)` / `backtrace:` | Native crash. Needs symbolication with `ndk-stack` and the unstripped `.so`; the Kotlin frames above it show the JNI entry point. |
| `Out of memory` / `OutOfMemoryError: Failed to allocate` | Bitmaps (use Coil/Glide sampling), leaking `Context`, unbounded caches. |

## Reading a stack trace

1. Start from the bottom: the **last** `Caused by:` is the root cause; everything above it is wrapping.
2. The first frame whose package starts with the app's namespace is where to read code; frames in `androidx.compose.runtime`, `kotlinx.coroutines` or `dagger.hilt` are infrastructure.
3. Obfuscated frames (`a.b.c`) mean a release build: the `mapping.txt` from `build/outputs/mapping/release/` retraces them (`retrace mapping.txt trace.txt`).

## Privacy

Logs may include auth tokens, e-mail addresses and user ids. Redact them before quoting a log in an answer, an issue or a commit message.
