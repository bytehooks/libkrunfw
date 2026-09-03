# Prebuilt: linux-x86_64/libkrunfw.so.5.5.0

Built from this repo's `main` at commit `852cd72` (the four config commits
enabling `CONFIG_DEBUG_INFO_BTF`, `CONFIG_KPROBES`/`CONFIG_BPF_JIT`,
`CONFIG_FTRACE`, and `CONFIG_FTRACE_SYSCALLS` — see their commit messages
for the exact symptom each one fixes), on an Ubuntu 24.04 x86_64 host
(`glassbox-node-1`) via `make -j24`, kernel source `linux-6.12.91`.

sha256: `f3830017a7595b388eb7ae95292b6ba8d91a50914e95b73253b1496ac6ca24fb`

## Why this exists

`msb` dlopens `libkrunfw` by its **exact versioned filename**
(`libkrunfw.so.<FULL_VERSION>`), not through the generic `libkrunfw.so.<ABI>`
symlink — confirmed by checking a running sandbox's VMM process's
`/proc/<pid>/maps`. To use this build, overwrite the stock file at that
exact name (back it up first):

```sh
LIBDIR=~/.microsandbox/lib   # or $MSB_HOME/lib
cp "$LIBDIR/libkrunfw.so.<stock-version>" "$LIBDIR/libkrunfw.so.<stock-version>.orig-backup"
cp prebuilt/linux-x86_64/libkrunfw.so.5.5.0 "$LIBDIR/libkrunfw.so.<stock-version>"
```

Verified working end to end on `glassbox-node-1`: `agentsight record`
against a real sandbox produced non-empty `process_nodes`/`audit_events`
rows (real pid/ppid/exit code/duration for a traced process) — see
`glassbox-daemon`'s `architecture_plan.md` §10 for the full trial log.

**Also required at runtime, every sandbox, not baked into this binary**:
`mount -t tracefs nodev /sys/kernel/tracing` (and probably `mount -t
debugfs nodev /sys/kernel/debug`) before running anything that attaches
kprobe/uprobe/tracepoint BPF programs — the guest's minimal init doesn't
mount tracefs on its own.
