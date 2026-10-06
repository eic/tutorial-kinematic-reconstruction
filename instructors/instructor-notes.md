---
title: "Instructor Notes"
---

This tutorial builds on the [Setting Up Your Environment](https://eic.github.io/tutorial-setting-up-environment/)
and [Analysis](https://eic.github.io/tutorial-analysis/) tutorials. Learners should already have a
working `eic-shell` and be comfortable opening ROOT files.

## Before the session

- Ask learners to complete the [Setup](../learners/setup.md) page in advance, including downloading
  at least one of the simulation files with `xrdcp`, since the download can be slow on some
  networks.
- The tutorial uses the current `eic-shell` together with files from the `26.04.1` reconstruction
  campaign. Simulation campaigns are periodically purged, so before running the session check that
  the `xrdcp` files in episode 1 still have an open-access copy (`rucio replica list file --protocols root --pfns --rses isopenaccess ...`) and repoint
  to a live campaign if needed (see the repository `MIGRATION.md`).

## Timing

Each of the four episodes is roughly 15 minutes of teaching plus a short exercise. Most of the
wall-clock time is spent running the ROOT macros over the simulation files.
