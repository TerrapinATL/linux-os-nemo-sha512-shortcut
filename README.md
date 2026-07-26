# [Nemo SHA512 Checksum Actions](https://github.com/TerrapinATL/nemo-sha512-actions/edit/main/Readme.md)

Linux Nemo file manager right-click menu items designed for validating FLAC library album and artist SHA-512 checksums directly within Nemo.

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



