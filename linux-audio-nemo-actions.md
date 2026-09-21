### linux-audio-nemo-actions

**Version: v5** — Regenerate-action report labels refined: an entry
whose checksum already existed under a different name (a folder or file
rename) now reports **[UPDATED]** instead of [NEW]; [NEW] is reserved
for genuinely new checksum values. (Refined 2026-09-21.)

Change log and version history are maintained separately:
[linux-audio-nemo-actions-changelog.md](linux-audio-nemo-actions-changelog.md)

---

01. Introduction

---

This guide installs a set of Nemo right-click actions for moOde-aware music libraries on Linux Mint. It brings together the SHA512 checksum verification actions and the ReplayGain actions into one place, as part of the **moOde Library Integrity Suite**.

Unlike the whole-library guides in this suite (moOde Cleanup and SHA512 Library), these Nemo actions are designed for quick, **Artist- or Album-specific** work. They are invoked by right-clicking on a file or folder in the Nemo file manager and run instantly, without loading a full workflow.

**Shortcut: automated installation.** The manual nano/paste steps in Parts 1–6, 2A and 2B describe exactly what to install, but you do not have to do them by hand — **[OpenCode](https://opencode.ai/) can install all of these actions automatically upon request.** Ask it to "install the Nemo actions from the guide" and it will extract every script and `.nemo_action` file from this document into `~/.local/bin/` and `~/.local/share/nemo/actions/` (replacing `<YOURUSERNAME>` placeholders, making scripts executable, and restarting Nemo), then re-verify the installation. It can also uninstall or refresh individual actions on request. This works for any guide in the suite: the guide text is the single source of truth, so an OpenCode-assisted install is always the current version.

The actions installed here are:

* Verify ALBUM SHA512 Checksums — verifies the individual track files in an album folder.
* Verify ARTIST SHA512 Checksums — verifies each album directory inside an artist folder.
* Regenerate ALBUM SHA512 Checksums — rebuilds `ALBUM.sha512sums.txt` for an album folder after an intentional change (re-tag, added/removed file).
* Regenerate ARTIST SHA512 Checksums — rebuilds `ARTIST.sha512sums.txt` from all album folders after an intentional change (folder rename, added album).
* Show ReplayGain — displays the current ReplayGain tags of one or more selected audio files.
* Apply ReplayGain (Loudgain) — computes and writes Album + Track ReplayGain across every supported audio file in a folder.
* Report Tag/Filename Mismatches — scans a folder and reports every file whose embedded tags disagree with the filename (a check that the moOde display is correct).
* Write Tags from Folder/File Names — losslessly rewrites Artist/Album/Year/Title/TrackNumber tags from the naming convention, fixing crosswired or missing tags.

Note: These actions print their results to the terminal (or a popup) rather than writing log files. That is a deliberate design choice — right-click actions are meant to be quick spot checks, and they leave no stray files behind in your music folders.

-- Important: replace YOURUSERNAME

Every `.nemo_action` file in this guide contains an `Exec=` line with a `<YOURUSERNAME>` placeholder, for example `/home/<YOURUSERNAME>/.local/bin/verify-album-sha512`. Before the actions will run, replace `<YOURUSERNAME>` with your actual Linux username in **each** action file. Do not paste the placeholder literally — Nemo will simply do nothing if the path does not exist.

This is the single most common reason the actions appear not to work for someone new. If a right-click action silently fails, check that you replaced the placeholder and that the script exists at the path you gave it.

---

02. Requirements

---

* Linux Mint with the Nemo file manager.
* Terminal access.
* sha512sum (included by default on most distros).
* ffprobe (for the Show ReplayGain action; ships with ffmpeg).
* zenity (for the Show ReplayGain popup; usually preinstalled on Linux Mint).
* loudgain (for the Apply ReplayGain action). Install with:

--- Bash Script Start ---
```bash

sudo apt install loudgain

```
--- Bash Script End ---

* flac package (provides `metaflac`, used by the Write Tags action for FLAC).
* eyeD3 (used by the Write Tags action for MP3). Install with `sudo apt install python3-eyed3` (Debian family) or `python3 -m pip install --user eyeD3`.
* AtomicParsley (used by the Write Tags action for M4A/MP4). Install with `sudo apt install atomicparsley`.
* jq (for the Report Tag/Filename Mismatches action). Install with `sudo apt install jq`.

-- Expected folder structure

The SHA512 actions rely on the two-level manifest convention used across the suite:

```
Artist Folder/
├── ARTIST.sha512sums.txt
└── Album Folder/
    └── ALBUM.sha512sums.txt
```

The ReplayGain actions work on any folder containing supported audio files (FLAC, MP3, M4A, OGG, Opus, and others).

---

03. How the Actions Fit the Suite

---

The moOde Library Integrity Suite uses two complementary layers of protection:

* **Artist level** — the quick check. `ARTIST.sha512sums.txt` stores one hash per album directory, so verifying an entire artist is a single fast operation.
* **Album level** — the thorough check. `ALBUM.sha512sums.txt` stores one hash per audio file, so it verifies the audio tracks themselves in detail.

The Nemo actions put both layers on the right-click menu, alongside the ReplayGain tools, so you never need to open a terminal for routine checks.

---

04. Part 1 — Verify ALBUM SHA512 Checksums

---

The album action verifies each track file listed in `ALBUM.sha512sums.txt` inside a single album folder.

-- Step 1 — Create the album verification script

--- Bash Script Start ---
```bash

nano ~/.local/bin/verify-album-sha512

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash

echo "==================================================="
echo " Verifying ALBUM SHA512 Checksums"
echo "==================================================="
echo

# If a file was dropped onto the script, change to its directory.
if [ -n "$1" ]; then
    cd "$(dirname "$1")" || exit 1
fi

if [ -f "ALBUM.sha512sums.txt" ]; then
    stdbuf -oL sha512sum -c ALBUM.sha512sums.txt 2>&1 |
    awk -F': ' '
        NF == 2 {
            printf "%-8s %s\n", $2, $1
            next
        }
        { print }
    '
else
    echo "MISSING  ALBUM.sha512sums.txt"
fi

echo
echo "Verification Complete."
echo
read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

-- Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/verify-album-sha512
ls -l ~/.local/bin/verify-album-sha512   # expect permissions starting with -rwx

```
--- Bash Script End ---

-- Step 5 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/verify-album-sha512.nemo_action

```
--- Bash Script End ---

-- Step 6 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Verify ALBUM SHA512 Checksums
Comment=Check track checksums in ALBUM.sha512sums.txt
Exec=/home/<YOURUSERNAME>/.local/bin/verify-album-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ALBUM.sha512sums.txt;
Icon-Name=dialog-information
Terminal=true
Active=true

```
--- nano Paste Script End ---

-- Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

05. Part 2 — Verify ARTIST SHA512 Checksums

---

The artist action verifies each album directory listed in `ARTIST.sha512sums.txt` inside an artist folder.

-- Step 1 — Create the artist verification script

--- Bash Script Start ---
```bash

nano ~/.local/bin/verify-artist-sha512

```
--- Bash Script End ---

-- Step 2 — Clear old contents (if replacing an existing script)

In nano, hold `Ctrl+K` until the file is empty.

-- Step 3 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash

if [ -n "$1" ]; then
    cd "$(dirname "$1")" || exit 1
fi

echo "==================================================="
echo " Verifying ARTIST SHA512 Checksums"
echo "==================================================="
echo

if [ -f "ARTIST.sha512sums.txt" ]; then
    artist=$(basename "$PWD")
    echo "=== $artist ==="
    echo

    while IFS= read -r line || [ -n "$line" ]; do
        [ -z "$line" ] && continue

        # Extract hash (first word) and album directory (rest of the line)
        stored_hash=$(awk '{print $1}' <<< "$line")
        album=$(sed 's/^[^ ]*[ ]*//' <<< "$line")

        if [ ! -d "$album" ]; then
            printf "%-10s %s\n" "MISSING" "$album"
            continue
        fi

        actual_hash=$(
            cd "$album" &&
            find . -type f ! -name "ALBUM.sha512sums.txt" -print0 |
            LC_ALL=C sort -z |
            xargs -0 sha512sum |
            sha512sum |
            cut -d' ' -f1
        )

        if [ "$stored_hash" = "$actual_hash" ]; then
            printf "%-10s %s\n" "OK" "$album"
        else
            printf "%-10s %s\n" "MISMATCH" "$album"
        fi
    done < ARTIST.sha512sums.txt
else
    printf "%-10s %s\n" "MISSING" "ARTIST.sha512sums.txt"
fi

echo
echo "Verification Complete."
echo
read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

-- Step 4 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 5 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/verify-artist-sha512
ls -l ~/.local/bin/verify-artist-sha512   # expect permissions starting with -rwx

```
--- Bash Script End ---

-- Step 6 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/verify-artist-sha512.nemo_action

```
--- Bash Script End ---

-- Step 7 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Verify ARTIST SHA512 Checksums
Comment=Check album directory checksums in ARTIST.sha512sums.txt
Exec=/home/<YOURUSERNAME>/.local/bin/verify-artist-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ARTIST.sha512sums.txt;
Icon-Name=dialog-information
Terminal=true
Active=true

```
--- nano Paste Script End ---

-- Step 8 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

06. Part 2A — Regenerate ALBUM SHA512 Checksums

---

Rebuilds `ALBUM.sha512sums.txt` for one album folder. Use this after an **intentional** change to an album's contents: re-tagging, adding or removing a file, replacing a corrupted track with a restored copy. It hashes every top-level file in the folder except the two manifest files, matching the SHA-512 Library guide's Step 2 convention exactly.

The script reports every file as `[SAME]`, `[UPDATED]` (checksum unchanged, name changed — a rename), `[NEW]`, `[CHANGED]` or `[REMOVED]` against the previous manifest. **`[CHANGED]` means the file's audio content changed** — if you did not intentionally change it, stop and investigate (possible bit rot or an incomplete copy) instead of accepting the new manifest.

-- Step 1 — Create the regeneration script

--- Bash Script Start ---
```bash

nano ~/.local/bin/regen-album-sha512

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```python

#!/usr/bin/env python3
# ============================================================
# regen-album-sha512 — Regenerate ALBUM.sha512sums.txt
# Nemo action: right-click ALBUM.sha512sums.txt (or the album
# folder). Hashes every top-level file except the two manifests,
# matching the sha512 guide's Step 2 convention.
# ============================================================
import hashlib
import os
import sys

EXCLUDE = {"ALBUM.sha512sums.txt", "ARTIST.sha512sums.txt"}


def sha512_file(path):
    h = hashlib.sha512()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(1024 * 1024), b""):
            h.update(chunk)
    return h.hexdigest()


def load_old(path):
    old = {}
    if os.path.isfile(path):
        with open(path, "r", errors="surrogateescape") as f:
            for line in f:
                parts = line.split(None, 1)
                if len(parts) == 2:
                    old[parts[1].strip()] = parts[0]
    return old


def main():
    target = sys.argv[1] if len(sys.argv) > 1 else os.getcwd()
    if os.path.isfile(target):
        target = os.path.dirname(os.path.abspath(target))
    if not os.path.isdir(target):
        print(f"ERROR: not a directory: {target}")
        return 1
    os.chdir(target)

    entries = sorted(
        f for f in os.listdir(".")
        if os.path.isfile(f) and f not in EXCLUDE
    )
    if not entries:
        print("ALERT: no hashable files in this folder. Nothing written.")
        return 1

    old = load_old("ALBUM.sha512sums.txt")
    old_by_hash = {}
    for n, h in old.items():
        old_by_hash.setdefault(h, n)
    mode = "UPDATE" if old else "CREATE"
    label = os.path.basename(os.path.abspath(target))
    print("=" * 51)
    print(f" {mode}: ALBUM.sha512sums.txt — {label}")
    print("=" * 51)

    lines = []
    changed = added = updated = same = 0
    total = len(entries)
    for i, name in enumerate(entries, 1):
        digest = sha512_file(name)
        lines.append(f"{digest}  {name}")
        prev = old.get(name)
        if prev == digest:
            status, same = "SAME   ", same + 1
        elif prev is not None:
            status, changed = "CHANGED", changed + 1
        elif digest in old_by_hash:
            status, updated = "UPDATED", updated + 1
        else:
            status, added = "NEW    ", added + 1
        print(f"[{status}] [{i}/{total}] {name}")
    for name in old:
        if name not in entries:
            print(f"[REMOVED]            {name}")

    tmp = "ALBUM.sha512sums.txt.new"
    with open(tmp, "w", errors="surrogateescape") as f:
        f.write("\n".join(lines) + "\n")
    os.replace(tmp, "ALBUM.sha512sums.txt")

    print(f"\nOK: {total} entries written to ALBUM.sha512sums.txt")
    print(f"    ({same} unchanged, {updated} renamed, {added} new, {changed} changed)")
    if changed:
        print("NOTE: changed hashes mean the audio changed — if this was")
        print("not intentional, restore the file from backup instead of")
        print("regenerating. See the SHA-512 guide's stray/corruption rules.")
    print("Reminder: the parent ARTIST.sha512sums.txt is now stale —")
    print("right-click it and choose Regenerate ARTIST SHA512 Checksums.")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```
--- nano Paste Script End ---

-- Step 3 — Save the script and make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/regen-album-sha512

```
--- Bash Script End ---

-- Step 4 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/regen-album-sha512.nemo_action

```
--- Bash Script End ---

-- Step 5 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Regenerate ALBUM SHA512 Checksums
Comment=Rebuild ALBUM.sha512sums.txt for this album folder
Exec=/home/<YOURUSERNAME>/.local/bin/regen-album-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ALBUM.sha512sums.txt;
Icon-Name=view-refresh
Terminal=true
Active=true

```
--- nano Paste Script End ---

-- Step 6 — How to use

Right-click the album's `ALBUM.sha512sums.txt` in Nemo → **Regenerate ALBUM SHA512 Checksums**. Review the per-file report, then re-run **Verify ALBUM SHA512 Checksums** (Part 1) to confirm the new manifest verifies clean. Afterwards the parent `ARTIST.sha512sums.txt` is stale — regenerate it too (Part 2B).

\ ---------------------------------------------------------------------------------------

07. Part 2B — Regenerate ARTIST SHA512 Checksums

---

Rebuilds `ARTIST.sha512sums.txt` for one artist folder from all of its album subdirectories. Use this after an **intentional** structural change: renaming an album folder, adding or removing an album, or after Part 2A. The hash-of-hashes algorithm matches the SHA-512 Library guide's Step 4 exactly, so manifests produced here verify against manifests produced by the whole-library run.

The script reports each album as `[SAME]`, `[UPDATED]`, `[NEW]`, `[CHANGED]` or `[REMOVED]` against the previous manifest. A **folder rename** shows as `[UPDATED]` (same hash, new folder name) plus `[REMOVED]` for the old name — that is expected. **`[CHANGED]` means an album's contents changed** — if that was not intentional, investigate before accepting.

It also flags album folders that have no `ALBUM.sha512sums.txt` (they are still hashed — the artist hash excludes the manifest — but lack the per-file protection layer).

-- Step 1 — Create the regeneration script

--- Bash Script Start ---
```bash

nano ~/.local/bin/regen-artist-sha512

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```python

#!/usr/bin/env python3
# ============================================================
# regen-artist-sha512 — Regenerate ARTIST.sha512sums.txt
# Nemo action: right-click ARTIST.sha512sums.txt (or the artist
# folder). Recomputes the hash-of-hashes for every album folder,
# matching the sha512 guide's Step 4 algorithm byte-for-byte:
#   find . -type f ! -name ALBUM.sha512sums.txt | LC_ALL=C sort -z
#   | xargs -0 sha512sum | sha512sum   (first field)
# ============================================================
import hashlib
import os
import sys


def sha512_file(path):
    h = hashlib.sha512()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(1024 * 1024), b""):
            h.update(chunk)
    return h.hexdigest()


def album_hash(album_dir):
    paths = []
    for root, dirs, files in os.walk(album_dir):
        for fn in files:
            if fn == "ALBUM.sha512sums.txt":
                continue
            full = os.path.join(root, fn)
            rel = "./" + os.path.relpath(full, album_dir)
            paths.append(rel)
    paths.sort(key=os.fsencode)
    h = hashlib.sha512()
    for rel in paths:
        fh = sha512_file(os.path.join(album_dir, rel[2:]))
        h.update(fh.encode() + b"  " + os.fsencode(rel) + b"\n")
    return h.hexdigest()


def load_old(path):
    old = {}
    if os.path.isfile(path):
        with open(path, "r", errors="surrogateescape") as f:
            for line in f:
                parts = line.split(None, 1)
                if len(parts) == 2:
                    old[parts[1].strip()] = parts[0]
    return old


def main():
    target = sys.argv[1] if len(sys.argv) > 1 else os.getcwd()
    if os.path.isfile(target):
        target = os.path.dirname(os.path.abspath(target))
    if not os.path.isdir(target):
        print(f"ERROR: not a directory: {target}")
        return 1
    os.chdir(target)

    albums = sorted(
        d for d in os.listdir(".")
        if os.path.isdir(d) and not d.startswith(".")
    )
    if not albums:
        print("ALERT: no album subdirectories found here. Nothing written.")
        return 1

    old = load_old("ARTIST.sha512sums.txt")
    old_by_hash = {}
    for n, h in old.items():
        old_by_hash.setdefault(h, n)
    mode = "UPDATE" if old else "CREATE"
    label = os.path.basename(os.path.abspath(target))
    print("=" * 51)
    print(f" {mode}: ARTIST.sha512sums.txt — {label}")
    print("=" * 51)

    lines = []
    changed = added = updated = same = 0
    total = len(albums)
    for i, album in enumerate(albums, 1):
        digest = album_hash(album)
        lines.append(f"{digest}  {album}")
        prev = old.get(album)
        if prev == digest:
            status, same = "SAME   ", same + 1
        elif prev is not None:
            status, changed = "CHANGED", changed + 1
        elif digest in old_by_hash:
            status, updated = "UPDATED", updated + 1
        else:
            status, added = "NEW    ", added + 1
        print(f"[{status}] [{i}/{total}] {album}")
    for name in old:
        if name not in albums:
            print(f"[REMOVED]            {name}")

    missing = [
        a for a in albums if not os.path.isfile(os.path.join(a, "ALBUM.sha512sums.txt"))
    ]
    if missing:
        print("\nNOTE: album folder(s) with no ALBUM.sha512sums.txt:")
        for a in missing:
            print(f"  {a}")
        print("They are still hashed (the artist hash excludes the manifest),")
        print("but they are not protected at the per-file layer. Consider")
        print("regenerating the album manifest for them too.")

    tmp = "ARTIST.sha512sums.txt.new"
    with open(tmp, "w", errors="surrogateescape") as f:
        f.write("\n".join(lines) + "\n")
    os.replace(tmp, "ARTIST.sha512sums.txt")

    print(f"\nOK: {total} album entries written to ARTIST.sha512sums.txt")
    print(f"    ({same} unchanged, {updated} renamed, {added} new, {changed} changed)")
    if changed:
        print("NOTE: a changed album hash means that album's contents changed")
        print("(rename, re-tag, added/removed file). If nothing was supposed")
        print("to change, investigate before accepting — do not blindly")
        print("regenerate to silence a MISMATCH.")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```
--- nano Paste Script End ---

-- Step 3 — Save the script and make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/regen-artist-sha512

```
--- Bash Script End ---

-- Step 4 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/regen-artist-sha512.nemo_action

```
--- Bash Script End ---

-- Step 5 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Regenerate ARTIST SHA512 Checksums
Comment=Rebuild ARTIST.sha512sums.txt from all album folders
Exec=/home/<YOURUSERNAME>/.local/bin/regen-artist-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ARTIST.sha512sums.txt;
Icon-Name=view-refresh
Terminal=true
Active=true

```
--- nano Paste Script End ---

-- Step 6 — How to use

Right-click the artist's `ARTIST.sha512sums.txt` in Nemo → **Regenerate ARTIST SHA512 Checksums**. Review the per-album report, then re-run **Verify ARTIST SHA512 Checksums** (Part 2) to confirm the new manifest verifies clean.

\ ---------------------------------------------------------------------------------------

08. Part 3 — Show ReplayGain

---

The Show ReplayGain action reads the current ReplayGain tags of one or more selected audio files and displays them. It is read-only and never modifies anything.

It uses ffprobe to read tags uniformly across FLAC (Vorbis comments), MP3 (ID3v2 TXXX frames), and M4A (MP4 freeform atoms). These formats store `REPLAYGAIN_TRACK_GAIN` / `_PEAK` and `REPLAYGAIN_ALBUM_GAIN` / `_PEAK` under the same key names when written by loudgain, so one code path covers all.

-- Step 1 — Create the script

--- Script Start ---
```bash

nano ~/.local/bin/show-replaygain

```
--- Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```python

#!/usr/bin/env python3
# Show ReplayGain tags for one or more selected FLAC/MP3/M4A files.
# Called from a Nemo Action (see show-replaygain.nemo_action).
#
# Uses ffprobe to read tags uniformly across formats:
#   FLAC -> Vorbis comments, MP3 -> ID3v2 TXXX frames, M4A -> MP4 freeform atoms
# All three store REPLAYGAIN_TRACK_GAIN / _PEAK and REPLAYGAIN_ALBUM_GAIN / _PEAK
# under the same key names when written by loudgain, so one code path covers all.

import os
import subprocess
import sys


def get_tags(path):
    try:
        out = subprocess.run(
            ["ffprobe", "-v", "error", "-show_entries", "format_tags",
             "-of", "default=noprint_wrappers=1", path],
            capture_output=True, text=True, timeout=30,
        ).stdout
    except Exception:
        out = ""
    tags = {}
    for line in out.splitlines():
        if line.startswith("TAG:") and "=" in line:
            key, val = line[4:].split("=", 1)
            if key.lower() not in tags:
                tags[key.lower()] = val
    return tags


def main():
    files = [f for f in sys.argv[1:] if os.path.isfile(f)]
    if not files:
        return

    rows = []
    album_gain_seen = album_peak_seen = album_artist_seen = album_seen = None
    mismatch = artist_mismatch = album_mismatch = False
    last = {}

    for path in files:
        tags = get_tags(path)
        tg = tags.get("replaygain_track_gain")
        tp = tags.get("replaygain_track_peak")
        ag = tags.get("replaygain_album_gain")
        ap = tags.get("replaygain_album_peak")
        aa = tags.get("album_artist")
        al = tags.get("album")

        name = os.path.basename(path)
        rows.append((name, tg or "Not set", tp or "Not set"))
        last = {"tg": tg, "tp": tp, "aa": aa, "al": al, "name": name}

        if ag:
            if album_gain_seen is None:
                album_gain_seen, album_peak_seen = ag, ap
            elif ag != album_gain_seen or ap != album_peak_seen:
                mismatch = True

        if aa:
            if album_artist_seen is None:
                album_artist_seen = aa
            elif aa != album_artist_seen:
                artist_mismatch = True

        if al:
            if album_seen is None:
                album_seen = al
            elif al != album_seen:
                album_mismatch = True

    artist = last["aa"] or album_artist_seen or "Unknown Artist"
    album = last["al"] or album_seen or "Unknown Album"
    title = f"{artist} — {album}"

    if len(files) == 1:
        text = (
            f"File: {last['name']}\n"
            f"Track Gain: {last['tg'] or 'Not set'}\n"
            f"Track Peak: {last['tp'] or 'Not set'}\n"
            f"Album Gain: {album_gain_seen or 'Not set'}\n"
            f"Album Peak: {album_peak_seen or 'Not set'}"
        )
        subprocess.run(
            ["zenity", "--info", f"--title={title}", "--width=420", f"--text={text}"],
            check=False,
        )
    else:
        text = (f"Album Gain: {album_gain_seen or 'Not set'}    "
                f"Album Peak: {album_peak_seen or 'Not set'}")
        if mismatch:
            text += "\n\u26a0 Album Gain/Peak values are NOT consistent across the selected files"
        if artist_mismatch or album_mismatch:
            text += "\n\u26a0 Tracks appear to be from multiple different albums or artists"

        cmd = [
            "zenity", "--list", f"--title={title}",
            "--width=760", "--height=480", f"--text={text}",
            "--column=Track", "--column=Track Gain", "--column=Track Peak",
        ]
        for row in rows:
            cmd.extend(row)
        subprocess.run(cmd, check=False)


if __name__ == "__main__":
    main()

```
--- nano Paste Script End ---

-- Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/show-replaygain

```
--- Bash Script End ---

-- Step 5 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/show-replaygain.nemo_action

```
--- Bash Script End ---

-- Step 6 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Active=true
Name=Show ReplayGain
Comment=Display ReplayGain tags for selected audio files
Exec=/home/<YOURUSERNAME>/.local/bin/show-replaygain %F
Icon=audio-x-generic
Selection=notnone
Extensions=flac;mp3;m4a;mp4;ogg;opus;wav;aiff;wv;ape;
Quote=double
Dependencies=ffprobe;zenity;python3;

```
--- nano Paste Script End ---

-- Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

09. Part 4 — Apply ReplayGain (Loudgain)

---

The Apply ReplayGain action computes and writes ReplayGain across every supported audio file in the folder, using loudgain. It is intended to be run by right-clicking inside (or on) the folder you want to process, and it reports progress in a terminal.

FLAC, MP3, OGG, Opus, WAV, AIFF, and the other non-MP4 formats get **Album + Track** gain. M4A/MP4 files get **Track gain only** — loudgain has an upstream segfault bug writing album-level tags into MP4/M4A atoms (the same bug documented in the Recertification guide, Step 2B), so this action applies the same workaround: an ffmpeg stream-copy container sanitize followed by track gain.

It does not verify ReplayGain afterwards — use the Show ReplayGain action for that. To re-certify an album after adding ReplayGain, run the relevant checksum steps from the Recertification guide.

-- Step 1 — Create the script

--- Bash Script Start ---
```bash

nano ~/.local/bin/apply-replaygain-folder

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash
# Apply ReplayGain to every supported audio file in a folder.
# Called from a Nemo Action (see apply-replaygain-folder.nemo_action).
#
# M4A/MP4 exception: loudgain has an upstream segfault bug when writing
# album-level tags into MP4/M4A atoms (documented in the Recertification
# guide, Step 2B). M4A/MP4 files are therefore container-sanitized with
# ffmpeg (stream copy) and given Track Gain only; every other format gets
# full Album + Track gain.

# If invoked with a folder path, move into it. Loudgain processes the
# current directory, so this works whether you right-click a folder or
# right-click inside the folder you are viewing.
if [ -n "$1" ] && [ -d "$1" ]; then
    cd "$1" || exit 1
fi

echo "==================================================="
echo " Apply ReplayGain (Loudgain)"
echo "==================================================="
echo
echo "Folder: $(pwd)"
echo

if ! command -v loudgain >/dev/null 2>&1; then
    echo "ERROR: loudgain was not found."
    echo "Install it with: sudo apt install loudgain"
    echo
    read -rp "Press Enter to close..."
    exit 1
fi

shopt -s nullglob nocaseglob
m4a_files=( *.m4a *.mp4 )
other_files=( *.flac *.mp3 *.ogg *.opus *.wav *.aiff *.aif )
shopt -u nullglob nocaseglob

total=$(( ${#m4a_files[@]} + ${#other_files[@]} ))

if [ "$total" -eq 0 ]; then
    echo "No supported audio files found in this folder."
    echo
    read -rp "Press Enter to close..."
    exit 0
fi

echo "Processing $total audio file(s)..."
echo

rc=0

if [ ${#m4a_files[@]} -gt 0 ]; then
    echo "--- M4A/MP4: container sanitize + Track Gain (${#m4a_files[@]} file(s)) ---"
    for f in "${m4a_files[@]}"; do
        tmp=$(mktemp "${TMPDIR:-/tmp}/rg-fixed.XXXXXX.${f##*.}")
        if ffmpeg -nostdin -v error -i "$f" -map 0 -map_metadata 0 -c copy -movflags +faststart "$tmp"; then
            mv "$tmp" "$f"
        else
            echo "Warning: FFmpeg container fix failed for $f (tagging skipped)"
            rm -f "$tmp"
        fi
    done
    loudgain -k -s e -L -- "${m4a_files[@]}"
    [ $? -ne 0 ] && rc=1
fi

if [ ${#other_files[@]} -gt 0 ]; then
    echo
    echo "--- Album + Track Gain (${#other_files[@]} file(s)) ---"
    loudgain -a -k -s e -L -- "${other_files[@]}"
    [ $? -ne 0 ] && rc=1
fi

echo
echo "----------------------------------------"
if [ "$rc" -eq 0 ]; then
    echo "SUMMARY: ReplayGain applied to $total file(s)."
else
    echo "SUMMARY: Loudgain finished with errors."
fi
echo "----------------------------------------"
echo

read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

-- Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/apply-replaygain-folder

```
--- Bash Script End ---

-- Step 5 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/apply-replaygain-folder.nemo_action

```
--- Bash Script End ---

-- Step 6 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Apply ReplayGain (Loudgain)
Comment=Compute and write Album + Track ReplayGain for all audio files in the folder
Exec=/home/<YOURUSERNAME>/.local/bin/apply-replaygain-folder %P
Selection=notnone
Extensions=dir;
Icon-Name=audio-x-generic
Terminal=true
Active=true
Dependencies=loudgain;

```
--- nano Paste Script End ---

Note: `%P` passes the folder the action was launched in, so the script processes the entire enclosing folder regardless of the exact item you right-click.

-- Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

10. Part 5 — Report Tag/Filename Mismatches

---

This action scans a folder (recursively, for the checked scope) and reports every audio file where the embedded metadata differs from the filename — flagging wrong titles, wrong track numbers, or missing tags before moOde ever displays them.

It compares case-insensitively and ignores whitespace, so harmless differences in capitalization are not flagged. Files whose name has no `NN - Title` prefix are reported specially rather than falsely matched.

-- Step 1 — Create the script

--- Bash Script Start ---
```bash

nano ~/.local/bin/report-tag-mismatches

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash
# Check for mismatches between embedded tags and filenames.
# Lenient comparison: case-insensitive, whitespace-trimmed.
# Handles files whose name has no "NN - " prefix (reported, not falsely matched).
#
# The report is written to ~/.logs/linux-audio-nemo-actions/ (suite
# convention: nothing is ever written into the music folders).

if [ -n "$1" ] && [ -d "$1" ]; then
    cd "$1" || exit 1
fi

START_DIR=$(pwd)
LOG_DIR="$HOME/.logs/linux-audio-nemo-actions"
mkdir -p "$LOG_DIR"
OUTPUT_FILE="$LOG_DIR/meta-tag-mismatches.md"
TEMP_RESULTS=$(mktemp)
trap 'rm -f "$TEMP_RESULTS"' EXIT

# Case-insensitive, whitespace-collapsed, trimmed. sed is used instead of
# xargs so titles containing quotes/apostrophes ("Don't Stop") are never
# mangled into false mismatches.
norm() { tr '[:upper:]' '[:lower:]' | tr -s '[:space:]' ' ' | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//'; }

find . -type f \( -iname "*.flac" -o -iname "*.mp3" -o -iname "*.m4a" \) | sort | while read -r filepath; do
    filename=$(basename "$filepath")
    name_no_ext="${filename%.*}"

    if [[ "$name_no_ext" =~ ^[0-9]+[[:space:]]+-?[[:space:]]+(.+)$ ]]; then
        file_track_num=${BASH_REMATCH[0]%%[^0-9]*}
        file_track_name="${BASH_REMATCH[1]}"
    else
        file_track_num="NONE"
        file_track_name="NONE"
    fi

    meta_json=$(ffprobe -v error -print_format json -show_format "$filepath" 2>/dev/null)
    if [ -n "$meta_json" ]; then
        meta_track_num=$(echo "$meta_json" | jq -r '.format.tags | to_entries[] | select(.key | ascii_downcase == "track" or ascii_downcase == "tracknumber") | .value' | head -1)
        meta_track_name=$(echo "$meta_json" | jq -r '.format.tags | to_entries[] | select(.key | ascii_downcase == "title") | .value' | head -1)
    fi

    if [ -z "$meta_track_num" ]; then meta_track_num="MISSING"; fi
    if [ -z "$meta_track_name" ]; then meta_track_name="MISSING"; fi

    file_num_clean="$file_track_num"
    if [ "$file_track_num" != "NONE" ]; then
        file_num_clean=$(printf '%s' "$file_track_num" | sed -E 's/^0+//')
        [ -z "$file_num_clean" ] && file_num_clean="0"
    fi

    meta_num_clean=$(printf '%s' "$meta_track_num" | sed -E 's/^0*([0-9]+).*/\1/')
    [ -z "$meta_num_clean" ] && meta_num_clean="0"

    file_name_norm=$(printf '%s' "$file_track_name" | norm)
    meta_name_norm=$(printf '%s' "$meta_track_name" | norm)

    mismatch=0
    if [ "$file_track_num" = "NONE" ]; then
        mismatch=1
    elif [ "$file_num_clean" != "$meta_num_clean" ]; then
        mismatch=1
    fi

    if [ "$file_track_name" = "NONE" ] || [ "$file_name_norm" != "$meta_name_norm" ]; then
        mismatch=1
    fi

    if [ "$mismatch" -eq 1 ]; then
        artist=$(awk -F/ '{print $(NF-2)}' <<< "$filepath")
        album=$(awk -F/ '{print $(NF-1)}' <<< "$filepath")
        echo "MISMATCH|$artist|$album|$file_num_clean|$file_track_name|$meta_num_clean|$meta_track_name" >> "$TEMP_RESULTS"
    fi
done

{
    echo "# Meta Tag vs Filename Mismatches"
    echo ""
    echo "Generated: $(date)"
    echo "Scope: $START_DIR"
    echo ""
    if [ -s "$TEMP_RESULTS" ]; then
        prev_artist=""; prev_album=""
        sort -t'|' -k2,2 -k3,3 "$TEMP_RESULTS" | while IFS='|' read -r _ artist album file_track file_name meta_track meta_name; do
            if [ "$artist|$album" != "$prev_artist|$prev_album" ]; then
                [ -n "$prev_album" ] && echo ""
                echo "## $artist - $album"
                echo ""
                printf '%s\n' "| File# | File Title | Tag# | Tag Title |"
                printf '%s\n' "|------:|------------|-----:|-----------|"
            fi
            [ "$file_track" = "NONE" ] && file_track="-"
            [ "$meta_track" = "MISSING" ] && meta_track="?"
            printf '| %s | %s | %s | %s |\n' "$file_track" "$file_name" "$meta_track" "$meta_name"
            prev_artist="$artist"; prev_album="$album"
        done
    else
        echo "No mismatches found."
    fi
} > "$OUTPUT_FILE"

echo "Report written to: $OUTPUT_FILE"

```
--- nano Paste Script End ---

-- Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/report-tag-mismatches

```
--- Bash Script End ---

-- Step 5 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/report-tag-mismatches.nemo_action

```
--- Bash Script End ---

-- Step 6 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Report Tag/Filename Mismatches
Comment=Scan for files where embedded tags differ from the filename
Exec=/home/<YOURUSERNAME>/.local/bin/report-tag-mismatches %P
Selection=notnone
Extensions=dir;mp3;m4a;flac;
Icon-Name=dialog-information
Terminal=true
Active=true
Dependencies=ffprobe;jq;

```
--- nano Paste Script End ---

Note: `%P` passes the folder the action was launched in, so the report covers the entire enclosing scope.

-- Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

11. Part 6 — Write Tags from Folder/File Names

---

This is the companion to Part 5: it fixes the mismatches automatically by deriving metadata from the naming convention and writing it losslessly into each file.

Run it from inside an album folder named `YYYY Album Name`, with tracks named `NN - Title.ext`. It infers:

* Artist — from the parent folder name.
* Album year — leading 4-digit year in the album folder name.
* Album name — the folder name with the year stripped off.
* Track number / title — from each filename.

It writes those tags losslessly per format: metaflac (FLAC), eyeD3 (MP3), and AtomicParsley (M4A). M4A is written in place with no re-encoding, so your audio is never altered or recompressed. Files that do not match the `NN - Title` pattern are skipped and reported, never guessed.

-- Naming convention is REQUIRED

This tool only works because the folder and filenames themselves already carry the correct metadata (year in the folder name, track number and title in the filename). That is a deliberate safety property of the library: even if every tag is destroyed, the full metadata can be rebuilt from the file tree alone.

Therefore the convention is a **required precondition**, not a suggestion. Do **not** run this on an album folder that does not follow it:

* The album folder name **must** begin with a 4-digit year, followed by a space: `2020 Demo Album`. Otherwise there is nowhere for the year to come from.
* Track files **must** be named `NN - Title.ext` (a track number, a space, a dash, a space, then the title).

The script refuses to guess. If the album folder has no leading year, it aborts and tells you, rather than write an album with a missing or wrong year that would silently break the rebuild-from-tags property. If a track name does not match the `NN - Title` pattern, that file is skipped and reported so you can fix its name first.

-- Step 1 — Create the script

--- Bash Script Start ---
```bash

nano ~/.local/bin/write-tags-from-names

```
--- Bash Script End ---

-- Step 2 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash
# Write tags (Artist / Album / Year / Title / TrackNumber) from folder + filename.
# Run from inside an album folder. Names expected:
#   Album folder : "YYYY Album Name"
#   Audio files  : "NN - Title.ext"
# Files that don't match the "NN - Title" pattern are skipped and reported.
# All writes are LOSSLESS (metaflac / eyeD3 / AtomicParsley) - no re-encoding.
set -o pipefail

START_DIR=$(pwd)

artist=$(awk -F/ '{print $(NF-1)}' <<< "$START_DIR")
album_dir=$(basename "$START_DIR")
if [[ "$album_dir" =~ ^([0-9]{4})[[:space:]]+(.+)$ ]]; then
    album_year="${BASH_REMATCH[1]}"
    album_name="${BASH_REMATCH[2]}"
else
    echo "==================================================="
    echo " Write Tags from Folder / File Names"
    echo "==================================================="
    echo
    echo "ABORT: The album folder name must begin with a 4-digit"
    printf 'year followed by a space, e.g. "2020 Demo Album".\n'
    printf 'Got: %s\n' "$album_dir"
    echo
    echo "The year has to come from the folder name. Rename the"
    echo "folder to 'YYYY Album Name', then run this again."
    echo
    read -rp "Press Enter to close..."
    exit 1
fi

echo "==================================================="
echo " Write Tags from Folder / File Names"
echo "==================================================="
printf 'Artist : %s\n' "$artist"
printf 'Album  : %s\n' "$album_name"
printf 'Year   : %s\n' "$album_year"
echo

SKIPPED=()
WRITTEN=0
FAILED=0

while IFS= read -r -d '' filepath; do
    filename=$(basename "$filepath")
    ext="${filepath##*.}"
    name_no_ext="${filename%.*}"

    if [[ "$name_no_ext" =~ ^([0-9]+)[[:space:]]+-?[[:space:]]+(.+)$ ]]; then
        track_num_str="${BASH_REMATCH[1]}"
        track_num=$(( 10#${track_num_str} ))
        track_name="${BASH_REMATCH[2]}"
    else
        SKIPPED+=("$filepath")
        printf 'SKIP    %-4s %s (name has no "NN - Title")\n' "$ext" "$filename"
        continue
    fi

    rc=0
    case "${ext,,}" in
        flac)
            metaflac \
                --remove-tag=ARTIST \
                --remove-tag=ALBUMARTIST \
                --remove-tag=ALBUM \
                --remove-tag=DATE \
                --remove-tag=YEAR \
                --remove-tag=TITLE \
                --remove-tag=TRACKNUMBER \
                "$filepath" 2>/dev/null
            metaflac \
                --set-tag="ARTIST=$artist" \
                --set-tag="ALBUMARTIST=$artist" \
                --set-tag="ALBUM=$album_name" \
                --set-tag="DATE=$album_year" \
                --set-tag="YEAR=$album_year" \
                --set-tag="TITLE=$track_name" \
                --set-tag="TRACKNUMBER=$track_num" \
                "$filepath"
            rc=$?
            ;;
        mp3)
            eyeD3 --artist "$artist" \
                --album-artist "$artist" \
                --album "$album_name" \
                --release-year "$album_year" \
                --title "$track_name" \
                --track "$track_num" \
                "$filepath" >/dev/null 2>&1
            rc=$?
            ;;
        m4a)
            args=()
            [ -n "$album_year" ] && args+=(--year "$album_year")
            AtomicParsley "$filepath" \
                --artist "$artist" \
                --albumArtist "$artist" \
                --album "$album_name" \
                --title "$track_name" \
                --tracknum "$track_num" \
                "${args[@]}" \
                --overWrite >/dev/null
            rc=$?
            ;;
        *)
            SKIPPED+=("$filepath")
            printf 'SKIP    %s (unsupported extension)\n' "$filename"
            continue
            ;;
    esac

    if [ "$rc" -eq 0 ]; then
        printf 'OK      %-4s %02d - %s\n' "$ext" "$track_num" "$track_name"
        ((WRITTEN++))
    else
        printf 'FAILED  %-4s %s\n' "$ext" "$filename"
        ((FAILED++))
    fi
done < <(find . -maxdepth 1 -type f \( -iname "*.flac" -o -iname "*.mp3" -o -iname "*.m4a" \) -print0 | sort -z)

echo
echo "----------------------------------------"
printf 'SUMMARY: %d written, %d failed, %d skipped.\n' "$WRITTEN" "$FAILED" "${#SKIPPED[@]}"
echo "----------------------------------------"
echo
printf 'Reminder: Tags were changed, so ALBUM.sha512sums.txt / ARTIST.sha512sums.txt\n'
printf 'are now stale. Re-run "Regenerate ALBUM/ARTIST Checksum" after tagging.\n'
echo
read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

-- Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

-- Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/write-tags-from-names

```
--- Bash Script End ---

-- Step 5 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/write-tags-from-names.nemo_action

```
--- Bash Script End ---

-- Step 6 — Paste the action

--- nano Paste Script Start ---
```ini

[Nemo Action]
Name=Write Tags from Folder/File Names
Comment=Losslessly write Artist/Album/Year/Title/Track tags from the naming convention
Exec=/home/<YOURUSERNAME>/.local/bin/write-tags-from-names %P
Selection=notnone
Extensions=dir;
Icon-Name=accessories-text-editor
Terminal=true
Active=true
Dependencies=AtomicParsley;eyeD3;metaflac;

```
--- nano Paste Script End ---

-- Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

---

12. Part 7 — Restart Nemo

---

--- Bash Script Start ---
```bash

nemo -q

```
--- Bash Script End ---

---

13. Part 8 — Testing

---

-- Test album SHA512:

     1. Open an album folder containing: ALBUM.sha512sums.txt
     2. Right-click it → Verify ALBUM SHA512 Checksums
     3. Expect `OK` output for each file

-- Test artist SHA512:

     1. Open an artist folder containing: ARTIST.sha512sums.txt
     2. Right-click it → Verify ARTIST SHA512 Checksums
     3. Expect `OK` output for each album

Known-working test:

Folder: /media/<username>/<drive>/ArtistName
File:   ARTIST.sha512sums.txt

Result:

=== ArtistName ===
OK  AlbumName

-- Test Show ReplayGain:

     1. Select one or more FLAC/MP3/M4A files
     2. Right-click → Show ReplayGain
     3. Expect a popup listing track/album gain and peak values

-- Test Apply ReplayGain:

     1. Open a folder with supported audio that lacks ReplayGain tags
     2. Right-click inside the folder → Apply ReplayGain (Loudgain)
     3. Expect a terminal showing loudgain progress and a `SUMMARY:` line
     4. Right-click the files → Show ReplayGain to confirm tags were written

---

14. Checksum File Formats

---

`ARTIST.sha512sums.txt` — one line per album: `SHA512_HASH  Album Directory Name`

Example:

b47535abe91048fd9f224d1d6ad74c3056b006491f74de9c0227b646b1a84e861422a4bcd85a7c33854ecf619074cd919298cb5768a0b933a578a207553631b8  AlbumName

The album name is a directory, not a file — the artist script computes a hash-of-hashes across the album's own contents (excluding `ALBUM.sha512sums.txt` itself) and compares it against the stored value.

`ALBUM.sha512sums.txt` — one line per track: `SHA512_HASH  filename.ext`, generated by sha512sum.

---

15. File Locations

---

| Item         | Path                          |
|--------------|-------------------------------|
| Scripts      | ~/.local/bin/                 |
| Nemo actions | ~/.local/share/nemo/actions/  |

Working `Exec=` format (do not change without testing): `Exec=/home/<YOURUSERNAME>/.local/bin/script-name %F` (or `%P` for the folder action).

The SHA512 scripts require `$1` because Nemo launches actions from an unknown working directory; the script `cd`s into the directory containing the selected manifest.

---

16. Troubleshooting

---

1. Script works from terminal but the Nemo right-click action does nothing.

* Confirm the action file is in: ~/.local/share/nemo/actions/
* Confirm the `Exec=` line matches your real username.
* Restart Nemo: nemo -q

2. "MISSING ALBUM.sha512sums.txt" or "MISSING ARTIST.sha512sums.txt".

The manifest has not been created in that folder. Regenerate it with the **Regenerate ALBUM/ARTIST SHA512 Checksums** actions (Parts 2A/2B), or run the checksum generation step from the Recertification or SHA512 Library guide.

3. An album/artist shows MISMATCH.

The audio has changed (or bit rot / an incomplete copy). Do not regenerate the hash to "fix" it — restore the file from a known-good backup, then re-certify.

4. ReplayGain action not applying.

Confirm loudgain is installed and the `Dependencies=loudgain;` line is present. The Apply ReplayGain action only works inside a folder with supported audio files.

---

17. Backup

---

Keep copies of:

* ~/.local/bin/verify-album-sha512
* ~/.local/bin/verify-artist-sha512
* ~/.local/bin/regen-album-sha512
* ~/.local/bin/regen-artist-sha512
* ~/.local/bin/show-replaygain
* ~/.local/bin/apply-replaygain-folder
* ~/.local/bin/report-tag-mismatches
* ~/.local/bin/write-tags-from-names
* ~/.local/share/nemo/actions/verify-album-sha512.nemo_action
* ~/.local/share/nemo/actions/verify-artist-sha512.nemo_action
* ~/.local/share/nemo/actions/regen-album-sha512.nemo_action
* ~/.local/share/nemo/actions/regen-artist-sha512.nemo_action
* ~/.local/share/nemo/actions/show-replaygain.nemo_action
* ~/.local/share/nemo/actions/apply-replaygain-folder.nemo_action
* ~/.local/share/nemo/actions/report-tag-mismatches.nemo_action
* ~/.local/share/nemo/actions/write-tags-from-names.nemo_action

---

18. Restore from Backup

---

Use after a Linux reinstall, system rebuild, or move to another machine.

    1. Copy the scripts to ~/.local/bin/
    2. Copy the .nemo_action files to ~/.local/share/nemo/actions/
    3. Make scripts executable:

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/verify-album-sha512
chmod +x ~/.local/bin/verify-artist-sha512
chmod +x ~/.local/bin/show-replaygain
chmod +x ~/.local/bin/apply-replaygain-folder
chmod +x ~/.local/bin/report-tag-mismatches
chmod +x ~/.local/bin/write-tags-from-names

```
--- Bash Script End ---

    4. Restart Nemo:

--- Bash Script Start ---
```bash

nemo -q

```
--- Bash Script End ---

    5. Test all six actions as described in Section 11 — Part 8: Testing.

Setup is confirmed restored once the actions appear in the Nemo right-click menu.

\---------------------------------------------------------------------------------------

-- Disclaimer

This guide was developed through iterative collaborative effort between ChatGPT, Claude, Gemini, Mistral and the user. I cannot thank OpenCode project enough. I was about to give up on the other four (well, actually I did) when I came across OpenCode. I run a 10+ year old laptop yet OpenCode ran perfectly well, offloading the heaving lifting to an offsite server.

https://opencode.ai/
