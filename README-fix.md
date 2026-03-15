# crun fork — cgroup root fix for CRIU re-checkpoint

This fork contains a minimal backport of upstream commit `c954b1b6` to crun 1.19.1, fixing CRIU re-checkpoint failures caused by stale cgroup namespace roots.

## Problem

After CRIU restores a container, re-checkpointing it fails with error -52 (-EBADE). The root cause: crun 1.19.1 does not pass `--cgroup-root` to CRIU during restore, so CRIU places the process in the OLD cgroup scope from the checkpoint image before calling `unshare(CLONE_NEWCGROUP)`. The cgroup namespace root becomes permanently stale, and any subsequent checkpoint fails the suffix validation in `proc_parse.c`.

## Fix

Branch `fix/cgroup-root-restore` (based on tag 1.19.1) adds `criu_add_cg_root(NULL, status->cgroup_path)` to the restore path in `src/libcrun/criu.c`. This tells CRIU to rewrite the checkpoint image's cgroup paths to match the new container's cgroup scope before restore, ensuring the cgroup namespace root is correct.

This fix was merged upstream in crun 1.21+ (commit `c954b1b6` by Giuseppe Scrivano, Feb 2025). **If you can upgrade to crun >= 1.21, use the upstream release instead of this fork.**

## Upstream references

- CRIU issue [#1793](https://github.com/checkpoint-restore/criu/issues/1793)
- crun issue [#1651](https://github.com/containers/crun/issues/1651)
- Upstream fix: [c954b1b6](https://github.com/containers/crun/commit/c954b1b6)
