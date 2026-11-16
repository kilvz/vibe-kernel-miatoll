# vibe-kernel-miatoll

LineageOS 22.2 kernel for the Redmi Note 9 Pro (joyeuse / miatoll),
with the FT8756 touch fix for the replacement screen, built with
ThinLTO on GitHub Actions.

## Why this exists

The phone has an aftermarket replacement screen with a FocalTech
FT8756 touch chip (Huaxing, SPI). On LineageOS the touch is dead
after every boot until the screen is toggled off/on: the first
display-on resume is a no-op in the FTS driver, so the chip never
gets a real reset after the panel powers on. The patch in this repo
makes the first display-on run the full resume/reset path, restores
the settle wait, and adds a read-path self-healing safety net.

## Files

- `ft8756-touch-fix.patch` — the source patch (self-contained,
  includes the firmware include files the driver needs to compile).
- `.github/workflows/build-kernel.yml` — the CI build (ThinLTO ON).
- `ci/anykernel.sh` — the AnyKernel3 flash script (kernel only).

## Build

Push to `main`, or run the workflow manually (Actions → Run
workflow). The workflow clones the LineageOS kernel (lineage-22.2,
pinned to `b539bd1`), applies the patch, builds with the stock
`miatoll_defconfig` (ThinLTO already on), and uploads the
AnyKernel3 zip as an artifact (`miatoll-ft8756-touchfix-lto`).

The ThinLTO final link needs ~16 GB RAM, which is why the build
runs on GitHub (a standard runner) instead of a small VM.

## Flash

Back up the Boot partition in recovery first. Flash the zip from
recovery (it replaces only the kernel Image), reboot, and touch
should work on the first boot with no screen toggle.

Flashing a new kernel removes the APatch kernel patch, so root is
lost until the new boot.img is re-patched with the APatch app and
re-flashed. See BUILD.md in the local build folder for the full
steps and the revert path.
