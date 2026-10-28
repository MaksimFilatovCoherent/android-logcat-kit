# android-logcat-kit

A small [Claude Code](https://docs.claude.com/en/docs/claude-code/plugins) plugin for Android developers: it reads `adb logcat` output and maps crashes, ANRs, StrictMode violations and Compose jank to your Kotlin, Jetpack Compose and Hilt code.

| Component | What it does |
|---|---|
| `/logcat-triage <applicationId>` | captures a bounded logcat dump from a connected device or emulator, classifies the failure, finds the source line, proposes a diff (read-only) |
| `logcat-reading` skill | signatures for `FATAL EXCEPTION`, coroutine and Hilt errors, Compose state errors, ANR, StrictMode, `Skipped N frames`, native crashes |

## Install

This repository is a plugin (the plugin manifest is at `.claude-plugin/plugin.json`). It is consumed as a GitHub-hosted plugin source by the `maksim-android-workbench` marketplace:

```
/plugin install android-logcat-kit@maksim-android-workbench
```

Requires `adb` on `PATH` and a device or emulator.

## License

MIT
