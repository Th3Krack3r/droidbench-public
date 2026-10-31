# DroidBench — Android Shop Toolkit

**Version 1.1.0** — by **Th3Krack3r** (Justin Westerheide)

> This repository contains the product documentation only. DroidBench source code is not public; DroidBench is sold as a commercial product with an email-bound license key.

Desktop tools for phone repair shops and power users: safe debloat with
one-click restore, app management, backups, fastboot + bootloader
tools, Samsung Download-mode flashing via Heimdall, a driver center,
and ticket-ready device reports. Pure Python — no pip installs, no
accounts, no cloud.

## Features

- **Device detection** — every connected phone with model, Android
  version and build fingerprint. Unauthorized/offline devices come with
  plain-English fix instructions ("unlock the phone and tap Allow…").
- **One-click debloat** — curated lists for Samsung, Google Pixel,
  Motorola and Xiaomi, plus a Universal list. Two modes: *Disable*
  (safe, reversible) and *Uninstall for user 0* (with a clear warning).
  **Every change is written to a session log and can be undone** with
  Restore — exactly what a shop needs when a customer changes their mind.
- **App manager** — list user/system apps, enable/disable, clear
  data+cache (labelled honestly — Android clears both), uninstall user
  apps, and **sideload APKs** (`apps install`, optional `-r` replace /
  `-g` grant permissions, always confirmed + logged). System apps are
  refused for full uninstall by design.
- **Backups** — pull app APKs (including splits), device info + package
  inventory snapshots in timestamped folders, app-data backup where the
  device still permits it (reported honestly when Android blocks it),
  and root-only partition imaging with by-name partition listing.
- **Fastboot tools** — device list, readable `getvar` summary (product,
  bootloader version, lock state, slot), reboot to system / recovery /
  bootloader / EDL. EDL is attempted only via the device's own
  `fastboot reboot edl` support and failures are reported, not faked.
- **Bootloader tab** — full status report (unlock state, bootloader
  version, product, secure state, A/B slots, `oem device-info` values,
  FRP hints where the device exposes them; Samsung Knox/warranty
  counters are honestly reported as *not readable over fastboot*), plus
  **guarded unlock/lock** — see Safety.
- **Samsung tab (Heimdall)** — Download-mode detection, visual PIT
  (partition table) reading, and guarded firmware flashing powered by
  the open-source **Heimdall** tool. Odin-style packages work too:
  open a `.tar`/`.tar.md5`, see its members auto-mapped to partitions
  (boot.img→BOOT, system.img→SYSTEM, …), adjust, dry-run, flash.
  **Odin itself is Samsung-proprietary and is NOT bundled** — Heimdall
  does the work. Heimdall is MIT-licensed open source.
- **Driver Center + setup bootstrapper** — `python -m droidbench setup`
  reports/installs everything DroidBench needs (platform-tools,
  Heimdall, scrcpy) into `./tools`, downloads only from official
  sources, and verifies SHA-256 pins. The Driver
  Center checks installed Windows USB drivers (Google, Samsung,
  Qualcomm 9008) and installs the official Google/Samsung packages
  with confirmation. The Qualcomm 9008 driver ships with QPST/QFIL,
  so instead of an unsigned mirror DroidBench prints the exact Device
  Manager steps.
- **EDL / BROM detection** — a Qualcomm device in EDL 9008 mode or a
  MediaTek device in BROM shows up in `devices` with an honest label
  (detection only; bring your own firehose/QPST/QFIL tooling).
- **Utilities** — screenshots, screen recordings, logcat dump + live
  streaming capture, wireless ADB pairing/connection (Android 11+),
  **file push/pull**, and **screen mirroring via scrcpy** (launch-only;
  downloaded on demand from its official release).
- **Debloat everywhere** — `--all-devices` (CLI) / the GUI checkbox
  runs the same logged debloat flow on every connected ready device.
- **Shop ticket reports** — clean text or printable HTML reports:
  device identity, battery health, storage, package counts and every
  action taken this session. Attach straight to the repair ticket.

## Safety by design

- **The destructive commands are few, named, and guarded.**
  Bootloader unlock/lock and Heimdall firmware flashing exist in 1.1 —
  behind hard gates, every time:
  - the device must be detected in the right mode first (fastboot
    device present, or Heimdall detect + a PIT read in the same
    session before any Samsung flash);
  - you must **type the confirmation word** (`UNLOCK`, `LOCK`, or
    `FLASH`) — and the `-y/--yes` flag pointedly does **not** skip it.
    The prompt itself states that unlock/lock wipe user data;
  - `--dry-run` prints the exact command without running it;
  - every attempt is written to the session log with the full command
    and its result, and after a flash DroidBench re-detects the device
    before reporting anything. Heimdall errors are shown raw.
- **Unlocking or locking a bootloader erases all user data** on most
  devices. DroidBench says so in the confirmation prompt itself, not
  in fine print.
- Every package change is logged (timestamp, serial, action, package)
  to `logs/session-*.jsonl` and is reversible from that log unless the
  log itself says otherwise (full uninstalls of user apps).
- Downloads (tools and drivers) come only from official/vendor sources,
  are checksum-verified where a pin exists, and nothing installs
  without an explicit confirmation that shows the exact file.
- Root features detect root first and skip with an explanation when it
  is absent. Nothing is ever reported as successful when it wasn't.

## Requirements

- Python 3.10 or newer (Windows, macOS or Linux) — or the packaged
  Windows installer, which needs no Python at all.
- Android SDK Platform Tools (`adb` / `fastboot`). Easiest path: run
  `python -m droidbench setup --install` once — it downloads the
  official platform-tools, verifies the checksum and puts them in
  `./tools` where DroidBench finds them automatically. Having them on
  your PATH or in a `platform-tools` folder next to DroidBench also
  works. Manual download:
  https://developer.android.com/tools/releases/platform-tools
- Phones need **USB debugging** enabled (Settings → Developer options),
  or wireless debugging for the Wi-Fi features.

## Install & run (Windows)

**Option A — installer:** run `DroidBench-Setup-1.1.0.exe`, then
launch DroidBench from the Start Menu. The installer may include the
tools already; if not, the app offers the one-click setup on first
run (Setup tab → "Install / repair all tools").

**Option B — from source:**

1. Install Python from https://www.python.org/downloads/ — tick
   **"Add python.exe to PATH"** during setup.
2. Unzip DroidBench to a folder, e.g. `C:\DroidBench`.
3. Open a terminal in the DroidBench folder and bootstrap the tools:

```
python -m droidbench setup --install  :: downloads adb/fastboot + scrcpy into .\tools
python -m droidbench devices          :: list phones
py -m droidbench.gui                  :: graphical interface
```

Or start the GUI by double-clicking a shortcut to
`pythonw -m droidbench.gui` run from the DroidBench folder.

## Command-line tour

```
python -m droidbench setup                          :: tool status (read-only)
python -m droidbench setup --install                :: bootstrap all tools into .\tools
python -m droidbench devices
python -m droidbench info
python -m droidbench debloat --list
python -m droidbench debloat --mode disable         :: asks before changing
python -m droidbench debloat --all-devices          :: every ready phone at once
python -m droidbench debloat --dry-run              :: preview only
python -m droidbench debloat --restore              :: undo the last session
python -m droidbench apps list --user
python -m droidbench apps disable com.example.app
python -m droidbench apps install app.apk --replace :: sideload (confirmed)
python -m droidbench backup snapshot --out backups
python -m droidbench backup apks com.example.app --out backups
python -m droidbench fastboot info --all
python -m droidbench fastboot status                :: bootloader report
python -m droidbench fastboot unlock                :: typed UNLOCK required; wipes data
python -m droidbench samsung detect                 :: Heimdall download-mode check
python -m droidbench samsung pit                    :: read the partition table
python -m droidbench samsung flash --tar AP.tar.md5 :: auto-mapped; --dry-run first
python -m droidbench drivers check                  :: Windows USB driver status
python -m droidbench utilities screenshot --out captures\front.png
python -m droidbench utilities push --file photo.jpg
python -m droidbench utilities mirror               :: scrcpy, if installed
python -m droidbench report --out ticket-1234.html
```

Mutating commands ask for confirmation unless you pass `-y/--yes` —
except unlock/lock/flash, which always require the typed word.

## Licensing

DroidBench runs in **trial mode** until activated: device detection,
device info, reports and the `setup` tool check work; debloat, app
management, backups, fastboot actions, Samsung flashing, driver
installs and utilities require a license.

Activate with the email and key from your purchase:

```
python -m droidbench license activate you@example.com DB1-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

…or use the **Activate license…** button in the GUI. Your key is tied to
your email; keep both together. Activation is stored in
`~/.droidbench/license.json` and works fully offline.

## Data files & logs

- `droidbench/data/*.json` — the debloat lists. Advanced users can add
  their own brand file in the same format.
- `logs/session-*.jsonl` — one JSONL log per work session, written in
  the folder you run DroidBench from. Keep these: they are your restore
  points and your paper trail for the ticket.

## Troubleshooting

- **Device shows UNAUTHORIZED** — unlock the phone and tap *Allow* on
  the USB debugging prompt; if no prompt appears, toggle USB debugging
  off/on in Developer options and reconnect.
- **Device shows OFFLINE** — try another cable/port; the cable is the
  problem more often than the phone.
- **"Could not find 'adb'"** — see Requirements above; the
  `platform-tools` folder must sit next to the `droidbench` folder or
  be on your PATH.
- **App-data backup fails** — modern Android versions and many apps
  block `adb backup`. APK backup and snapshots still work; DroidBench
  reports the block instead of pretending the backup exists.
- **"fastboot reboot edl" rejected** — many bootloaders simply do not
  implement it. A vendor EDL cable / test point or vendor tool is
  required for those models.
- **Samsung flash says "PIT required"** — read the PIT first (same
  session): the flash plan is only ever built against the partition
  table of the device actually attached.
- **Heimdall not found** — the Setup tab's installer explains how to
  stage it manually (Heimdall publishes no binary release assets, so
  DroidBench won't fetch it from a mirror)..
- **Setup refuses a download (SHA-256 mismatch)** — that is the pin
  doing its job; usually the vendor shipped a new release. Verify
  manually and re-pin per the pinned hashes.

## Author

DroidBench is built and sold by **Th3Krack3r** — **Justin Westerheide**
(West Electronics LLC). The branding is baked into the app: window
title, About dialog, reports and installer all carry it.
