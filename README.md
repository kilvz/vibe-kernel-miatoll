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

## Variants — four builds, one patch

All four are built from the same touch-fix patch (v6) and the same
ThinLTO build (kernel 4.14.336). Pick one:

| Variant | Branch | Release | What you get |
|---|---|---|---|
| **Clean** | `main` | [v1.0-lto](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto) | Touch fix only (clang r530567). No root. Flashing removes the APatch patch, so root is lost until the new boot.img is re-patched with the APatch app. |
| **Root** | `root` | [v1.0-lto-root](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-root) | Touch fix only, built with clang r536225 (same as stock LineageOS) and CFI off. NOT pre-patched with APatch — patch the released boot.img yourself with the APatch app. |
| **DroidSpaces** | `droidspaces` | [v1.0-lto-droidspaces](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-droidspaces) | Touch fix + full DroidSpaces container support (namespaces, cgroups, seccomp, NAT, overlayfs, user namespaces + the non-GKI cgroup patch) + APatch pre-patched (kpimg-android 0.13.9). Root survives flashing; no re-patch needed. Two guide options could not be enabled: CFS_BANDWIDTH (blocked by Qualcomm's SCHED_WALT) and CGROUP_NS (does not exist in this 4.14 tree). DroidSpaces + APatch confirmed working on-device. |
| **NetHunter** | `nethunter` | [v1.0-lto-nethunter](https://github.com/kilvz/vibe-kernel-miatoll/releases/tag/v1.0-lto-nethunter) | Touch fix + DroidSpaces + Kali NetHunter support: HID gadget, RNDIS, Wi-Fi monitor mode + packet injection via USB adapters (RTL8812AU/8814AU, rtw88, ath9k_htc, rt2800usb, rtl8187), Bluetooth Arsenal, CAN, RTL-SDR, USB serial. NOT pre-patched with APatch — patch it yourself. Internal-WiFi (qcacld-3.0) injection was removed: that patch broke boot. |

## Changelog

- **Touch fix v6 (current)** — delayed work ~20 s after probe:
  reset GPIO pulse (low 20 ms, high, wait 50 ms), then
  suspend→resume. Replaces v5 (register writes only, did not
  revive the IC) and v4 (stopping the firmware re-flash broke
  screen off/on recovery). Firmware re-flash stays ON.
- **v1.0-lto** — the base build: FT8756 touch fix, ThinLTO ON,
  stock `vendor/xiaomi/miatoll_defconfig`, clang r530567.
- **v1.0-lto-root** — same touch fix, clang r536225 (stock
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
- **v1.0-lto-nethunter** — everything in v1.0-lto-droidspaces,
  plus Kali NetHunter support from the official kernel-builder
  4.14 patch set: mac80211 packet injection, RTL8812AU/8814AU
  and rtw88 USB Wi-Fi drivers, ath9k naming fix, UB500 Bluetooth
  device id, and the `ci/nethunter.config` options (HID core,
  RNDIS gadget, mac80211, USB Wi-Fi drivers, Bluetooth
  RFCOMM/HIDP/BNEP, CAN, RTL-SDR, USB serial). The qcacld-3.0
  frame-injection patch is NOT included: it broke boot (the
  phone's built-in Wi-Fi driver). Internal-WiFi injection is
  unavailable; use a USB adapter. The rtl8188eus patch is
  skipped: this tree's rtl8xxxu is newer and already has
  RTL8188E support. All drivers built in (`=y`), so the zip
  stays kernel-only. Unpatched: patch the released boot.img
  with the APatch app.

## Files

- `ft8756-touch-fix.patch` — the source patch (self-contained).
- `.github/workflows/build-kernel.yml` — the CI build (ThinLTO ON).
  Each branch builds its own variant; the workflow triggers on push
  to its branch and on manual dispatch.
- `ci/anykernel.sh` — the AnyKernel3 flash script (kernel only).
- `ci/droidspaces.config` — DroidSpaces config fragment (droidspaces
  branch only).
- `ci/droidspaces-02-cgroup-prefix.patch` — DroidSpaces non-GKI
  cgroup patch (droidspaces branch only).
- `ci/nethunter-*.patch` + `ci/nethunter.config` — Kali NetHunter
  4.14 patches and config fragment (nethunter branch only).
- `ci/apatch-compat.config` — CFI off (root, droidspaces and
  nethunter branches only).

## Build

Push to a branch, or run the workflow manually (Actions → Run
workflow, pick the branch). The workflow clones the LineageOS kernel
(lineage-22.2 branch tip), applies the touch-fix patch (plus the
DroidSpaces patch and config on the droidspaces branch, plus the
NetHunter patches and config on the nethunter branch, plus the
APatch pre-patch on the droidspaces branch only), builds with the
stock `miatoll_defconfig` (ThinLTO already on), and uploads the
AnyKernel3 zip as an artifact.

The ThinLTO final link needs ~16 GB RAM, which is why the build
runs on GitHub (a standard runner) instead of a small VM.

## Flash

Back up the Boot partition in recovery first. Flash the zip from
recovery (it replaces only the kernel Image), reboot, and wait
~30 seconds after boot before judging touch (the revive runs at
~20 s). Touch should work with no screen toggle.

- **Clean (v1.0-lto) and Root (v1.0-lto-root):** root is lost after
  flashing. Patch the new boot.img with the APatch app and flash
  it to get root back.
- **DroidSpaces (v1.0-lto-droidspaces):** APatch pre-patched,
  root survives flashing. Keep the APatch manager app installed.
- **NetHunter (v1.0-lto-nethunter):** root is lost after flashing.
  Patch the new boot.img with the APatch app.

Revert: restore the Boot backup from recovery.
