<a href="https://soroushse7o.github.io/termux-to-ish/">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=150&section=header&text=termux-to-ish&fontSize=30&animation=fadeIn" width="100%">
</a>

مبدل دستور ترموکس به iSH

**فارسی** · [English](README.en.md)

<a href="https://soroushse7o.github.io/termux-to-ish/">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=22&pause=1000&color=22C55E&center=true&vCenter=true&width=780&lines=Convert+Termux+commands;to+iSH+seamlessly" alt="Typing SVG">
</a>
A single-file HTML page (Persian by default, with an English toggle) that converts **Termux** commands (Android) into commands compatible with **iSH** (a Linux shell on iPhone). Everything runs in the browser; nothing is sent anywhere.

---

## 1. What iSH is and why commands differ

Based on the official iSH source (`README.md`, `app/`, `kernel/`):

- iSH runs a Linux shell on iOS using **user-mode x86 emulation** and **Linux syscall translation**. The emulated CPU is **32-bit (i386/i686)**, not arm64 or amd64.
- The default filesystem is **Alpine Linux**. The apk repositories are set to **Alpine v3.19**, architecture **x86** (`app/gen_apk_repositories.py`, `app/CurrentRoot.h`), served as a frozen snapshot from the `apk.ish.app` mirror.
- The default launch command is `/bin/login -f root` (`app/UserPreferences.m`), so you are **root** and `sudo` is unnecessary. `TERM` is `xterm-256color`.
- Alpine's default shell is `ash` (busybox), not `bash`.

Termux runs on Android/ARM with the `pkg` package manager (apt-based) and `/data/data/com.termux/files/...` paths, so many commands will not run in iSH unchanged.

## 2. What the tool does

### 2.1 Package manager conversion

| Termux | iSH |
|---|---|
| `pkg install` / `apt install` / `apt-get install` | `apk add` |
| `pkg remove` / `uninstall` / `purge` | `apk del` |
| `pkg update` | `apk update` |
| `pkg upgrade` / `full-upgrade` | `apk upgrade` |
| `pkg search` | `apk search` |
| `pkg list-installed` | `apk info` |
| `pkg show` | `apk info -a` |
| `pkg files` | `apk info -L` |
| `pkg reinstall` | `apk fix --reinstall` |
| `pkg clean` | `apk cache clean` |
| `pkg autoremove` | dropped (apk does not need it) |

Flags such as `-y`, `--yes` and `--no-install-recommends` are dropped because `apk` never asks for confirmation.

### 2.2 Package name mapping

| Termux | Alpine |
|---|---|
| `python` | `python3` |
| `python-pip` | `py3-pip` |
| `nodejs` / `nodejs-lts` | `nodejs npm` |
| `golang` | `go` |
| `rust` | `rust cargo` |
| `build-essential` | `build-base` |
| `openssl-tool` | `openssl` |
| `libffi`, `libxml2`, `zlib` … | `libffi-dev`, `libxml2-dev`, `zlib-dev` … |
| `dnsutils` | `bind-tools` |
| `netcat` | `netcat-openbsd` |
| `pkg-config` | `pkgconf` |
| `python-numpy` | `py3-numpy` |

The full list is the `PKG` object in the code. **Packages not in the table keep their name** and the tool warns you to check them with `apk search name`.

Termux-only packages (`termux-api`, `termux-tools`, `root-repo`, `x11-repo`, `proot-distro`, `tsu`, …) are removed with a warning (`DROP` set).

### 2.3 `bash <(curl …)` conversion

Process substitution (`<( )`) does not work in `ash`, so the pattern becomes "download the file, then run it":

```sh
# input
bash <(curl -fsSL https://example.com/install.sh)
# output
curl -fsSL https://example.com/install.sh -o install.sh && chmod +x install.sh && bash install.sh
```

- The interpreter is preserved: `bash`, `sh` or `zsh` (a warning reminds you to install zsh).
- `source <(curl …)` and `. <(curl …)` load the file with `. ./file` in the current shell so functions and variables are not lost.
- Trailing arguments (e.g. `--install`) and `-H`/`-A` headers are preserved.
- URLs containing special characters (`&`, `?` …) are quoted automatically.
- `curl … | bash` needs no conversion, but `bash` must be installed (see 2.7).

### 2.4 Paths and variables

| Termux | iSH |
|---|---|
| `/data/data/com.termux/files/usr` and `$PREFIX` | `/usr` |
| `/data/data/com.termux/files/home` | `/root` |
| `$TMPDIR` | `/tmp` |
| `~/storage/shared`, `/sdcard`, `/storage/emulated/0` | `/mnt/files` (after mounting) |

### 2.5 `termux-*` commands that have equivalents

The iSH source creates two special devices (`app/AppDelegate.m`, `fs/devices.h`):

| Termux | iSH | Notes |
|---|---|---|
| `termux-clipboard-get` | `cat /dev/clipboard` | reads the iOS clipboard |
| `termux-clipboard-set text` | `printf %s text > /dev/clipboard` | writes the clipboard |
| `… \| termux-clipboard-set` | `… > /dev/clipboard` | |
| `termux-location` | `cat /dev/location` | output is `+lat,+lon` (not JSON); needs location permission |
| `termux-wake-lock` | `cat /dev/location > /dev/null &` | known trick to keep the app alive in the background |
| `termux-wake-unlock` | `pkill -f 'cat /dev/location'` | |
| `termux-setup-storage` | `mkdir -p /mnt/files && mount -t ios . /mnt/files` | access iPhone files |
| `termux-info` | `cat /proc/ish/version` | |

Notes from the code:

- Clipboard: if the clipboard changes while you are reading, the read fails with an error; the maximum buffer is 8 MB (`app/PasteboardDevice.m`).
- Location: the source enables `allowsBackgroundLocationUpdates`, which is why reading `/dev/location` keeps the app awake, at the cost of battery.

`termux-fix-shebang` and `termux-reload-settings` are dropped because they are not needed in iSH.

### 2.6 Commands that cannot be converted (warning only)

These are left unchanged with a warning: the other `termux-*` commands (`termux-notification`, `termux-vibrate`, `termux-battery-status`, `termux-open-url`, `termux-sms-send`, …), plus `proot`, `proot-distro`, `tsu`, `sv`, `am`, `pm`, `getprop`, `dumpsys`, `logcat` and `settings`. They are Android APIs or tools with no iSH equivalent.

### 2.7 Options

- **Prepend prerequisites:** puts `apk update && apk add curl bash …` first. Other tools (`git`, `python3`, `py3-pip`, `nodejs`, `openssl`, `jq`) are added only if the command really uses them and does not already install them.
- **`--break-system-packages` for pip:** Alpine 3.19 blocks system-wide pip installs (`externally-managed-environment`). This option adds the flag to `pip install`. Safer alternatives: a venv, or `apk add py3-name`.

### 2.8 Architecture warnings

If the command mentions `arm64`, `aarch64`, `armv7`, `amd64` or `x86_64`, the tool warns that iSH only runs 32-bit x86 binaries (`386`/`i386`/`i686`).

### 2.9 Handling complex commands

- Chains (`&&`, `||`, `;`) are converted piece by piece; operators inside quotes, parentheses and comments are left alone.
- Lines continued with `\` are joined.
- Heredoc bodies (`<<EOF … EOF`) are **not** converted, only Termux paths and variables inside them are fixed.
- Comment and blank lines are left untouched.

## 3. Usage

1. Open `index.html` in a browser (online or offline; only the Vazirmatn font loads from Google, with a fallback font when offline).
2. Use the language button (top corner) to switch between Persian (default) and English; your choice is remembered in the browser.
3. Paste the Termux command; the output updates live.
4. Read the warnings (amber); anything that could not be converted is explained there.
5. Press "Copy command" and paste it into iSH.
6. "Example" cycles through several sample commands.

### Examples

```sh
# input
pkg update && pkg upgrade -y && pkg install -y python git openssh nodejs-lts
# output
apk update && apk upgrade && apk add python3 git openssh nodejs npm
```

```sh
# input
termux-setup-storage && cp ~/storage/shared/Download/a.txt ~/ && echo "ok" | termux-clipboard-set
# output
mkdir -p /mnt/files && mount -t ios . /mnt/files && cp /mnt/files/Download/a.txt ~/ && echo "ok" > /dev/clipboard
```

## 4. Accessing iPhone files in iSH

`mkdir -p /mnt/files && mount -t ios . /mnt/files` opens an iOS folder picker and shows the chosen folder at `/mnt/files`. The source (`app/iOSFS.m`) registers two filesystem types, `ios` and `ios-unsafe`; the latter uses raw filesystem operations instead of iSH's safer wrappers, so prefer plain `ios` unless you have a specific need.

## 5. Limitations and caveats

- **Pinned repositories:** iSH manages `/etc/apk/repositories` itself and, if the `/ish` directory exists, rewrites it on launch (the comment inside the file says so). Manual repository edits do not persist. Packages are a snapshot, so `apk upgrade` usually brings little new.
- **Emulation is slow:** heavy workloads (compiling, Node.js, Rust, Go) may be slow or problematic; this tool makes no guarantees about them.
- **32-bit binaries:** any downloaded binary must be built for 32-bit x86.
- **Incomplete syscalls:** some syscalls (e.g. `inotify`) are only stubs in the source; programs depending on them may misbehave.
- **No Android APIs:** there are no equivalents for notifications, SMS, camera, vibration, etc.
- **Shell startup:** the default login shell is `ash`, so `~/.bashrc` is not read unless you start `bash` manually. For `ash`, use `~/.profile`.
- **Rule-based conversion:** the tool uses regex-style rules; review the output before running sensitive commands (deleting files, download-and-run scripts).

## 6. Code structure

Everything is in one file:

| Part | Role |
|---|---|
| `MSG` / `UI` | Persian and English message and interface strings |
| `PKG` / `DROP` | name mapping table and the set of packages to drop |
| `splitOps` | splits a line on `&&`, `\|\|`, `;` while respecting quotes and parentheses |
| `convPkg` | `pkg`/`apt` to `apk` |
| `convTermux` | `termux-*` commands |
| `convFetch` | `bash <(curl …)` |
| `convSeg` | combines the rules for each segment (drops `sudo`, pip, Android command warnings) |
| `convertAll` | paths, line-by-line processing, heredoc, prerequisites; returns `{out, warns, infos}` where warnings are message keys plus arguments |
| `paint` / `run` / `setLang` | rendering in the current language and UI wiring |

### Adding a package

Add a line to `PKG`:

```js
'termux-name':['alpine-name'],
```

Verify the name with `apk search` inside iSH.

### Adding a message or a language

Warnings and notes are stored as keys (`k0`, `k1`, …) with `{0}`, `{1}` placeholders. To add a language, add an entry to both `MSG` and `UI` and a button/branch in `setLang`.

### Testing

The conversion logic is testable without a DOM: call `convertAll(text, {pre:false, brk:false})` and inspect `out`, `warns` and `infos`.

## 7. Sources (iSH source tree)

| Topic | File in `ish-master` |
|---|---|
| Overview and build | `README.md` |
| apk repositories and Alpine version | `app/gen_apk_repositories.py`, `app/CurrentRoot.h`, `app/CurrentRoot.m` |
| `/dev/clipboard` and `/dev/location` | `app/AppDelegate.m`, `app/PasteboardDevice.m`, `app/LocationDevice.m`, `fs/devices.h` |
| Mounting iOS folders | `app/iOSFS.m` |
| Launch command and `TERM` | `app/UserPreferences.m`, `app/AppDelegate.m` |
| `/proc/ish` | `fs/proc/ish.c` |
| Syscalls | `kernel/calls.c` |

## 8. License

iSH is released under **GPLv3** (with the additional terms in `LICENSE.IOS`). This tool only refers to iSH's documentation and behavior and copies none of its code.
