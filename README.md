# vibe-kernel-miatoll

LineageOS 22.2 kernel for the Redmi Note 9 Pro (joyeuse / miatoll),
with the FT8756 touch fix for the replacement screen, built with
ThinLTO on GitHub Actions.

## Why this exists

The phone has an aftermarket replacement screen with a FocalTech
FT8756 touch chip (Huaxing, SPI). On LineageOS the touch is dead
after every boot until the screen is toggled off/on. The replacement
panel merges touch/display lines, so the touch IC needs a hardware
reset after the boot-time firmware download before it will accept
register writes. The patch in this repo schedules a delayed work
~20 s after probe that pulses the touch reset GPIO, then runs the
suspend→resume path, so touch comes up without a manual screen
off/on. The firmware re-flash stays on (stopping it broke the
screen off/on recovery too).

Status: the touch fix is still being tested on-device. v6 is the
current attempt (v5 booted but touch stayed dead).

## Variants — three builds, one patch

All three are built from the same touch-fix patch and the same
ThinLTO build (clang 19.0.0 r530567, kernel 4.14.336). Pick one:

| Variant | Branch | Release | What you get |
|---|---|---|---|
| **Clean** | `main` | [v1.0-lto](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto) | Touch fix only. Flashing it removes the APatch kernel patch, so root is lost until the new boot.img is re-patched with the APatch app and re-flashed. |
| **Root** | `root` | [v1.0-lto-root](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-root) | Touch fix + kernel pre-patched with APatch kpimg 0.13.9. Root survives flashing; no re-patch needed. The APatch manager app must stay installed. |
| **DroidSpaces** | `droidspaces` | [v1.0-lto-droidspaces](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-droidspaces) | Everything in Root + full DroidSpaces container support (namespaces, cgroups, seccomp, NAT, overlayfs, user namespaces + the non-GKI cgroup patch). Two guide options could not be enabled (see changelog): CFS_BANDWIDTH (blocked by Qualcomm's SCHED_WALT) and CGROUP_NS (does not exist in this 4.14 tree). |

## Changelog

Each release page carries its own changelog:

- **v1.0-lto** — the base build: FT8756 touch fix (probe sets
  `suspended=true` so the first display-on runs the full resume,
  in-cell reset + `fts_wait_tp_to_valid()` settle restored, read-path
  `fts_tp_state_recovery()` safety net), ThinLTO ON, stock
  `vendor/xiaomi/miatoll_defconfig`.
- **v1.0-lto-root** — everything in v1.0-lto, plus the built Image
  pre-patched with KernelPatch kpimg 0.13.9 (verified `patched=true`,
  kpimg `0xd09`). No custom superkey. Zip
  `miatoll-ft8756-touchfix-lto-root.zip`, 21,984,545 bytes,
  md5 `2cb4fcc239ce3c220696ce583531fb2e`.
- **v1.0-lto-droidspaces** — everything in v1.0-lto-root, plus the
  DroidSpaces kernel support from the Droidspaces-OSS non-GKI guide
  (kernels 3.18–4.19; miatoll is 4.14): the required options merged
  from `ci/droidspaces.config` (namespaces, cgroups, seccomp,
  netfilter/NAT, overlayfs, user namespaces, UFW/Fail2Ban firewall
  options) and the non-GKI cgroup file-prefix patch
  (`ci/droidspaces-02-cgroup-prefix.patch`). The guide's patch 01
  (xt_qtaguid panic fix) is intentionally skipped:
  `net/netfilter/xt_qtaguid.c` does not exist in this tree (the
  driver was removed), so the panic it fixes cannot occur. Two
  options in the fragment cannot take effect on 4.14
  (`CONFIG_FW_LOADER_COMPRESS`, added in 5.3; `CONFIG_CGROUP_NET_PRIO`,
  removed in 4.5) — the build warns about them and ignores them.
  Two more options could not be enabled at all, verified from the
  config extracted from the built Image (19 of 21 key options `=y`):
  `CONFIG_CFS_BANDWIDTH` is blocked by `CONFIG_SCHED_WALT`
  (Qualcomm's scheduler — the `--cpus` limit is unavailable, which
  the guide says is skipped with a warning), and `CONFIG_CGROUP_NS`
  does not exist in this 4.14 tree. Both are optional per the guide.

## Files

- `ft8756-touch-fix.patch` — the source patch (self-contained,
  includes the firmware include files the driver needs to compile).
- `.github/workflows/build-kernel.yml` — the CI build (ThinLTO ON).
  Each branch builds its own variant; the workflow triggers on push
  to its branch and on manual dispatch.
- `ci/anykernel.sh` — the AnyKernel3 flash script (kernel only).
- `ci/droidspaces.config` — DroidSpaces config fragment (droidspaces
  branch only).
- `ci/droidspaces-02-cgroup-prefix.patch` — DroidSpaces non-GKI
  cgroup patch (droidspaces branch only).

## Build

Push to a branch, or run the workflow manually (Actions → Run
workflow, pick the branch). The workflow clones the LineageOS kernel
(lineage-22.2 branch tip), applies the touch-fix patch (plus the
DroidSpaces patch and config on the droidspaces branch, plus the
APatch pre-patch on the root and droidspaces branches), builds with
the stock `miatoll_defconfig` (ThinLTO already on), and uploads the
AnyKernel3 zip as an artifact.

The ThinLTO final link needs ~16 GB RAM, which is why the build
runs on GitHub (a standard runner) instead of a small VM.

## Flash

Back up the Boot partition in recovery first. Flash the zip from
recovery (it replaces only the kernel Image), reboot, and touch
should work on the first boot with no screen toggle.

- **Clean (v1.0-lto):** root is lost after flashing. Dump the new
  boot.img from recovery (`adb shell dd if=/dev/block/bootdevice/by-name/boot
  of=/sdcard/boot-fixed.img`), patch it with the APatch app, and
  `fastboot flash boot` it to get root back.
- **Root / DroidSpaces (v1.0-lto-root, v1.0-lto-droidspaces):** root
  survives flashing. Keep the APatch manager app installed.

Revert: restore the Boot backup from recovery.

## DroidSpaces check (droidspaces variant only)

After flashing, open the DroidSpaces app → Settings → Requirements →
Check Requirements. The must-have and recommended items should pass,
except the two known gaps (CFS_BANDWIDTH and CGROUP_NS, see the
changelog above).

## Status

Touch fix v6 is still being tested on-device (touch stays dead after
boot so far). This repo stays on the default ft8756_spi driver.
DroidSpaces + APatch are confirmed working on-device.
