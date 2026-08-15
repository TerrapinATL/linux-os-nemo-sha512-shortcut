# [Nemo SHA512 Checksum Actions](https://github.com/TerrapinATL/nemo-sha512-actions/edit/main/Readme.md)

Linux Nemo file manager right-click menu items designed for validating audio library album and artist SHA-512 checksums directly within Nemo.

## Overview

This repository contains setup instructions and scripts for two Nemo right-click actions to verify FLAC library checksums:

    Verify ALBUM SHA512 Checksums — runs against ALBUM.sha512sums.txt to check individual track files in an album folder.

    Verify ARTIST SHA512 Checksums — runs against ARTIST.sha512sums.txt to check album directories inside an artist folder.

## Included Tools & Guides

* **Album Verification Script (`verify-sha512`):** Validates individual track files against an `ALBUM.sha512sums.txt` manifest.

* **Artist Verification Script (`verify-artist-sha512`):** Computes a hash-of-hashes across album directories inside an artist folder and compares them against `ARTIST.sha512sums.txt`.

* **Nemo Actions:** Right-click context menu configurations (`.nemo_action`) to execute both verification scripts seamlessly.

## Prerequisites

Ensure you have the following requirements on your system:

* Linux Mint with the Nemo file manager
* Terminal access
* `sha512sum` (included by default on most distributions)

## Recommended Workflow

A four part series to clean, verify, and lockdown securely the integrity of an audio file library. 

1. linux-audio-moode-prep: https://github.com/TerrapinATL/linux-audio-moode-prep

2. linux-audio-sha512-checksums: https://github.com/TerrapinATL/linux-audio-sha512-checksums

3. linux-os-nemo-sha512-shortcut:https://github.com/TerrapinATL/linux-os-nemo-sha512-shortcut

4. linux-audio-folder-recertification: https://github.com/TerrapinATL/linux-audio-folder-recertification

## Disclaimer

This file was created as a mix of AI generated content, user input, and user editing. It was a cooperative effort between Claude, Gemini, ChatGPT, and user.

## IMPORTANT

Your Original Library should be treated as immutable.

You should only work on a COPY of your Original Library when processing these scripts. The workflow is designed around creating a validated secondary copy, testing the results, and only then promoting that copy to become a replacement.

Before promotion, files should be cleaned, verified with flac -t, and protected with two layers of SHA-512 checksums.

The purpose is to ensure you have a verifiable library that can be copied, backed up, and restored repeatedly while still matching the validated cleaned copy.
