# Related formats and prior art

Frisson overlaps with several existing ideas, but is not intended to replace all of them.

## M3U / M3U8

M3U and M3U8 are simple flat playlist formats. They are useful as export targets after a Frisson document has been resolved into a concrete queue.

Frisson differs by describing structure and behaviour before resolution.

## XSPF / JSPF

XSPF and JSPF are structured playlist formats that are useful references for track catalogs, metadata, and media locations.

Frisson differs by emphasizing section-level playback behaviour: shuffle this section, choose one of these blocks, repeat this section, and so on.

## SMIL

SMIL is a multimedia presentation language with strong concepts for sequencing, parallel playback, timing, repeats, and media clipping.

Frisson borrows conceptually from this space, but targets music playlist authoring rather than general multimedia presentation.

## Smart playlists

Smart playlists usually select tracks from a media library using rules.

Frisson can coexist with that idea, but its main concern is playback flow: how selected tracks and sections are arranged, varied, and resolved into a queue.

## DJ software and Auto DJ systems

DJ software often supports crossfades, cue points, beat grids, and automatic mixing.

Frisson may describe transition hints, but does not initially define beat matching, time stretching, or audio analysis.
