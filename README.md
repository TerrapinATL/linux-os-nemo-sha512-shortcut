# [Nemo SHA512 Checksum Actions](https://github.com/TerrapinATL/nemo-sha512-actions/edit/main/Readme.md)

A collection of right-click verification tools and bash scripts designed for validating FLAC library album and artist checksums directly within the Nemo file manager on Linux.

## Overview

This repository contains documentation and scripts to help maintain a secure, verified digital music library. It focuses on integrating `sha512sum` integrity checks into graphical file management workflows for both individual album tracks and artist directory hashes.

## Included Tools & Guides

* **Album Verification Script (`verify-sha512`):** Validates individual track files against an `ALBUM.sha512sums.txt` manifest.
* **Artist Verification Script (`verify-artist-sha512`):** Computes a hash-of-hashes across album directories inside an artist folder and compares them against `ARTIST.sha512sums.txt`.
* **Nemo Actions:** Right-click context menu configurations (`.nemo_action`) to execute both verification scripts seamlessly.

## Prerequisites

Ensure you have the following requirements on your system:

* Linux Mint with the Nemo file manager
* Terminal access
* `sha512sum` (included by default on most distributions)



