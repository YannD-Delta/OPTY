# ⚡ OPTY — Windows Optimizer & Cleaner

> A single self-contained `.bat` that **cleans**, **configures** and **repairs** Windows 10/11.
> No install, no dependencies, pure CMD — and it explains every single thing it does before doing it.
>
> 🇫🇷 🇬🇧 **Fully bilingual.** The language is detected from your Windows profile and can be switched at any time.

---

## 🚀 Quick start

1. Download [`OPTY.bat`](https://github.com/YannD-Deltagon/OPTY/blob/master/OPTY.bat) — nothing else is needed
2. **Right-click → Run as administrator**
3. Follow the menu

⚡ **Fast full cleanup + reboot:** `1` `Enter`, then `3r` `Enter`

> An internet connection is recommended for the update check but not required.

---

## 🧭 What is actually in here

Two menus do the work. Everything else is reporting, repair or undo.

| Key | Menu | When you use it |
|-----|------|-----------------|
| `1` | **CLEAN** | Regularly. Deletes only what regenerates on its own. |
| `2` | **SETUP** | Once, on a new PC. Registry, services, power, network. |
| `3` | Reports | Network report, diagnose, crashes and freezes, open the log folder |
| `4` | Restore | Undo everything OPTY changed, re-assert Windows defaults |
| `5` | Repair Windows | Guided DISM, SFC, disk check |
| `6` | Maintenance | Driver store, prune OPTY's own files |
| `0` | Exit | |

---

## 📇 Every setting is explained before it is applied

This is the part that makes OPTY different from a list of `reg add` lines.

**225 settings** each carry a card, in French and English, that tells you:

- **what the setting is** — not the marketing name, the mechanism
- **what changing it actually does** — and what *each possible value* does, one by one
- **the gain** — and where there is none, the card says *"no measurable gain"* rather than inventing one
- **the cost** — what breaks, what slows down, what you lose permanently
- **the Windows default** — including when that default is *the value not existing at all*
- **what is not verified** — 132 of the 225 cards carry an explicit "unverified" note

Press `?` on any question for the long version: the reasoning behind the profile table, known bugs, and the exact registry path or command it will run.

### The 5 profiles

Every question in the whole script uses the same five answers, so one digit means the same intent everywhere:

| | Profile | Intent |
|---|---------|--------|
| `1` | **GAMING** | Maximum performance, minimum latency. Desktop, mains power. |
| `2` | **SERVER** | Torrent, Plex, game servers. Throughput over interactivity. |
| `3` | **OFFICE** | Quiet, low power draw, everything works. |
| `4` | **LAPTOP** | Battery first. On-battery caps are **respected**, never lifted. |
| `5` | **WINDOWS** | The shipped default — including deleting a value Windows ships absent. |

Press `P` in the SETUP menu to answer every question with one profile in about two minutes. Anything that can lose data still stops and asks.

> **Four identical columns is a normal answer.** Roughly half the cards recommend the same value for all four real profiles, because the setting genuinely does not vary by use case. Inventing a difference to fill the table would be worse than admitting there is none.

### Once you answer, the value is written

Even if it already looks correct. On a fresh or foreign machine the write is what makes it true. The log still distinguishes `SET`, `FIXED` and `written (was already correct)`, so you can tell a repair from a no-op.

---

## 🧹 CLEAN — three modes

| Mode | Behaviour |
|------|-----------|
| **1 — Manual** | Asks before each step and **explains it fully**, including whether the same step also runs unattended in the auto modes. |
| **2 — Auto lite** | Quick regular pass: Windows Update, then the file cleanup. |
| **3 — Auto full** | Everything: apps stopped, DNS, DISM, SFC, Windows Update, cleanup, WSL/Docker compaction, defrag. |

**Suffixes:** `3r` = Auto full + reboot · `3s` = + shutdown · same for `2r` / `2s`.

What runs in which mode is declared **once**, as data, and the release build fails if the code and that declaration ever disagree:

```
Auto lite            Windows Update, file cleanup
Auto full            the above + stop apps, DNS, DISM, SFC, WSL/Docker, defrag
Manual only          CHKDSK
```

### What gets deleted

The rule is **regeneration time, not file type**: anything that comes back on its own in well under 30 minutes and loses no user data.

Temp files, GPU and shader caches, browser caches (**never** cookies, history, saved logins or bookmarks), game-launcher caches, Windows Update cache, Delivery Optimization, crash dumps, logs, thumbnails, `Windows.old`, upgrade rollback folders, Recycle Bin. Crash dumps are the exception while **freeze capture** is on: then they are kept, because they are the evidence.

Development caches are **in scope** — npm, pip, Gradle, Cargo and friends refetch. A cache big enough that refetching is measured in hours gets its size reported before it goes.

---

## 🧊 Crashes and freezes

`3` Reports → `4` Crashes / freezes. Built from a real hunt for a PC that froze on the desktop and on wake, with nothing in the logs but *Kernel-Power 41, BugcheckCode 0*.

| Key | What it does |
|-----|--------------|
| `1` | **Freeze capture** — during a freeze, hold **Right Ctrl** and press **Space twice**: Windows stops on a deliberate blue screen (`0xE2`) and writes a kernel dump of the frozen state. Works on tenkeyless keyboards, which have no Scroll Lock for the documented combination. Active after one reboot. |
| `2` | **Back to Windows defaults** — removes the crash key, dump type back to automatic, memory test back to standard. |
| `3` | **Memory test at next reboot** — Windows Memory Diagnostic, extended mix, one pass, cache left at its default (forcing it off made a 32 GB run show no progress for a whole night). |
| `4` | **Crash report** — unexpected restarts with their decoded bugcheck, blue screens, dumps Windows *failed* to write, WHEA hardware errors, memory test results, dumps on disk. If `cdb.exe` (Debugging Tools for Windows) is installed, the newest dump is analysed automatically and the third-party drivers it loaded are listed, with a warning when several low-level hardware-access drivers (monitoring / RGB / overclocking tools) run at once. |

---

## 🛡️ What this tool refuses to do

Written down because most of these were mistakes it *used to* make:

- **Never deletes configuration, saved logins, cookies, history or entitlements.** The release build refuses to publish if any delete targets a folder known to hold them.
- **Never touches a cloud mount.** Google Drive reports itself to Windows as a fixed disk, byte for byte identically to `C:`. Deleting there propagates to every device syncing that account, so drives without a real local volume signature are skipped and the skip is logged with its reason.
- **Never claims a success it did not achieve.** A delete that could not happen, a registry write the driver ignored, a scheduled task that does not exist on this build — each is reported as what it was.
- **Never forces a preference.** A setting is either a *repair* (a shipped default something else broke, applied automatically) or a *preference* (real trade-offs, always asked).

---

## 🔧 Reliability

- **CRLF self-heal.** GitHub's release asset is served with LF line endings while the raw file is CRLF. CMD computes `call`/`goto :eof` return addresses as byte offsets assuming CRLF, so an LF copy drifts into the wrong section partway through a long run. OPTY detects this at startup and repairs itself before doing anything.
- **Runs in place from a git checkout.** Launched from a folder that holds `.git`, OPTY no longer relocates itself to `C:\OPTY_by-YannD` — the relocation deletes the original, and in a checkout the original is the working-tree `OPTY.bat`.
- **Automatic restore point** before any change.
- **Complete timestamped logs**, the 5 most recent kept.
- **Disk-space report** — free space before and after every run.
- **14 integrity gates** on the release build: CRLF, no stray control bytes, every `goto` and `call` resolves, no card missing a translation, no profile column that cannot be written, no card asked twice, no user-data deletion, no code reading a subroutine's output from inside its own parenthesised block.

Every one of those gates exists because the bug it catches was found in shipped code.

---

## ⚠️ Good to know

- **Closes apps** during cleanup: Docker Desktop, browsers, the Store window.
- **`Windows.old` and the upgrade staging folders are deleted** — you lose the ability to roll back a Windows upgrade.
- **`DISM /StartComponentCleanup /ResetBase`** — installed updates can no longer be uninstalled.
- **`CHKDSK /f /r`** may schedule a check on the next reboot.
- WSL/Docker compaction stops Docker and does **not** restart it.

Run **Mode 1 (Manual)** first if in doubt: it asks before each step and tells you exactly what that step does.

---

## 🖥️ Compatibility

Windows **10** / **11** (x64), administrator rights required.

---

## 🙏 Credits

Made by **[@YannD-Deltagon](https://github.com/YannD-Deltagon)**.
Use at your own risk — read the script before running it on a machine you care about.
