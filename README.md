# Samsung GSI Loader (SM-X110 / Himax Touchscreen Fork)

A userspace container-based Generic System Image (GSI) loader for Samsung devices (targeted at the SM-X110), forked to add full support for Himax touchscreens. Instead of flashing system partitions, it pivots into a raw GSI rootfs (`system.raw`) inside an isolated mount namespace while preserving One UI state for safe restoration.

## What's Changed in This Fork

* **Input Device Configuration (`.idc`) Injection:** Generates and stages `touch.deviceType = touchScreen` profiles for `himax-touchscreen`, `himax_touchscreen`, `himax-ts`, `himax_ts`, `sec_touchscreen`, and dynamically detected `/sys/class/input/input*/name` devices, bind-mounting them to `/vendor/usr/idc` and overlaying `/system/usr/idc` for read-only GSIs.


* **TDDI Panel Wake Cycling:** Forces a `0 -> 1` toggle on `/sys/class/input/input*/enabled` (and `/sys/class/sec/tsp/enabled` when present) after guest `SurfaceFlinger` startup and display wakeup (`KEYCODE_WAKEUP`) so the Himax controller exits sleep after host `surfaceflinger` stops.


* **Evdev Node Permissions:** Sets `/dev/input/event*` to `0666` so guest `EventHub` can read touch events during a session, and restores `root:input` (`0660`) when returning to One UI.



## Features

* **Non-Destructive Userspace Takeover:** Mounts an ext4 or EROFS `system.raw` image on a loop device and switches into it via `unshare -m` and `pivot_root` without modifying host partitions.


* **Multi-Slot Support:** Supports both a default installation (`/data/local/tmp/lineage_stage`) and isolated named slots under `slots/`.


* **State Snapshot & Clean Return (`return_to_oneui.sh`):** Snapshots `persist.*` properties via `resetprop`, backs up One UI Home launcher databases, `shared_prefs`, `appwidgets.xml`, and Quick Settings tiles, and restores SELinux enforcing mode and host services on exit.


* **Watchdog & Boot Recovery:** Keeps kernel watchdogs fed during the session and arms a 10-minute boot watchdog (`boot_watchdog.sh`) that automatically returns to One UI if the guest fails to reach the home screen.


* **Optional Compatibility Patches:** Supports framework, `PermissionController`, APEX list, and `gvld` service-isolation patches for LineageOS 23.2 GSIs while booting generic GSIs unpatched by default.



## Requirements

* Rooted Samsung device (Magisk or KernelSU) with working `resetprop` and `busybox` binaries.


* An uncompressed raw GSI image (`system.raw` in ext4 or EROFS format).


* USB debugging (`adb`) enabled for log inspection and recovery.



## Building from Source

1. Build the APK using `./build_apk.sh` on macOS or `./build_apk_linux.sh` on Linux.


2. Verify the package with `./verify_apk.sh` (or bundle release artifacts with `./make_dist.sh`).


3. Install the rebuilt APK and open the app before booting a GSI so the updated scripts in `assets/` are extracted to `/data/user/0/com.purple.gsiloader-x110/files/`.



## Documentation

* **`ARCHITECTURE.md`** — Container mount namespace layout, binder/APEX bridging, and takeover lifecycle.


* **`PATCHES.md`** — Details on optional framework, `PermissionController`, and `gvld` isolation patches.


* **`TROUBLESHOOTING.md`** — Boot failure diagnostics, watchdog recovery, and log inspection.


* **`CREDITS` & `LICENSE**` — Upstream attributions and licensing terms.
