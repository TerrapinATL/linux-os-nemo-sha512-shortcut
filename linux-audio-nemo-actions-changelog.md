### linux-audio-nemo-actions — Change Log

All version changes are appended to this file, newest last, one `## vX Change Log` section per version.

**Update rule:** before writing to the version-less main guide file, the current content must first be saved as a versioned copy (e.g. `linux-audio-nemo-actions-v1.md`) so every published version stays retrievable.

**Current version: v3** — converts the Show ReplayGain action's script
from bash to extensionless Python (see the v3 entry below).

Main guide: [linux-audio-nemo-actions.md](linux-audio-nemo-actions.md)

---

## v1 Change Log (2026-09-14)

First version. Merges the SHA512 and ReplayGain Nemo actions into a single
guide, and adds tag-verification and tag-writing actions.

Review corrections applied 2026-09-14:

* **Apply ReplayGain M4A/MP4 crash fix** — the action previously ran
  loudgain in album mode on M4A files, which the suite's Recertification
  guide documents as an upstream loudgain segfault bug (album-level tags
  written into MP4/M4A atoms). M4A/MP4 files are now container-sanitized
  with ffmpeg (stream copy) and given Track Gain only, matching Recert
  Step 2B; all other formats keep full Album + Track gain. The format
  list was also aligned with the Recertification guide (aac/ape/wv/mpc/
  spx removed; wav/aiff added).
* **Mismatch report moved out of the music folders** — the Part 5 report
  was written to `meta-tag-mismatches.md` inside the album folder,
  contradicting this guide's own "no stray files in music folders"
  promise. It now goes to
  `~/.logs/linux-audio-nemo-actions/meta-tag-mismatches.md`.
* **False-mismatch fix for titles with quotes** — the lenient name
  comparison used `xargs` to trim whitespace, which mangles titles
  containing apostrophes or quotes ("Don't Stop") into false mismatches;
  replaced with a safe sed trim.
* **Extension lists aligned with the suite** — Show ReplayGain now offers
  flac/mp3/m4a/mp4/ogg/opus/wav/aiff/wv/ape (was flac/mp3/m4a only);
  Apply ReplayGain's internal file list now matches the Recertification
  guide's supported formats.
* **Requirements completed** — added flac (metaflac), eyeD3,
  AtomicParsley, and jq, which the Write Tags and Mismatch Report actions
  require and their Nemo action files declare in `Dependencies=`.
* **Cross-guide note** — the ARTIST-manifest verification algorithm used
  here matches the canonical SHA-512 guide (recursive, all files except
  `ALBUM.sha512sums.txt`). The Recertification guide's manifest steps
  were aligned to the same algorithm in its own update.
* **Step numbering simplified** — the original single continuous counter
  (Steps 1–43 across all Parts) was replaced with per-Part numbering:
  each Part restarts at Step 1, so "Part 5, Step 3" replaces the old
  global "Step 31". Parts remain the unique handle for cross-references
  (no prose referenced the global step numbers).
* Minor fixes: "seven actions" → six in the restore instructions; grammar
  in the YOURUSERNAME note; SKIP report column formatting; long dash
  dividers standardized to the suite's 87-dash convention.

---

## v2 Change Log (2026-09-17)

* **Automated-installation note added to the Introduction.** Documents
  that [OpenCode](https://opencode.ai/) can install all six actions
  automatically upon request — extracting every script and
  `.nemo_action` file from the guide into `~/.local/bin/` and
  `~/.local/share/nemo/actions/`, replacing `<YOURUSERNAME>` placeholders,
  making scripts executable, and restarting Nemo — since the guide text
  is the single source of truth, an OpenCode-assisted install is always
  the current version. It can also uninstall or refresh individual
  actions on request.
* No changes to the six actions themselves; v1's script and action
  content is unchanged.
* First OpenCode-assisted install performed 2026-09-17: all six actions
  deployed from the guide (three refreshed, three newly installed;
  prior copies backed up under
  `~/.local/share/nemo/actions/OLD/scripts-backup-2026-09-17/`).

---

## v3 Change Log (2026-09-20)

* **Show ReplayGain script converted to Python** — `show-replaygain.sh`
  (bash) is now `show-replaygain` (extensionless Python with a
  `#!/usr/bin/env python3` shebang), per the no-`.sh`-files convention.
  Functionality is unchanged: ffprobe tag reading, single-file info popup,
  multi-file comparison table with album/artist consistency warnings.
* **Exec line updated** — `Exec=/home/<YOURUSERNAME>/.local/bin/show-replaygain %F`
  (was `show-replaygain.sh`).
* **Dependencies updated** — the action's `Dependencies=` line now includes
  `python3` (`Dependencies=ffprobe;zenity;python3;`).
* **Guide steps updated** — Steps 1/2/4/6 of Part 3 now show the Python
  script and the extensionless filename; the Backup list and the Restore
  section's chmod commands were updated to match.
* **Versioned copy** — the prior guide (v2) was archived as
  `linux-audio-nemo-actions-v2.md` before editing, per the update rule.

## v4 Change Log (2026-09-21)

* **New Part 2A — Regenerate ALBUM SHA512 Checksums.** Rebuilds
  `ALBUM.sha512sums.txt` for one album folder (hashes every top-level
  file except the two manifests, matching the SHA-512 Library guide's
  Step 2 convention). Reports each file as SAME/NEW/CHANGED/REMOVED
  against the previous manifest; warns on CHANGED so a genuine change
  is never silently accepted.
* **New Part 2B — Regenerate ARTIST SHA512 Checksums.** Rebuilds
  `ARTIST.sha512sums.txt` from all album folders, replicating the
  SHA-512 Library guide's Step 4 hash-of-hashes algorithm byte-for-byte
  (verified against the original bash pipeline on a fixture: recursive
  `find` excluding `ALBUM.sha512sums.txt`, `LC_ALL=C` byte sort,
  `sha512sum` of the listing). A folder rename shows as REMOVED + NEW
  with identical hashes; CHANGED flags content changes.
* **Both scripts are extensionless Python** in `~/.local/bin/`
  (`regen-album-sha512`, `regen-artist-sha512`) per the no-`.sh`
  convention, with live per-item progress lines.
* **Filled a dangling reference:** the moode-cleanup guide's Step 9
  already told readers to "Re-run ... the 'Regenerate ALBUM/ARTIST
  Checksum' Nemo action" — those actions did not exist until this
  version.
* Guide parts renumbered (2A/2B inserted after Part 2 per the
  no-renumber convention); Troubleshooting and Backup sections updated;
  versioned copy of the prior guide archived as
  `linux-audio-nemo-actions-v3.md` before editing.

## v5 Change Log (2026-09-21)

* **Report labels refined in Parts 2A/2B.** An entry whose checksum
  already existed under a different name — a folder or file rename —
  previously reported as [NEW] + [REMOVED], which wrongly implied new
  data. It now reports **[UPDATED]** (checksum unchanged, name changed),
  reserving [NEW] for genuinely new checksum values. Full label set:
  SAME / UPDATED / NEW / CHANGED / REMOVED; summary line counts
  unchanged / renamed / new / changed.
* Both embedded scripts and the installed `~/.local/bin/` copies updated
  together (byte-identical); rename, content-change and new-album
  scenarios re-tested on a fixture.
* **Versioned copy** — the prior guide (v4) was archived as
  `linux-audio-nemo-actions-v4.md` before editing, per the update rule.
