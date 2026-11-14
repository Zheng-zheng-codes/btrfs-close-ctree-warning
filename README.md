# WARNING in close_ctree (fs/btrfs/disk-io.c:4546, v7.4-rc2)

Syzkaller-found bug on Linux v7.4-rc2, reproduced and verified on the same
kernel build (fuzzing pipeline reproducer: confirmed=true, status=verified,
attempt 4 with 150-program state recipe).

## Summary

A warning fires in `close_ctree()` (fs/btrfs/disk-io.c:4546) during btrfs
unmount:

```
RIP: 0010:close_ctree+0x11a9/0x16a0 fs/btrfs/disk-io.c:4546
```

`close_ctree()` is the btrfs unmount handler. The warning fires during the
filesystem shutdown sequence when the crafted image's on-disk/in-memory
state reaches an unexpected condition.

## How to reproduce

Environment: Debian trixie x86_64 guest (6 GiB RAM, 4 vCPU, KVM), kernel
built from v7.4-rc2 with CONFIG_KASAN=y, CONFIG_BTRFS_FS=y, booted with
`oops=panic` and `panic_on_warn=1`. See `environment.txt` for the exact
QEMU command line and `kernel.config` for the configuration.

The crash depends on btrfs state built up by the preceding programs. The
reproducer is a multi-program syzkaller execution log (150 programs):

```sh
# in the guest, as root:
syz-execprog -procs=8 -threaded -repeat=0 -executor ./syz-executor repro.kernsight.prog
```

Verified by the fuzzing pipeline's reproducer machinery on a fresh VM
(reproducer pipeline outcome: confirmed=true, status=verified). A C
reproducer has not been included yet.

## Files

- `kernel.config` — kernel configuration used (also required to interpret addresses)
- `environment.txt` — kernel image info (sha256), QEMU command line, syzkaller commit, reproducer command
- `reproducer/repro.kernsight.prog` — pipeline-verified reproducer (150-program ordered set)
- `reproducer/repro.prog` — minimal triggering testcase (state-dependent; does not reproduce alone)
- `reproducer/repro.priming.prog` — state-priming program set (pipeline repro recipe)
- `logs/log-*.txt`, `logs/report-*.txt` — original syzkaller execution logs and crash reports from the fuzzing pipeline
- `logs/machineInfo-*.txt`, `logs/title-stat.json` — guest environment info and title statistics

## Notes for maintainers

- The warning is in the unmount path (`close_ctree()`); the crafted image
  contents drive the btrfs shutdown sequence into an unexpected state.
- We can provide the kernel image, additional crash instances or raw
  artifacts on request.
