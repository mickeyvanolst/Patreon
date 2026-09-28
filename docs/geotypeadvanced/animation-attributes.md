# Typography attributes for animation

For the GTA-specific attribute list with descriptions and example uses, see the
[POP attribute reference](pop-attributes.md).

These attributes are stamped onto each glyph's geometry and carried through
`out_surface`, `out_flat_surface`, `out_flat_outline`, and `out_type_data`.
Group metadata is shared by every point of a glyph. `glyphInkUV` and `contourU`
are geometry-point attributes; use the surface/outline outputs for those.

## Sequence and grouping

| Attribute | Meaning |
|---|---|
| `textIndexNorm` | Source `textIndex` divided by the largest measured source index. Internal spaces and newlines retain their gaps. |
| `visibleGlyphIndex` | Contiguous index excluding whitespace, including punctuation. |
| `visibleGlyphIndexNorm` | Visible index divided by visible count minus one. |
| `wordIndexNorm`, `lineIndexNorm` | Existing group index divided by its largest measured index. Blank visual lines retain their gaps. |
| `visibleGlyphIndexInWordNorm`, `textIndexInLineNorm` | Existing index within the group divided by that group's maximum. Line indices include spaces. |
| `visibleGlyphCountInWord`, `visibleGlyphCountInLine` | Non-whitespace character counts, including punctuation. |
| `wordInkSize`, `lineInkSize` | Width and height of the combined ink bounds in TD units, excluding whitespace. |

All single-element index ramps are zero. Source indices describe the existing
character-based layout, not a new shaping or ligature model. Trailing newlines
have no measured glyph and do not extend the normalization range.

## Naming conventions and anchors

Names identify the subject (`glyph`, `word`, `line`) and what is measured.
`Ink` means visible outline bounds; `Layout` includes browser spacing.
`Norm` marks an index ramp, while `U`/`UV` describe spatial coordinates.
`pointIndex` always refers to geometry points. `textIndex` retains whitespace
positions; `visibleGlyphIndex` excludes whitespace and includes punctuation.
Y-reference attributes (`baselineY`, `xHeightY`, `capHeightY`, `ascenderY`,
`descenderY`) are absolute coordinates, not heights above the baseline.

Three different anchors are intentionally exposed:

- `glyphBaselineOrigin`: layout box's left X, baseline Y.
- `glyphBaselineCenter`: ink centre X, baseline Y; useful for baseline pivots.
- `glyphInkCenter`: ink centre X and Y; useful for centred rotation/scaling.

`glyphInkCenterOffsetInLine = glyphInkCenter.x - lineLayoutLeft`.
It is a distance from the line start, whereas `glyphBaselineOrigin.x` is an
absolute position at the character's layout-left edge. Their difference
therefore includes both the line's position and the glyph's bearing/half-width.
`wordInkCenter` and `lineInkCenter` use combined ink bounds, excluding whitespace.

## Proportional placement along a path

| Attribute | Meaning |
|---|---|
| `glyphBaselineOrigin` | Browser-measured left edge and baseline, in TD units. |
| `glyphLayoutWidth` | Width of the browser's character layout box, in TD units; not ink width. |
| `glyphInkCenterOffsetInLine` | Glyph ink-centre X minus its visual line's layout-left X, in TD units. |
| `lineLayoutWidth` | Full measured layout width of that visual line, including whitespace. |
| `glyphInkCenterUInLine` | `glyphInkCenterOffsetInLine / lineLayoutWidth`; zero for a zero-width layout. |

Layout positions retain the spacing actually produced by the browser, including
spaces, CSS spacing and any kerning present in that layout. They do not add a
new shaping engine. Ink overhangs can put `glyphInkCenterUInLine` slightly outside 0–1; it is
not clamped, so bearings remain intact.

For a line strip sampled uniformly by arc length:

1. Use `out_type_data` as the lookup's first input and the resampled path as its
   second input.
2. Look up `P` using `glyphInkCenterUInLine`, with normalized index units and interpolation on.
   Write the result to a separate attribute such as `pathPosition` if you need
   to keep the original position.
3. Translate each glyph by `pathPosition - float3(glyphInkCenter, 0)` to place its ink
   centre on the path. Use a shared transform for every point with that
   `textIndex`. Optional tangent-based rotation needs the same pivot.

This fits each visual line's full layout width to the path. For original-size
spacing, use `(glyphInkCenterOffsetInLine + animatedDistance) / pathLength` instead. Different
lines each start their own layout range; separate or offset them deliberately.

## Spatial and outline coordinates

`glyphInkUV` is `(P.xy - (glyphInkCenter - glyphInkSize/2)) / glyphInkSize`. It is fixed to the
original geometry: X runs left-to-right, Y bottom-to-top. A zero-size dimension
produces zero. Use it for reveals and local gradients; it is not a vertex-order
index.

`contourU` is cumulative polyline length divided by the closed contour's full
perimeter. It restarts for each `(textIndex, contourIndex)`. A duplicated
closing endpoint reaches 1; otherwise the closing edge spans the final value
back to zero. It is zero on surface points and on zero-length contours. Its
accuracy follows Mesh Detail. A contour's start and direction follow the font
outline; they are not guaranteed to match between different fonts.

`pointIndexInGlyphNorm` and `pointIndexInWordNorm` remain mesh-order ramps, with no
promise of equal spatial spacing or consistent values after remeshing.

## Reconstructing a horizontal line

After subtracting `glyphBaselineCenter` from each glyph's `P.xy`, add
`glyphInkCenterOffsetInLine` to `P.x`. This retains proportional spacing and
puts the line's layout-left at X = 0. An evenly spaced sample per `textIndex`
will instead produce fixed spacing regardless of font.

## Pre-release migration

These names replace the previous names without changing their numeric values.
There are no legacy aliases. Update attribute scopes, shader accessors, and
any channels derived from these attributes (for example `MaxtextIndex`).
The machine-readable mapping is [attribute-renames.json](attribute-renames.json).

| Previous | Current |
|---|---|
| `origin` | `glyphBaselineCenter` |
| `bounds` | `glyphInkCenter` |
| `inkSize` | `glyphInkSize` |
| `wordCenter` | `wordInkCenter` |
| `lineCenter` | `lineInkCenter` |
| `wordSize` | `wordInkSize` |
| `lineSize` | `lineInkSize` |
| `layoutOrigin` | `glyphBaselineOrigin` |
| `layoutOffset` | `glyphInkCenterOffsetInLine` |
| `layoutU` | `glyphInkCenterUInLine` |
| `layoutWidth` | `lineLayoutWidth` |
| `glyphAdvance` | `glyphLayoutWidth` |
| `glyphUV` | `glyphInkUV` |
| `glyphIndex` | `textIndex` |
| `glyphIndexNorm` | `textIndexNorm` |
| `indexInLine` | `textIndexInLine` |
| `indexInLineNorm` | `textIndexInLineNorm` |
| `indexInWord` | `visibleGlyphIndexInWord` |
| `indexInWordNorm` | `visibleGlyphIndexInWordNorm` |
| `glyphCountInWord` | `visibleGlyphCountInWord` |
| `glyphCountInLine` | `visibleGlyphCountInLine` |
| `ptIndexInGlyph` | `pointIndexInGlyph` |
| `ptIndexInGlyphNorm` | `pointIndexInGlyphNorm` |
| `ptIndexInWord` | `pointIndexInWord` |
| `ptIndexInWordNorm` | `pointIndexInWordNorm` |
| `baseline` | `baselineY` |
| `xHeight` | `xHeightY` |
| `capHeight` | `capHeightY` |
| `ascender` | `ascenderY` |
| `descender` | `descenderY` |
| `glyphClass` | `characterClass` |
| `geomType` | `geometryType` |
| `WebRenderLoaded` | `webRenderLoaded` |
