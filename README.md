# [Nemo SHA512 Checksum Actions](https://github.com/TerrapinATL/nemo-sha512-actions/edit/main/Readme.md)

A collection of right-click verification tools and bash scripts designed for validating FLAC library album and artist checksums directly within the Nemo file manager on Linux[cite: 2].

## Overview

This repository contains documentation and scripts to help maintain a secure, verified digital music library. It focuses on integrating `sha512sum` integrity checks into graphical file management workflows for both individual album tracks and artist directory hashes[cite: 2].

## Included Tools & Guides

* **Album Verification Script (`verify-sha512`):** Validates individual track files against an `ALBUM.sha512sums.txt` manifest[cite: 2].
* **Artist Verification Script (`verify-artist-sha512`):** Computes a hash-of-hashes across album directories inside an artist folder and compares them against `ARTIST.sha512sums.txt`[cite: 2].
* **Nemo Actions:** Right-click context menu configurations (`.nemo_action`) to execute both verification scripts seamlessly[cite: 2].

## Prerequisites

Ensure you have the following requirements on your system[cite: 2]:

* Linux Mint with the Nemo file manager[cite: 2]
* Terminal access[cite: 2]
* `sha512sum` (included by default on most distributions)[cite: 2]



