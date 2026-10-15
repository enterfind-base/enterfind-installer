# Enterfind installer

> Enterfind and its installer are in beta. Expect changes between releases, and please report problems you run into at https://github.com/enterfind-base/enterfind-server/issues.

One program that installs [Enterfind](https://enterfind.com) on your computer and keeps it
running: the Enterfind server, the Java runtime it needs, a PostgreSQL database, and Tesseract
for reading text in images and scanned documents. Everything goes into one folder in your home
directory. It needs no administrator rights and no package manager, and installs nothing
system-wide.

- [Requirements](#requirements)
- [Install](#install)
- [First use](#first-use)
- [Running in the background](#running-in-the-background)
- [Using the installer later](#using-the-installer-later)
- [Commands](#commands)
- [Updates](#updates)
- [Settings](#settings)
- [Logs and diagnostics](#logs-and-diagnostics)
- [Uninstall](#uninstall)
- [Troubleshooting](#troubleshooting)
- [WSL2](#wsl2)
- [What is installed where](#what-is-installed-where)
- [License](#license)

## Requirements

| System | Version | Download |
|---|---|---|
| Windows 11 | x64 (not Arm) | `enterfind-x86_64-pc-windows-msvc.exe` |
| macOS, Apple silicon (M1 and later) | tested on macOS 15 and 26 | `enterfind-aarch64-apple-darwin` |
| macOS, Intel | tested on macOS 15 | `enterfind-x86_64-apple-darwin` |
| Linux x64 | glibc 2.28 or newer (Debian 10, Ubuntu 18.10, RHEL 8, Fedora 29 and later) | `enterfind-x86_64-unknown-linux-gnu` |
| Linux arm64 | glibc 2.28 or newer | `enterfind-aarch64-unknown-linux-gnu` |
| WSL2 on Windows | a glibc distribution such as Ubuntu | the Linux x64 file (see [WSL2](#wsl2)) |

Not supported: Windows on Arm, 32-bit systems, Alpine and other musl-based Linux, WSL1.

Also needed:
- **Disk**: 3 GB free to install. The programs take about 1 GB; the index grows with your
  documents.
- **Memory**: 4 GB or more is comfortable. The installer sizes Java and PostgreSQL to the
  computer's memory (about 40% of it together).
- **Network**: about 700 MB is downloaded from GitHub, Adoptium (Java) and Maven Central
  (PostgreSQL) during the install, and less for updates.
- **Linux**: `tar`, and for starting at boot either systemd (as a user service) or a cron
  daemon. Most distributions have all of them.

## Install

### With a terminal (recommended)

Open a terminal, paste the line for your computer and press Enter. It downloads the installer
into the current folder and runs it. The installer asks a few questions, downloads the rest and
starts Enterfind; it then shows the address to open in your browser.

**macOS, Apple silicon** (in Terminal)

```sh
curl -fsSLo enterfind https://github.com/enterfind-base/enterfind-installer/releases/latest/download/enterfind-aarch64-apple-darwin && chmod +x enterfind && ./enterfind
```

**macOS, Intel**

```sh
curl -fsSLo enterfind https://github.com/enterfind-base/enterfind-installer/releases/latest/download/enterfind-x86_64-apple-darwin && chmod +x enterfind && ./enterfind
```

**Windows 11** (in PowerShell: Start menu, type "PowerShell")

```powershell
irm https://github.com/enterfind-base/enterfind-installer/releases/latest/download/enterfind-x86_64-pc-windows-msvc.exe -OutFile enterfind.exe; .\enterfind.exe
```

**Linux and WSL2, x64**

```sh
curl -fsSLo enterfind https://github.com/enterfind-base/enterfind-installer/releases/latest/download/enterfind-x86_64-unknown-linux-gnu && chmod +x enterfind && ./enterfind
```

**Linux, arm64**

```sh
curl -fsSLo enterfind https://github.com/enterfind-base/enterfind-installer/releases/latest/download/enterfind-aarch64-unknown-linux-gnu && chmod +x enterfind && ./enterfind
```

Without questions, with the defaults: run `./enterfind install` (Windows: `.\enterfind.exe
install`) instead of `./enterfind` at the end.

### With a browser

The installer is not signed yet, so macOS and Windows warn about a file downloaded with a
browser. A download in a terminal, as above, is not affected. Go to the [latest
release](https://github.com/enterfind-base/enterfind-installer/releases/latest), then:

- **macOS**: download `enterfind-aarch64-apple-darwin.zip` (Intel:
  `enterfind-x86_64-apple-darwin.zip`). Safari unpacks it by itself (unless "Open safe files
  after downloading" is off in its settings); otherwise double-click the `.zip`.
  Double-click the unpacked `enterfind-…-apple-darwin`. macOS refuses it the first time
  ("cannot be opened because Apple cannot check it for malicious software"): click **Done**,
  open **System Settings › Privacy & Security**, scroll down to the message about
  `enterfind-…`, click **Open Anyway**, confirm with your password or Touch ID, and
  double-click the file again. It opens in Terminal.
- **Windows**: download `enterfind-x86_64-pc-windows-msvc.exe` and double-click it. If
  SmartScreen says "Windows protected your PC", click **More info**, then **Run anyway**. Your
  browser may also ask whether to keep the file: keep it.
- **Linux**: download the file for your system, then in a terminal `chmod +x` it and run it.

Every file in a release has a `.sha256` next to it, to check the download
(`shasum -a 256 <file>` on macOS, `sha256sum` on Linux, `Get-FileHash` in PowerShell).

### What the guided install asks

It shows the settings it will use and offers to change them before anything is downloaded:

- **Port**: 8080, or the next free one.
- **Reach**: this computer only (the default), or all networks. Enterfind starts with
  authentication off (you can turn it on and add users in Enterfind itself) and has no HTTPS of
  its own: on all networks, until authentication is on, anyone who can reach the computer can
  search your documents. To use Enterfind from elsewhere, prefer its built-in remote access (an
  invitation code, set up in Enterfind's settings) to opening it to the network.
- **Java memory** and **PostgreSQL tuning**: computed from the computer's memory and processors.
- **Autostart**: start Enterfind with the computer and keep it running (yes by default; see
  [Running in the background](#running-in-the-background)).

Then it shows the plan and the download size and asks once more. A download that drops is
retried; an install that fails stops what it started and can simply be run again.

## First use

When the install is done, the installer shows the address, normally
**http://127.0.0.1:8080**, and offers to open it. Bookmark it; `enterfind status` shows it again.
(`http://localhost:8080` works too, except from Windows into WSL; see [WSL2](#wsl2).)

In Enterfind, add a **data source**: a folder on this computer, or a remote file system. Enterfind
then scans it and indexes the documents; the first scan of a large folder takes a while, and
search results grow as it goes. Text in images and scanned PDFs is read by Tesseract, with all
of its languages installed.

The installer itself is only needed to install, update, change settings or uninstall. You can
keep the downloaded file or delete it: the installer keeps a copy of itself in the installation
(`~/.enterfind/bin/`).

## Running in the background

Enterfind runs as two background programs, the Enterfind server (Java) and PostgreSQL. They keep
running when you close the terminal or the installer's window.

With autostart on (the default), Enterfind starts again by itself after a restart, and a check
every 5 minutes starts whatever has stopped or crashed:

| System | Starts | How |
|---|---|---|
| Windows | when you log in | a scheduled task `enterfind-ensure` in your user account; no window appears |
| macOS | when you log in | a LaunchAgent `com.enterfind.ensure` (`~/Library/LaunchAgents/`) |
| Linux with systemd | at boot (if "lingering" is allowed, which the installer turns on), else when you log in, and it stops when you log out | systemd user units `enterfind-ensure.service` and `.timer` |
| Linux without systemd | at boot | two lines in your crontab marked `# enterfind-managed` (needs a running cron daemon) |
| WSL2 | when you log in to Windows | the Windows scheduled task, which starts the distribution |

On macOS the servers keep running when you log out, and start at the next login after a
restart. On Windows they stop when you log out and start at the next login.

**`stop` means stopped**: after `enterfind stop` (or **Stop** in the menu), the autostart check
leaves Enterfind off, also across restarts, until you run `enterfind start`.

Turn autostart off with `enterfind config --no-autostart`, on again with `enterfind config
--autostart`. Without autostart, Enterfind runs until the computer restarts; then start it with
`enterfind start`.

## Using the installer later

Run the installer again with no arguments (or double-click it) for the guided mode. It checks
for updates, repairs what is not running, and shows a menu:

- running: **Open http://127.0.0.1:8080** (its address), **Stop**, **Restart**, **Settings**, **Logs**, **Uninstall**,
  **Exit**
- stopped: **Start**, **Settings**, **Logs**, **Uninstall**, **Exit**

Any copy works: the one you downloaded, a newer download, or `~/.enterfind/bin/enterfind`
(Windows: `%USERPROFILE%\.enterfind\bin\enterfind.exe`). An older download hands over to the
newer installer in the installation, so it never downgrades anything.

Only one installer works on an installation at a time. If another is busy (another window, or
the autostart check), you see "another enterfind is running": wait a moment and try again.

## Commands

For scripts and for anyone who prefers commands, every action is a command. `enterfind --help`
lists them, `enterfind <command> --help` explains one. Commands never ask questions: whatever
needs a confirmation takes `--yes`. In the examples, `enterfind` stands for your copy
(`./enterfind`, `.\enterfind.exe`, or `~/.enterfind/bin/enterfind`).

| Command | What it does |
|---|---|
| `install` | install with the defaults and start; options `--port <port>`, `--bind local\|all`, `--heap <size>` (e.g. `2g`), `--pg-tune auto\|KEY=VALUE` (repeatable), `--no-autostart` |
| `status` | versions, processes, health, the address and known updates; `--check-updates` checks first |
| `start` | start Enterfind and keep it running (ends a `stop`) |
| `stop` | stop Enterfind; autostart leaves it off until `start` |
| `restart` | stop and start |
| `update` | list (`--check`) or install (`--yes`) updates; `--only app,jre,postgres,tesseract,installer` |
| `config` | no options: print the settings; with `--port`, `--bind`, `--heap`, `--pg-tune`, `--autostart`, `--no-autostart`: change them and restart what they affect |
| `logs [app\|postgres\|installer]` | the last lines of a log (`--lines <n>`, default 50; `--follow` keeps printing) |
| `doctor` | diagnostics: disk, system libraries, the installed files, OCR, ports, autostart |
| `uninstall` | `--keep-data` or `--delete-data`, with `--yes` |
| `ensure` | bring Enterfind to the state it should be in (what autostart runs) |
| `interactive` | the guided mode (needs a terminal) |
| `version` | the installer's version |

`install`, `start`, `stop`, `restart`, `ensure`, `update`, `config` and `uninstall` take
`--dry-run`: print what they would do, and stop.

Options for every command: `--json` (JSON lines, `"schema":1`, for programs), `--quiet` (only
errors), `--verbose` (error details), `--no-color` (also the `NO_COLOR` environment variable).

Exit codes:

| Code | Meaning |
|---|---|
| 0 | success; `status`: running and healthy |
| 1 | error (also wrong options, and `doctor` with a failed check) |
| 2 | a confirmation or decision is needed (add `--yes`, or the missing option) |
| 3 | another enterfind is busy with this installation |
| 4 | `status` only: installed, but not healthy or stopped |
| 5 | `status` only: not installed (also an unfinished install, or data kept after uninstall) |

## Updates

The guided mode checks for updates every time it starts. With commands:

```sh
enterfind update --check     # list what is available
enterfind update --yes       # install it
```

- Updates stay **within the installed major version** of each part (server 1.x stays 1.x,
  PostgreSQL 18 stays 18), so they never need a data migration of their own. A newer major of the
  server is mentioned, not installed.
- The installer updates itself first, then the server, Java, PostgreSQL and Tesseract.
- New versions are downloaded and checked while Enterfind keeps running; then Enterfind restarts
  on them, which takes a minute or two. One previous version of each part stays on disk.
- If the new PostgreSQL does not start, the old one is used again. If the server does not come
  up after an update of Java, PostgreSQL or Tesseract alone, those are switched back. If it
  does not come up after a server update, nothing is switched back (the new server may already
  have changed the database): see `enterfind logs app`.
- Free disk space needed: twice the downloads plus 1 GB.
- `--only` limits an update, e.g. `enterfind update --yes --only app` for the server alone.

## Settings

`enterfind config` prints the settings; the options change them, one or several at a time,
and restart only what is affected (nothing, if you stopped Enterfind):

```sh
enterfind config --port 8090          # another port (it must be free)
enterfind config --bind all           # reachable from other computers (see the warning above)
enterfind config --bind local         # this computer only
enterfind config --heap 4g            # Java memory
enterfind config --pg-tune work_mem=64MB   # a PostgreSQL setting; --pg-tune auto: the computed ones
enterfind config --no-autostart       # see "Running in the background"
```

In the guided mode, **Settings** changes one value at a time. The settings live in
`~/.enterfind/config/enterfind.toml`; you may edit it and then run `enterfind restart`.

## Logs and diagnostics

```sh
enterfind status             # is it running, which versions, which address
enterfind logs               # the server's log (app); also: postgres, installer
enterfind logs -f            # keep printing new lines (Ctrl+C to end)
enterfind doctor             # checks with a pass/warn/fail each
```

The log files are in `~/.enterfind/logs/` (`installer.log` is the installer's own, including
what autostart did) and `~/.enterfind/data/postgres/log/`. Report problems with Enterfind or
the installer in the [Enterfind server's issues](https://github.com/enterfind-base/enterfind-server/issues),
and include the output of `enterfind doctor` and `enterfind status`, and the end of the log it
points to.

## Uninstall

In the guided mode: **Uninstall**. With a command:

```sh
enterfind uninstall --keep-data --yes     # remove the programs, keep the index and settings
enterfind uninstall --delete-data --yes   # remove everything
```

Both stop Enterfind and remove the autostart entries first.

- `--keep-data` removes the programs and logs and keeps `data/` (the database), `models/` (the
  server's AI models) and `config/` (settings, and the key that encrypts credentials Enterfind stores, e.g. of a
  remote data source).
  A later install picks them up again.
- `--delete-data` removes the whole `~/.enterfind` folder. Your documents themselves are never
  touched; only Enterfind's index of them goes. The guided mode shows the size and asks twice.

On Linux, uninstall leaves systemd "lingering" for your user on (other services may use it):
`loginctl disable-linger` turns it off. Your downloaded installer file stays where you put it.

## Troubleshooting

**The address does not open.** `enterfind status` says whether Enterfind runs, and on which
address. After a start, the server needs up to a few minutes before it answers (status:
"starting"). If it stays down, `enterfind logs app` shows why; `enterfind restart` tries again.

**Port already in use.** At install, a taken 8080 makes the installer use the next free port.
If another program later takes Enterfind's port, starting fails with "port … is in use by
another program": stop that program, or move Enterfind with `enterfind config --port <port>`.
PostgreSQL's and the server's internal ports move by themselves.

**Not enough disk space.** The install needs 3 GB free, an update twice its downloads plus 1 GB.
Free some space and run it again; nothing is lost.

**It does not start after a restart.** Check that autostart is on (`enterfind doctor`, line
"autostart") and that you did not `stop` it (`enterfind status` says "stopped (by `enterfind stop`…)";
`enterfind start` ends that). `enterfind config --autostart` registers it again. The autostart check
writes to `~/.enterfind/logs/installer.log`.

**macOS: "cannot be opened" / "Apple cannot check it".** The browser download is blocked once:
use **Open Anyway** as described in [With a browser](#with-a-browser), or install with the
terminal line instead.

**Windows: "Windows protected your PC".** SmartScreen: **More info**, then **Run anyway**. Or
install with the PowerShell line.

**Linux without systemd or cron.** The installer then cannot start Enterfind at boot: install
says so and `doctor` warns. Install `cron` (or `cronie`) and start its service, then run
`enterfind config --autostart`. Until then, run `enterfind start` after each boot.

**Linux with systemd: stops at logout.** Then lingering is off for your user and could not be
turned on. Ask an administrator for `loginctl enable-linger <user>`, then run `enterfind config
--autostart`.

**"another enterfind is running".** Another installer or the autostart check is working on the
installation. Wait a minute and try again.

**"this installation was made by enterfind N.x".** Installers of different major versions do
not mix: use the installer of that major version (in `~/.enterfind/bin/`), or uninstall with it
first.

**Damaged files.** `enterfind doctor` checks that each installed part has its programs (line
"integrity"). If one is damaged, reinstall the programs and keep your data:
`enterfind uninstall --keep-data --yes`, then `enterfind install`.

## WSL2

Install inside your Linux distribution with the Linux x64 line, as on Linux. What differs:

- Enterfind starts when you log in to Windows, through a Windows scheduled task that starts the
  distribution in the background; no window appears, and no WSL terminal needs to be open.
  Nothing is registered inside the distribution.
- Open **http://127.0.0.1:8080** in Windows (the installer shows and opens this address). Avoid
  `localhost` from Windows: it may try IPv6 first, which does not reach the server.
- Data sources in Windows folders work (`/mnt/c/…`), but reading them through WSL is slower than
  a folder inside the distribution.
- WSL gives the distribution half of the computer's memory by default; the installer sizes
  Enterfind to what the distribution has.

## What is installed where

Everything is in `~/.enterfind` (Windows: `%USERPROFILE%\.enterfind`):

| Folder | Contents |
|---|---|
| `bin/` | the installer's own copy; autostart runs it |
| `app/` | the Enterfind server |
| `runtime/` | Java (Eclipse Temurin), PostgreSQL, Tesseract with its language models; on Windows, Java and PostgreSQL also get Microsoft's Visual C++ runtime in their `bin` folders, so they do not depend on the Visual C++ Redistributable being installed |
| `data/` | the database |
| `models/` | the server's AI models, downloaded by the server |
| `config/` | `enterfind.toml` (settings), `secrets.json` (generated passwords and key; private) |
| `logs/` | the installer's and the server's logs |
| `state.json`, `run/`, `downloads/` | the installer's bookkeeping and temporary files |

Outside it, only the autostart entry (see [Running in the background](#running-in-the-background)).
PostgreSQL listens on `127.0.0.1` only, with generated passwords.

## License

The installer (`enterfind`) is MIT-licensed, Copyright (c) 2026 Mikhail Garber: see
[LICENSE](LICENSE). It is built with Deno; [THIRD_PARTY_NOTICES](THIRD_PARTY_NOTICES) lists the
software it contains and their licenses.

**The Enterfind server is not covered by this license.** The installer downloads it from its
own [releases](https://github.com/enterfind-base/enterfind-server/releases), and it is licensed
under the [GNU Affero General Public License v3.0](https://www.gnu.org/licenses/agpl-3.0.html)
(AGPL-3.0). The other components the installer downloads (Java, PostgreSQL, Tesseract and its
models) come under their own licenses, which are in their folders under `runtime/`. On Windows,
the Microsoft Visual C++ runtime (taken from conda-forge's `vc14_runtime` package) comes under
Microsoft's [Visual C++ Runtime license terms](https://aka.ms/VCRedistLicense).
