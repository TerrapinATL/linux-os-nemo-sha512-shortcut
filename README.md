# Nemo Actions for Audio Library Maintenance

Linux Nemo file manager right-click actions for verifying, tagging, and applying ReplayGain to an audio library from within Nemo. Companion to the SHA-512 checksum and moOde cleanup guides.

**Guide version: v8** — Nine Nemo right-click actions (SHA-512 verification and regeneration, FLAC integrity test, ReplayGain show/apply, tag-mismatch report, tag write). v4 added **Regenerate ALBUM SHA512 Checksums** and **Regenerate ARTIST SHA512 Checksums** (Parts 2A/2B) for re-certifying after intentional changes such as folder renames; v5 refined the report labels so a rename reports [UPDATED] rather than [NEW]; v6 added the Press-Enter closing prompt to the regeneration actions (2026-09-21, see the change log); v7 added **FLAC Integrity Test (flac -t)** (Part 1A, 2026-09-26, see the change log); v8 changed the SHA-512 regeneration/verification actions to hash AUDIO FILES ONLY (2026-09-26, matching the SHA-512 guide's v16 convention). v3 converted the Show ReplayGain script to extensionless Python. v2 added the automated-installation note: OpenCode can install all actions from the guide on request.

* Full guide: [linux-audio-nemo-actions.md](linux-audio-nemo-actions.md)
* Change log: [linux-audio-nemo-actions-changelog.md](linux-audio-nemo-actions-changelog.md)

## Overview

This repository contains a set of nine Nemo right-click actions:

* **Verify ALBUM SHA512 Checksums** — checks individual track files against an `ALBUM.sha512sums.txt` manifest.
* **FLAC Integrity Test (flac -t)** — recursively runs `flac -t` on every FLAC file under a selected folder and reports a per-file OK/FAIL tally (same check as Recertification guide Step 1; read-only).
* **Verify ARTIST SHA512 Checksums** — computes a hash-of-hashes across album directories inside an artist folder and compares them against `ARTIST.sha512sums.txt`.
* **Regenerate ALBUM SHA512 Checksums** — rebuilds `ALBUM.sha512sums.txt` after an intentional change (re-tag, added/removed file); reports SAME/NEW/CHANGED/REMOVED per file.
* **Regenerate ARTIST SHA512 Checksums** — rebuilds `ARTIST.sha512sums.txt` from all album folders after an intentional change (folder rename, added album); byte-compatible with the whole-library SHA-512 generator.
* **Show ReplayGain** — displays the current ReplayGain tags of selected files in a popup.
* **Apply ReplayGain** — applies ReplayGain to a folder: Album + Track gain for FLAC/MP3/OGG/Opus/WAV/AIFF; Track gain only for M4A/MP4 (loudgain's upstream MP4 atom bug, documented in the Recertification guide).
* **Report Tag/Filename Mismatches** — compares embedded tags against the folder/filename naming convention; the report is written to `~/.logs/linux-audio-nemo-actions/`, never into the music folders.
* **Write Tags from Folder/File Names** — losslessly writes Artist/Album/Year/Title/TrackNumber tags from the naming convention.

## Prerequisites

Ensure you have the following on your system:

* Linux Mint with the Nemo file manager
* Terminal access
* `sha512sum`, `flac`/`metaflac`, `ffmpeg`/`ffprobe`, `eyeD3`, `AtomicParsley`, `loudgain`, `zenity`, and `jq` (the guide lists these per action)

## Recommended Workflow

A four part series to clean, verify, and lockdown securely the integrity of an audio file library.

1. linux-audio-moode-prep: https://github.com/TerrapinATL/linux-audio-moode-prep

2. linux-audio-sha512-checksums: https://github.com/TerrapinATL/linux-audio-sha512-checksums

3. linux-os-nemo-sha512-shortcut: https://github.com/TerrapinATL/linux-os-nemo-sha512-shortcut

4. linux-audio-folder-recertification: https://github.com/TerrapinATL/linux-audio-folder-recertification

## IMPORTANT

Every `.nemo_action` file contains a `<YOURUSERNAME>` placeholder in its `Exec=` line — replace it with your real username before the actions will run.

Your Original Library should be treated as immutable.

You should only work on a COPY of your Original Library when processing these scripts. The workflow is designed around creating a validated secondary copy, testing the results, and only then promoting that copy to become a replacement.

Before promotion, files should be cleaned, verified with `flac -t`, and protected with two layers of SHA-512 checksums.

The purpose is to ensure you have a verifiable library that can be copied, backed up, and restored repeatedly while still matching the validated cleaned copy.

## Disclaimer

This file was created as a mix of AI generated content, user input, and user editing. It was a cooperative effort between Claude, Gemini, ChatGPT, Mistral, and the user, built and polished with the OpenCode project: https://opencode.ai/
