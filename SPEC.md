# Frisson Draft Specification

Version: 0.1-draft

## 1. Overview

Frisson is a structured smart playlist format for describing dynamic playback experiences.

A Frisson document defines:

1. a catalog of tracks,
2. named playback sections,
3. a top-level flow that resolves those sections into a concrete queue.

A traditional playlist says:

```text
play A, then B, then C
```

A Frisson document can say:

```text
play this fixed intro,
then choose one of these sections,
then shuffle three tracks from this pool,
then play this fixed outro,
using this transition behaviour.
```

Frisson is declarative. It describes intended playback behaviour. A player, resolver, or compiler is responsible for turning that behaviour into a concrete playback queue.

## 2. File extension

Frisson documents use the `.frisson` extension.

The v0.1 draft uses a YAML-compatible syntax. Implementations should treat `.frisson` as its own media type / document type, not as a generic YAML document.

## 3. Design goals

Frisson should:

- be readable enough to author by hand,
- be simple enough to parse and validate,
- support fixed and variable playlist structure,
- support deterministic randomization,
- degrade to ordinary playlists after resolution,
- avoid requiring a specific playback engine,
- avoid becoming a complete music-library database.

Frisson should not initially attempt to define:

- beat matching,
- time stretching,
- BPM analysis,
- waveform analysis,
- audio synthesis,
- library search/indexing semantics.

Those may be useful implementation features, but they are outside the core format.

## 4. Document structure

A Frisson document has this top-level shape:

```yaml
frisson: "0.1"
title: Example Set
creator: Optional Creator

tracks: {}
sections: {}
flow: []
```

Required fields:

- `frisson`
- `tracks`
- `sections`
- `flow`

Optional fields:

- `title`
- `creator`
- `description`
- `defaults`
- `metadata`

## 5. Tracks

The `tracks` map defines the track catalog for the document.

```yaml
tracks:
  intro_theme:
    title: Opening Theme
    creator: Artist A
    locations:
      - file:///music/intro-theme.flac
    duration_ms: 92000
```

Track keys are local identifiers. They are referenced by section items.

### 5.1 Track fields

Required:

- `locations`: list of one or more media URIs

Recommended:

- `title`
- `creator`
- `duration_ms`

Optional:

- `album`
- `identifier`
- `metadata`

A track catalog entry identifies media. It should not contain section-level playback logic.

## 6. Sections

A section is a named playback block.

```yaml
sections:
  main:
    mode: shuffle
    items:
      - track: song_a
      - track: song_b
```

Required section fields:

- `mode`
- `items`

Optional section fields:

- `repeat`
- `selection`
- `mix`
- `transition_in`
- `transition_out`
- `metadata`

## 7. Section modes

### 7.1 `sequence`

Play items in the order provided.

```yaml
sections:
  intro:
    mode: sequence
    items:
      - track: station_id
      - track: opening_theme
```

### 7.2 `shuffle`

Play items in randomized order.

```yaml
sections:
  main:
    mode: shuffle
    items:
      - track: song_a
      - track: song_b
      - track: song_c
```

If `selection` is omitted, all items are selected.

### 7.3 `choice`

Choose one item from the section.

```yaml
sections:
  interlude:
    mode: choice
    items:
      - track: interlude_a
      - track: interlude_b
```

### 7.4 `weighted_choice`

Choose one item using item weights.

```yaml
sections:
  mood_block:
    mode: weighted_choice
    items:
      - section: calm_block
        weight: 3
      - section: high_energy_block
        weight: 1
```

If an item omits `weight`, the default weight is `1`.

## 8. Items

An item references either a track or another section.

```yaml
- track: song_a
```

```yaml
- section: chorus_block
```

An item must not contain both `track` and `section`.

Optional item fields:

- `weight`
- `clip`
- `mix`
- `metadata`

## 9. Flow

The top-level `flow` defines the main playback program.

```yaml
flow:
  - section: intro
  - section: main
  - section: outro
```

Flow entries use the same item shape as section items, but usually reference sections.

## 10. Selection

A `selection` object controls how many items are selected from a section.

```yaml
selection:
  count: 3
  without_replacement: true
```

Fields:

- `count`: number of items to select
- `without_replacement`: whether an item may be selected more than once

For v0.1, `count` must be a non-negative integer.

## 11. Repeat

A section may repeat.

Fixed repeat:

```yaml
repeat: 2
```

Range repeat:

```yaml
repeat:
  min: 2
  max: 4
```

If repeat is omitted, the section is played once.

## 12. Clips

A `clip` object selects a portion of a track.

```yaml
clip:
  start_ms: 30000
  end_ms: 180000
```

Fields:

- `start_ms`
- `end_ms`

A resolver should reject clips where `end_ms` is less than or equal to `start_ms`.

## 13. Mix and transition hints

Frisson can describe transition intent, but v0.1 does not require a player to support every hint.

```yaml
mix:
  gain_db: -2
  crossfade_ms: 6000
  fade_in_ms: 1000
  fade_out_ms: 3000
  normalize: true
```

Suggested fields:

- `gain_db`
- `crossfade_ms`
- `fade_in_ms`
- `fade_out_ms`
- `normalize`

Section-level mix settings apply to items in that section unless overridden by item-level mix settings.

## 14. Defaults

Top-level defaults may define playback hints inherited by sections and items.

```yaml
defaults:
  mix:
    crossfade_ms: 4000
    normalize: true
```

Precedence from lowest to highest:

1. top-level defaults,
2. section settings,
3. item settings.

## 15. Deterministic resolution

A resolver may use a seed to make shuffle and choice decisions deterministic.

```yaml
seed: evening-set-2026-06-06
```

Given the same Frisson document, same resolver version, and same seed, a resolver should produce the same resolved queue.

## 16. Resolution output

A resolver turns a Frisson document into a concrete queue.

Example conceptual output:

```yaml
resolved:
  seed: evening-set-2026-06-06
  items:
    - track: intro_theme
    - track: track_b
    - track: outro_theme
```

Resolved output may then be exported to ordinary formats such as M3U8.

## 17. Compatibility strategy

Frisson is intended to coexist with existing playlist formats:

- M3U/M3U8: useful as a resolved flat playlist export.
- XSPF/JSPF: useful conceptual references for playlist metadata and track catalogs.
- SMIL: useful conceptual reference for sequencing and timing semantics.

Frisson does not claim compatibility with those formats at the document level.

## 18. Minimal v0.1 feature set

The v0.1 draft should focus on:

- `.frisson` text documents,
- track catalog,
- named sections,
- `sequence`, `shuffle`, `choice`, `weighted_choice`,
- top-level flow,
- repeat,
- selection count,
- clip start/end,
- basic mix hints,
- deterministic seed,
- resolved queue export.

## 19. Open questions

- Should `flow` allow inline anonymous sections?
- Should the syntax remain YAML-compatible or become a custom grammar?
- Should section modes use long names or shorter names like `seq` and `pick`?
- Should randomization be specified strongly enough for cross-implementation reproducibility?
- Should Frisson define a media type such as `application/vnd.frisson`?
- Should `locations` allow non-audio media?
- Should the spec include library-query smart playlist rules, or only playback-flow rules?
