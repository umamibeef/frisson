# Frisson

Frisson is a draft specification for a structured smart playlist format.

Traditional playlists are usually flat lists of media items. Smart playlists usually select media from a library using rules. Frisson adds another layer: **playback flow**.

A Frisson file describes how playback moves through named sections, including fixed sections, shuffled sections, choices, repeats, clips, and transition hints.

## Goals

- Describe a playlist as a structured playback program, not only as a flat list.
- Support fixed intros/outros with variable middle sections.
- Support section-level behaviour such as `sequence`, `shuffle`, `choice`, and `weighted_choice`.
- Allow deterministic resolution using a seed.
- Allow rich players to honour transitions, gain, fades, and clips.
- Allow simple tools to resolve a Frisson file into an ordinary playlist such as M3U8.

## File extension

Frisson documents use the `.frisson` extension.

The v0.1 draft uses a small YAML-compatible text encoding, but the public format extension is intentionally not `.yaml` or `.json`.

Example:

```text
radio-hour.frisson
ambient-set.frisson
evening-drive.frisson
```

## Minimal example

```yaml
frisson: "0.1"
title: Evening Set

tracks:
  intro_theme:
    title: Opening Theme
    creator: Artist A
    locations:
      - file:///music/intro-theme.flac

  track_a:
    title: Track A
    creator: Artist B
    locations:
      - file:///music/track-a.flac

  track_b:
    title: Track B
    creator: Artist C
    locations:
      - file:///music/track-b.flac

sections:
  intro:
    mode: sequence
    items:
      - track: intro_theme

  main:
    mode: shuffle
    selection:
      count: 1
      without_replacement: true
    items:
      - track: track_a
      - track: track_b

flow:
  - section: intro
  - section: main
```

## Repository layout

```text
SPEC.md                         Main draft specification
examples/                       Example .frisson documents
schema/frisson.schema.json       Semantic schema after parsing
```

## Status

This is an early design draft. Names, field names, and semantics may change.
