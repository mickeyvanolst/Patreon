# GeoTypeAdvanced (GTA)

GeoTypeAdvanced turns text into animatable per-character geometry in
TouchDesigner. It provides filled surfaces, outlines, and per-glyph data, with
attributes for character, word, and line animation.

## Get started

1. Download and extract the GeoTypeAdvanced release from
   [Patreon](https://www.patreon.com/cw/MickeyvanOlst).
2. Open `GTA_Examples.toe`, or drag `GeoTypeAdvanced.tox` into your own project.
3. Choose a system font or set **Fontfolder** to a directory of `.ttf` / `.otf`
   files. Keep the supplied fonts beside the example project.
4. Enter your text and connect `out_surface` to your POP/render network.

## Guides

| Guide | What it covers |
|---|---|
| [User guide](guide.md) | Outputs, parameter settings, font setup, HTML/CSS styling, and text flow. |
| [POP attribute reference](pop-attributes.md) | 46 GTA-specific attributes with descriptions, example uses, and eight grouped illustrations. |
| [Animation notes](animation-attributes.md) | Normalized indices, pivots, proportional path placement, and attribute-name migration. |

## Help and feedback

[Open an issue](https://github.com/mickeyvanolst/Patreon/issues/new/choose) with
`[GTA]` in the title. Include your GTA version, TouchDesigner version/build,
operating system, and steps to reproduce the problem.

[All documentation](../README.md) · [Patreon resources](../../README.md)
