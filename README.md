# vibe-kernel-miatoll

LineageOS 22.2 kernel for the Redmi Note 9 Pro (joyeuse / miatoll),
built with ThinLTO on GitHub Actions.

## Why this exists

LineageOS 22.2 kernel for the Redmi Note 9 Pro, with APatch and
DroidSpaces variants.

## Variants — three builds

All three are built from the stock LineageOS kernel (4.14.336) with
ThinLTO. Pick one:

| Variant | Branch | Release | What you get |
|---|---|---|---|
| **Clean** | `main` | [v1.0-lto](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto) | Stock kernel only (clang r530567). No root. Flashing removes the APatch patch, so root is lost until the new boot.img is re-patched with the APatch app. |
| **Root** | `root` | [v1.0-lto-root](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-root) | Stock kernel, built with clang r536225 (same as stock LineageOS) and CFI off. NOT pre-patched with APatch — patch the released boot.img yourself with the APatch app. |
| **DroidSpaces** | `droidspaces` | [v1.0-lto-droidspaces](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-droidspaces) | Stock kernel + full DroidSpaces container support (namespaces, cgroups, seccomp, NAT, overlayfs, user namespaces + the non-GKI cgroup patch) + APatch pre-patched (kpimg-android 0.13.9). Root survives flashing; no re-patch needed. Two guide options could not be enabled: CFS_BANDWIDTH (blocked by Qualcomm's SCHED_WALT) and CGROUP_NS (does not exist in this 4.14 tree). DroidSpaces + APatch confirmed working on-device. |

## Changelog

- **v1.0-lto** — the base build: stock kernel, ThinLTO ON,
  stock `vendor/xiaomi/miatoll_defconfig`, clang r530567.
- **v1.0-lto-root** — same stock kernel, clang r536225 (stock
  LineageOS toolchain) + CFI off. Unpatched: patch the released
  boot.img with the APatch app.
- **v1.0-lto-droidspaces** — everything in v1.0-lto-root, plus
  the DroidSpaces kernel support from the Droidspaces-OSS non-GKI
  guide (kernels 3.18–4.19; miatoll is 4.14): the required options
  merged from `ci/droidspaces.config` and the non-GKI cgroup
  file-prefix patch (`ci/droidspaces-02-cgroup-prefix.patch`).
  APatch pre-patched with kpimg-android 0.13.9 (the Android
  KernelPatch image — kpimg-linux, the Linux variant, hangs at
  the first logo). DroidSpaces + APatch confirmed working
  on-device. Two options could not be enabled: `CONFIG_CFS_BANDWIDTH`
  (blocked by `CONFIG_SCHED_WALT`) and `CONFIG_CGROUP_NS`
  (does not exist in 4.14). Both are optional per the guide.

## Files

- `.github/workflows/build-kernel.yml` — the CI build (ThinLTO ON).
  Each branch builds its own variant; the workflow triggers on push
  to its branch and on manual dispatch.
- `ci/anykernel.sh` — the AnyKernel3 flash script (kernel only).
- `ci/droidspaces.config` — DroidSpaces config fragment (droidspaces
  branch only).
- `ci/droidspaces-02-cgroup-prefix.patch` — DroidSpaces non-GKI
  cgroup patch (droidspaces branch only).
- `ci/apatch-compat.config` — CFI off (root and droidspaces
  branches only).

## Build

Push to a branch, or run the workflow manually (Actions → Run
workflow, pick the branch). The workflow clones the LineageOS kernel
(lineage-22.2 branch tip), applies the DroidSpaces patch and config
on the droidspaces branch (plus the APatch pre-patch on the
droidspaces branch only), builds with the
stock `miatoll_defconfig` (ThinLTO already on), and uploads the
AnyKernel3 zip as an artifact.

The ThinLTO final link needs ~16 GB RAM, which is why the build
runs on GitHub (a standard runner) instead of a small VM.

## Flash

Back up the Boot partition in recovery first. Flash the zip from
recovery (it replaces only the kernel Image), then reboot.

- **Clean (v1.0-lto) and Root (v1.0-lto-root):** root is lost after
  flashing. Patch the new boot.img with the APatch app and flash
  it to get root back.
- **DroidSpaces (v1.0-lto-droidspaces):** APatch pre-patched,
  root survives flashing. Keep the APatch manager app installed.

Revert: restore the Boot backup from recovery.
