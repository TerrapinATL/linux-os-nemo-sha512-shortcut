-- Guide Introduction

01. Nemo SHA512 Checksum Actions

Right-click verification for FLAC library checksums in the Nemo file manager. Part of the Master Library SHA512 verification project.

|           |                   |
|-----------|-------------------|
| Created   | July 23, 2026     |
| Tested on | Linux Mint / Nemo |
| Status    | Working           |


-- Overview

This sets up two Nemo right-click actions:

* Verify ALBUM SHA512 Checksums  — runs against: ALBUM.sha512sums.txt   - checking individual track files in an album folder.
* Verify ARTIST SHA512 Checksums — runs against: ARTIST.sha512sums.txt  - checking album directories inside an artist folder.


-- Requirements

* Linux Mint with the Nemo file manager
* Terminal access
* sha512sum (included by default on most distros)


-- Expected folder structure

Artist Folder/
├── ARTIST.sha512sums.txt
└── Album Folder/
    └── ALBUM.sha512sums.txt


-- How it works

Right-click ALBUM.sha512sums.txt
        │
        ▼
Nemo runs verify-sha512
        │
        ▼
Individual album files are verified


Right-click ARTIST.sha512sums.txt
        │
        ▼
Nemo runs verify-artist-sha512
        │
        ▼
Each album directory is verified

--------------------------------------------------------------------------------------------------------------------------------------------------------------

02. Part 1 — Album SHA512 verification

--------------------------------------------------------------------------------------------------------------------------------------------------------------

03. Step 1 — Create the album verification script

--- Bash Script Start ---
```bash

nano ~/.local/bin/verify-sha512

```
--- Bash Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

04. Step 2 — Paste the script

--- nano Paste Script Start ---
```bash

#!/bin/bash

echo "==================================================="
echo " Verifying SHA512 Checksums"
echo "==================================================="
echo

if [ -n "$1" ]; then
    cd "$(dirname "$1")" || exit 1
fi

if [ -f "ALBUM.sha512sums.txt" ]; then
    sha512sum -c ALBUM.sha512sums.txt
else
    echo "MISSING ALBUM.sha512sums.txt"
fi

echo
echo "Verification Complete."
echo
read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

05. Step 3 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

----------------------------------------------------------------------------------------------------------------------------------------------------

06. Step 4 — Make it executable

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/verify-sha512
ls -l ~/.local/bin/verify-sha512   # expect permissions starting with -rwx

```
--- Bash Script End ---

--------------------------------------------------------------------------------------------------------------------------------------------------------------

07. Part 2 — Album Nemo action

--------------------------------------------------------------------------------------------------------------------------------------------------------------

08. Step 5 — Create the action file

--- Bash Script Start ---

```bash

nano ~/.local/share/nemo/actions/verify-album-sha512.nemo_action

```
--- Bash Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

09. Step 6 — Paste the action

--- nano Paste Script Start ---

```ini

[Nemo Action]
Name=Verify ALBUM SHA512 Checksums
Comment=Check track checksums in ALBUM.sha512sums.txt
Exec=/home/YourUserName/.local/bin/verify-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ALBUM.sha512sums.txt;
Icon-Name=dialog-information
Terminal=true

```
--- nano Paste Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

10. Step 7 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

--------------------------------------------------------------------------------------------------------------------------------------------------------------

11. Part 3 — Artist SHA512 verification

--------------------------------------------------------------------------------------------------------------------------------------------------------------

12. Step 8 — Create the artist verification script

--- Bash Script Start ---

```bash

nano ~/.local/bin/verify-artist-sha512
```
--- Bash Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

13. Step 9 — Clear old contents (if replacing an existing script)

In nano, hold `Ctrl+K` until the file is empty.

----------------------------------------------------------------------------------------------------------------------------------------------------

14. Step 10 — Paste the script

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

    while IFS= read -r line; do
        [ -z "$line" ] && continue

        stored_hash=${line%%  *}
        album=${line#*  }

        if [ ! -d "$album" ]; then
            echo "MISSING $album"
            continue
        fi

        actual_hash=$(
            cd "$album" &&
            find . -type f ! -name ALBUM.sha512sums.txt -print0 |
            LC_ALL=C sort -z |
            xargs -0 sha512sum |
            sha512sum |
            cut -d" " -f1
        )

        if [ "$stored_hash" = "$actual_hash" ]; then
            echo "OK        $album"
        else
            echo "MISMATCH  $album"
        fi
    done < ARTIST.sha512sums.txt
else
    echo "MISSING ARTIST.sha512sums.txt"
fi

echo
echo "Verification Complete."
echo
read -rp "Press Enter to close..."

```
--- nano Paste Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

15. Step 11 — Save the script

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

----------------------------------------------------------------------------------------------------------------------------------------------------

16. Step 12 — Make it executable

--- Bash Script Start ---

```bash
chmod +x ~/.local/bin/verify-artist-sha512
ls -l ~/.local/bin/verify-artist-sha512   # expect permissions starting with -rwx

```
--- Bash Script End ---

--------------------------------------------------------------------------------------------------------------------------------------------------------------

17. Part 4 — Artist Nemo action

--------------------------------------------------------------------------------------------------------------------------------------------------------------

18. Step 13 — Create the action file

--- Bash Script Start ---
```bash

nano ~/.local/share/nemo/actions/verify-artist-sha512.nemo_action

```
--- Bash Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

19. Step 14 — Paste the action

--- nano Paste Script Start ---

```ini
[Nemo Action]
Name=Verify ARTIST SHA512 Checksums
Comment=Check album directory checksums in ARTIST.sha512sums.txt
Exec=/home/YourUserName/.local/bin/verify-artist-sha512 %F
Selection=s
Extensions=txt;
Conditions=exact-name ARTIST.sha512sums.txt;
Icon-Name=dialog-information
Terminal=true
Active=true

```
--- nano Paste Script End ---

----------------------------------------------------------------------------------------------------------------------------------------------------

20. Step 15 — Save

In nano: `Ctrl+O`, `Enter`, `Ctrl+X`

--------------------------------------------------------------------------------------------------------------------------------------------------------------

21. Part 5 — Restart Nemo

--------------------------------------------------------------------------------------------------------------------------------------------------------------

--- Bash Script Start ---
```bash

nemo -q

```
--- Bash Script End ---

--------------------------------------------------------------------------------------------------------------------------------------------------------------

22. Part 6 — Testing

--------------------------------------------------------------------------------------------------------------------------------------------------------------

-- Test album:

     1. Open an album folder containing: ALBUM.sha512sums.txt
     2. Right-click it → Verify ALBUM SHA512 Checksums
     3. Expect `OK` output for each file

-- Test artist:

     1. Open an artist folder containing: ARTIST.sha512sums.txt
     2. Right-click it → Verify ARTIST SHA512 Checksums

Known-working test:

Folder: /media/YourUserName/drive/ArtistName
File:   ARTIST.sha512sums.txt

Result:

=== ArtistName ===
OK  AlbumName

----------------------------------------------------------------------------------------------------------------------------------------------------

23. Checksum file format

ARTIST.sha512sums.txt — one line per album: SHA512_HASH  Album Directory Name


Example:

b47535abe91048fd9f224d1d6ad74c3056b006491f74de9c0227b646b1a84e861422a4bcd85a7c33854ecf619074cd919298cb5768a0b933a578a207553631b8  AlbumName


The album name is a directory, not a file — the artist script computes a hash-of-hashes across the album's own contents (excluding ALBUM.sha512sums.txt itself) and compares it against the stored value.

----------------------------------------------------------------------------------------------------------------------------------------------------

24. File locations

| Item         | Path                           |
|--------------|--------------------------------|
| Scripts      | ~/.local/bin/                  |
| Nemo actions | ~/.local/share/nemo/actions/   |

Working Exec= format (do not change without testing): Exec=/home/YourUserName/.local/bin/script-name %F

The artist script requires `$1` because Nemo launches actions from an unknown working directory; the script `cd`s into the directory containing the selected ARTIST.sha512sums.txt.

----------------------------------------------------------------------------------------------------------------------------------------------------

25. Troubleshooting

Script works from terminal but the Nemo right-click action does nothing:

* Confirm the action file is in: ~/.local/share/nemo/actions/
* Confirm the Exec= line matches: Exec=/home/YourUserName/.local/bin/verify-artist-sha512 %F
* Restart Nemo: nemo -q

----------------------------------------------------------------------------------------------------------------------------------------------------

26. Backup

Keep copies of:

* ~/.local/bin/verify-sha512
* ~/.local/bin/verify-artist-sha512
* ~/.local/share/nemo/actions/verify-album-sha512.nemo_action
* ~/.local/share/nemo/actions/verify-artist-sha512.nemo_action

----------------------------------------------------------------------------------------------------------------------------------------------------

27. Restore from backup

Use after a Linux reinstall, system rebuild, or move to another machine.

    1. Copy: verify-sha512 and verify-artist-sha512 to ~/.local/bin/
    2. Copy: verify-album-sha512.nemo_action and verify-artist-sha512.nemo_action to ~/.local/share/nemo/actions/
    3. Make scripts executable:

--- Bash Script Start ---
```bash

chmod +x ~/.local/bin/verify-sha512
chmod +x ~/.local/bin/verify-artist-sha512

```
--- Bash Script End ---

    4. Restart Nemo:

--- Bash Script Start ---
```bash

nemo -q

```
--- Bash Script End ---

    5. Test both actions as described in: Part 6 — Testing  (#part-6--testing).

Setup is confirmed restored once both actions appear in the Nemo right-click menu.


