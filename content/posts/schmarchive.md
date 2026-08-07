---
title: "Schmarchive: A photo organizer"
description: "Building a local AI powered tool for fixing my photo archive"
date: 2026-07-31
tags: [photo-management, local-ai, python, computer-vision, metadata, vibe-coding]
draft: false
---

{{< repository-link
  url="https://github.com/gokhandemirtas/schmarchive"
  name="Schmarchive"
  description="Browse the source code and documentation."
>}}

I started Schmarchive because my photo library has been steadily growing by me dumping stuff from every phone, camera, backup, and backups of backups on a drive. And I kept procrastinating and never organized it. There is a particular despair instilled by looking at a filename like
`IMG_20250703_143012.jpg`. It tells me when the camera thinks the picture was
taken, but not where I was, what I was looking at, or whether I already have
four almost identical copies somewhere else. I wanted a small tool that could
weed out unwanted, redundant and pointless photos.

Enter Schmarchive: a command-line archive organizer that non-destructively does that.
It can rename pictures from metadata stored as EXIF dates and GPS, reverse-geocode locations,
and use that to scaffold a folder structure, find duplicate or near-
duplicate images. A local vision model can then help decide which repeated
shot is best, sort photos into broad categories, or find every image containing
something like a white cat.

## The unglamorous bits are the interesting bits

The first version was mostly metadata and file operations. That is still the
heart of it. Before asking a model anything, Schmarchive extracts file
hash for comparing exact copies and a perceptual hash for images that look similar. Only
the candidate groups are sent to the vision endpoint. The model confirms the
group and picks the sharpest, best-exposed, best-composed image. The remaining
files are renamed with a duplicate suffix before they can be moved aside.

That intermediate step matters. File management is one of those areas where a
confident wrong answer is worse than no answer. I would rather see
`photo__near_dupe.jpg` and decide what to do than discover that an assistant
quietly deleted a memory.

## Geotagging
Geotagging has a similar practical compromise. Calling a free reverse-geocoding API (Nominatim)
once per image would be wasteful and impolite, so photos with nearby coordinates are first
clustered by a configurable distance and one result is cached for the group. The file gets a name such as `cordoba-2025-03-24.jpg`, while the geocoding work is kept in a local `locations.csv` cache for later runs.

## Why local AI?

I wanted the ability to pluck photos by subject, e.g. "a white cat", which prompts the vision model to return Detected / Not detected for that subject.
If no subject is provided, it can also do it by a preset list of generic categories e.g. `nature, cityscape, nightshot` etc. which can classify photos without supervision.

Simple renaming, copying, moving, as well as any other deterministic action is done programmatically.

## Other goodies

Metadata is inconsistently present, model can be wrong,
and my archive is full of old photos. But it solves a real problem for me:
turning an overwhelming number of files into something I can enjoy.

Here are the full list of commands you can run:

```
  1. Geotag files — rename photos by location and date.
  2. Date-tag files — rename photos using capture dates.
  3. Deduplicate — find exact and near-duplicate images with hashes and AI verification.
  4. Move duplicates — move marked duplicates into a separate folder.
  5. Normalize filenames — replace accented/non-ASCII characters.
  6. Organize photos — sort by year/month/location or AI-detected subject.
  7. Identify landmarks — rename photos using GPS-based landmark information.
  8. Pluck images by subject — find and move images matching a description.
  9. Configure settings — change folders, model, API, thresholds, and related options.
  10. Process videos — geotag and organize videos by date and location.
```
