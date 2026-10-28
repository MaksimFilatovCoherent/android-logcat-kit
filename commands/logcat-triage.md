---
description: Capture recent logcat from a connected Android device or emulator and diagnose the crash, ANR, StrictMode violation or Compose jank in it
argument-hint: "<applicationId, e.g. com.techcorp.shop> [device serial]"
allowed-tools: Read, Grep, Glob, Bash(adb devices:*), Bash(adb logcat:*), Bash(adb shell pidof:*), Bash(find:*)
---

# /logcat-triage - diagnose a failure from logcat

Diagnose problems of the app **`$ARGUMENTS`** (first word: `applicationId`; optional second word: device serial passed as `adb -s <serial>`). Read-only: never clear the log buffer, uninstall, or edit files.

## Steps

1. **Pick the device.** Run `adb devices`. If none is in state `device`, stop and say how to start an emulator or authorise USB debugging. If several are connected and no serial was given, ask which one.
2. **Capture, do not stream.** Use the dump flag so the command returns:
   - Process-scoped: `adb logcat -d -v threadtime --pid=$(adb shell pidof -s <applicationId>) -t 1500`
   - If the process is dead (it crashed), the pid is gone: use `adb logcat -d -v threadtime -t 3000` and filter by the package name and by `AndroidRuntime`, `ActivityManager`, `StrictMode`, `Choreographer`.
   Never run `adb logcat -c` (it destroys the evidence) unless the user asks.
3. **Classify** using the `logcat-reading` skill, into one primary class: **crash** (`FATAL EXCEPTION`), **ANR** (`ANR in`), **native crash** (`signal 11`, `backtrace:`), **StrictMode violation**, **main-thread jank** (`Skipped N frames`), or **no problem found**.
4. **Map to source.** Take the last `Caused by:` and the first stack frame whose package starts with the app's namespace. Find it with Grep in the project (`class` name -> file) and read `file:line`. Quote the exact log line and the source line together.
5. **Redact before quoting.** Logs can contain tokens, e-mail addresses and user ids. Replace them with `<redacted>` in your answer.
6. **Propose a fix** as a diff, with one sentence on why it addresses the root cause. Do not apply it.

## Report format

```
CLASS:      <crash | ANR | native crash | StrictMode | jank | none>
DEVICE:     <model, Android version from the log header or adb devices -l>
SYMPTOM:    <one quoted log line>
ROOT CAUSE: <file:line and explanation>
FIX:        <diff>
VERIFY:     <how to reproduce and what the log should show afterwards>
```
