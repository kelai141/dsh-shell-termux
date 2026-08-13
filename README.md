# @dsh-android/dsh-shell-termux

Android/Termux service provider for the bash capability seam (`ctx.shell`).

## What it does

- **Explicit Termux environment**: every spawn injects `PATH`/`LD_LIBRARY_PATH`/`HOME`/`PREFIX`/`TERMUX_VERSION`/`SHELL` from config — execution never depends on the ambient environment being accidentally correct (M0 lesson: PATH expanded by the launcher breaks).
- **Reuses local mechanics**: extends `LocalBashExecutor` (`runArgv`/`startArgv`), inheriting process-group SIGTERM→SIGKILL escalation, output caps + spill, grace period, background lifecycle, and teardown ownership unchanged.
- **Honest sandbox declaration**: `sandboxMode = 'workspace-write'` (so permission presets mount), with per-process `enforcement: 'partial'` — the protection boundary is the Android app domain (SELinux u0_aXXX) plus the approval flow, not a path-level confiner. No fake sandboxing.
- **Probe diagnostics**: `probe()` reports bash presence/version and missing toolchain (`pkg install bash coreutils findutils grep` hints). Misconfigured bash fails loud with repair guidance.

## Config

| key | meaning | default |
|---|---|---|
| `bashPath` | absolute bash binary path | required |
| `prefix` | Termux prefix root (contains bin/ lib/) | required |
| `home` | Termux home directory | required |
| `termuxVersion` | TERMUX_VERSION value | `0.118.3` |
| `extraPath` | extra PATH entries prepended | `[]` |
| `cwd`/`timeoutMs`/`maxTimeoutMs`/`maxOutputBytes`/`maxSpillBytes`/`graceMs` | inherited local-executor knobs (editable via shell settings) | mirror bash-local |

## Mounting (android profile patch)

```yaml
- id: bash-sandbox
  disabled: true
- id: bash-local
  disabled: true
- insert:
    - id: shell-termux
      name: '@dsh-android/dsh-shell-termux'
      config:
        bashPath: /data/data/com.termux/files/usr/bin/bash
        prefix: /data/data/com.termux/files/usr
        home: /data/data/com.termux/files/home
```

## Known limitations

- PTY/persistent terminals remain unavailable on Android (node-pty not wired into `ctx.terminals`); pipe-mode spawns work.
- The shell settings section edits only the inherited knobs; `bashPath`/`prefix`/`home` are deployment config (patch-level).
- 16KB-page devices require a rebuilt environment (out of scope for the Termux form; see M2 build matrix).
