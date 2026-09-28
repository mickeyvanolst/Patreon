# GeoTypeAdvanced user guide

A self-contained TouchDesigner COMP that turns text into animatable per-character geometry. Each glyph is triangulated in a browser using opentype.js, sent to TD over a WebSocket, and enriched with typographic attributes (baseline, x-height, cap height, word index, line index, etc.) so individual characters can be animated with typographically correct pivot points.

The release zip contains a standalone
`GeoTypeAdvanced.tox`, the self-contained `GTA_Examples.toe` project, and the
Sono fonts used by its folder-font examples. Works on macOS and Windows.

## Quick start

1. Extract the release zip and open `GTA_Examples.toe`, or drag
   `GeoTypeAdvanced.tox` into your own project
2. Pick a font — turn on **Use System Font** and choose one, or point
   **Fontfolder** at a directory of `.ttf` / `.otf` files
3. Type your text into the **Text** parameter
4. Connect **out\_surface** into your POP/render network

Geometry and per-glyph attributes appear automatically. Use
**out\_flat\_outline** for outline and contour rendering.

## Outputs

Four outputs, all sharing one coordinate space and the same `textIndex`, so
they can be combined freely.

| Output | Content |
|--------|---------|
| `out_surface` | The triangulated fill mesh, extruded when **Depth** > 0 — front face, back face and side walls. |
| `out_flat_surface` | The fill mesh without extrusion: one flat face per glyph, whatever Depth is set to. |
| `out_flat_outline` | Closed contour polylines — outer body and inner holes, such as the counter of an *O*, as separate primitives. |
| `out_type_data` | One point per glyph, carrying the typographic attributes without any geometry. Useful for driving instancing or animation from the text alone. |

Every typographic attribute is written onto the points of all four.

## Parameters

### Text

| Parameter | Type | Description |
|-----------|------|-------------|
| Text | String (multiline) | The text to render. Supports inline HTML tags — see [HTML tag support](#html-tag-support). |
| Text DAT | DAT | Take the text from a DAT instead. Greys out **Text** while it is set. |

The COMP also has a **DAT input**. Wire a DAT into it and its text is used,
which greys out both **Text** and **Text DAT** — the wire wins because it is the
one source you can see in the network. A wire that carries no text is ignored,
so the order is: wire, then **Text DAT**, then **Text**.

Newlines work in all three, including when **Wrapwidth** is set. A trailing
newline on a DAT is dropped rather than rendered as a blank last line.

### Font

| Parameter | Type | Description |
|-----------|------|-------------|
| Fontfolder | Folder | Directory of `.ttf` / `.otf` files, scanned recursively. See [Fonts](#fonts). |
| Fontfamily | Menu | Active font family. Auto-populated after fonts load. |
| Fontweight | Int (100–900) | Weight of the face to use. For a folder font it picks among the files already loaded; for a system font it fetches the matching face. A family that ships one weight ignores it. |
| Font Style | Menu (Regular / Italic) | Style of the face to use. Style is honoured ahead of weight, so an italic request never falls back to an upright face just because its weight is nearer. |
| Fontsize | Float | CSS font size in pixels. Controls the scale of the generated geometry relative to **Scale**. |
| Leading | Float | Line height multiplier. |
| Reload Fonts | Pulse | Re-scans Fontfolder and rebuilds the Fontfamily menu without changing other settings. |
| Use System Font | Toggle | Use a font installed on this machine instead of one from Fontfolder. Greys out the Fontfolder controls. |
| System Font | Menu | The installed font to use. Populated from the fonts on this machine that can actually be read — see [System fonts](#system-fonts). |
| Refresh System Fonts | Pulse | Re-scan installed fonts and rebuild the System Font menu. |

### Typography

| Parameter | Type | Description |
|-----------|------|-------------|
| Tracking | Float (em) | Letter spacing, in em rather than pixels, so it holds its proportion when **Fontsize** changes. `0` leaves the font's own spacing alone. Negative values tighten. |
| Word Spacing | Float (em) | Extra space on word spaces only, on top of Tracking. Also in em. |
| Align | Menu | Left / Center / Right / Justify. See [Alignment](#alignment). |
| Case | Menu | As Typed / UPPERCASE / lowercase / Capitalize. A real change of glyph, not a styling flag — the geometry is rebuilt from the transformed character. |

#### Alignment

Alignment is measured against the **canvas**, not against the text, and the
canvas centre is the TD origin. So `Left` — the default — sets the text off at
about `-Scale / 2`, and `Center` is what lands it on the origin. With
**Wrapwidth** set the block is that width instead, and alignment happens inside
it.

`Justify` needs a **Wrapwidth**: it works by stretching the spaces in a line
until the line fills the measure, and without a wrap width there is no measure
and every line ends in a hard break, which CSS leaves alone. The menu entry says
so while no wrap width is set. As in print, the last line of a paragraph is not
justified.

One CSS behaviour worth knowing: **Tracking** adds its space *after* every
character, including the last one on a line. Centered text with heavy tracking
therefore sits half a tracking unit left of where you might expect.

#### Word Break

On the **Text Flow** page, beside **Wrapwidth**, because it decides what happens
to a word too long for the measure. Greyed out until a wrap width is set, since
without one there is no measure to be too long for.

| Value | Behaviour |
|-------|-----------|
| Never (overflow) | The word hangs out past the wrap box. CSS's default, and rarely what is wanted once a wrap width has been set on purpose. |
| Break Long Words | A word that cannot fit on a line of its own is broken; ordinary words still break only at spaces. |
| Break Anywhere | Every line fills to the measure and breaks wherever it lands, mid-word or not. |

Measured on a 300 px measure with `Supercalifragilisticexpialidocious and short
words too`: Never runs 258 px past the box, Break Long Words splits the long
word only, and Break Anywhere breaks the short words too.

### Layout

| Parameter | Type | Description |
|-----------|------|-------------|
| Scale | Float | Size of the geometry in TD units. Increase to make the text larger in TD space. |
| Depth | Float | Extrusion depth in TD units. `0` = flat `ShapeGeometry`; `> 0` = `ExtrudeGeometry` with front face, back face, and side walls. |
| Depth Segments | Int (1–128) | Subdivisions along the extrusion's Z axis. Default `1` preserves the original mesh. `16` gives 17 depth levels for multistop point-color gradients and deformation. Adds side-wall polygons; does not subdivide the caps or affect flat geometry. |
| Bevel | Toggle | Round the extrusion's front/back edges. Off by default; ignored at zero Depth. Caps inset so the side walls retain the original outline. |
| Bevel Size | Float | Inset distance in TD units, default `0.01`. Keep small relative to the thinnest stroke; large values can overlap or distort narrow glyphs. |
| Bevel Thickness | Float | Depth of each rounded edge in TD units, default `0.02`. Limited internally to just under half of Depth. The finished mesh still spans Z = 0 to Depth. |
| Bevel Segments | Int (1–32) | Layers per rounded edge, default `4`. `1` produces a chamfer; higher values make a smoother curve. Mesh Detail controls the glyph outline resolution; Depth Segments controls the straight wall between the bevels. Adjoining bevel faces share vertices for smooth shading, with caps and sharp corners kept separate. |
| Mesh detail | Int (1–64) | Bézier curve subdivision for glyph outlines. Higher values give smoother curves at the cost of polygon count. Low values are fine for geometric faces; raise it for scripts and serifs. |

### Style

| Parameter | Type | Description |
|-----------|------|-------------|
| CSS | DAT | A Text DAT whose content is injected into the browser page as a `<style>` block. Use this to apply custom CSS to the rendered text — per-element font sizes, colours, and anything the parameters do not cover. Spacing, alignment and case now have parameters of their own; prefer those. |

### System

| Parameter | Type | Description |
|-----------|------|-------------|
| Autoport | Toggle | Automatically selects the first free port in the range 9001–9099. On by default. |
| Webport | Int | The active HTTP/WebSocket port. Read-only when Autoport is on. |
| Generate | Pulse | Manually trigger a geometry pass (re-measures text, re-triangulates, re-sends geometry). |
| Status | String (read-only) | Current state — font loading progress, ready message, or error/warning. |
| Charcount | Int (read-only) | Number of non-space glyphs in the last measurement pass. |
| Install to Palette | Pulse | Saves the COMP as a `.tox` into the TD user palette folder. Refresh the Palette Browser afterwards. |

## HTML tag support

The **Text** parameter accepts inline HTML. Characters inside styled elements are measured and triangulated using the correct font variant:

```
<b>Bold</b> and <i>italic</i> text
<b><i>Bold italic</i></b>
```

Per-character `fontWeight` and `fontStyle` attributes reflect the actual rendered variant. Bold and italic require dedicated font files in Fontfolder — see [Fonts](#fonts).

## CSS injection

Wire a Text DAT into the **CSS** DAT input to supply a stylesheet. Edits update the browser and geometry automatically, without a full page reload. A connected CSS DAT takes priority over **CSS Override**; an empty connected DAT clears the stylesheet. Disconnect it to restore the field's value.

The **CSS Override** field also accepts literal CSS, an expression returning CSS, or a sibling DAT name.

Use it to control anything CSS can express — font size per element, `text-transform`, `letter-spacing`, colour (exported as the native RGBA `Color` POP attribute, with RGB `Cd` retained for compatibility), or custom `@font-face` rules for fonts outside Fontfolder:

```css
#sentence {
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

b {
  color: rgb(255, 100, 50);
}
```

## Fonts

### Fontfolder

Point **Fontfolder** at a directory containing `.ttf` or `.otf` files. Subfolders are scanned recursively, so a Google Fonts download (one subfamily per folder) works out of the box. The **Fontfamily** menu is populated automatically after the browser loads the files.

If no Fontfolder is set, one system font is loaded as a fallback so geometry
still renders, and a warning appears in **Status**. That fallback is a safety
net, not font selection — to choose an installed font, use **Use System Font**
below.

### System fonts

Turn on **Use System Font** to use a font installed on this machine instead of
one from a folder, and pick it from the **System Font** menu.

The menu lists the installed families GeoTypeAdvanced can actually use, so
everything in it works. The selected font file is read and handed to the page,
which uses that one file both to lay the text out *and* to build the geometry —
so what you see and what you get cannot drift apart.

**System `.ttc` font collections are supported.** The selected face is extracted
into a standalone font before it is sent to the browser. This includes fonts
such as Helvetica and Helvetica Neue on macOS.

Press **Refresh System Fonts** after installing a font; the scan is cached.

### Bold and italic require dedicated font files

CSS `font-weight` and `font-style` are **not synthesised**. If you use `<b>bold</b>` or `<i>italic</i>` in your text, the matching font variant must be present as a separate file in Fontfolder — for example `Roboto-Bold.ttf` alongside `Roboto-Regular.ttf`.

When a requested variant is absent, the pipeline falls back to the closest available weight in the same family. The geometry looks identical to the nearest loaded variant and no error is shown.

| What you want | What you need in Fontfolder |
|---|---|
| Regular | `MyFont-Regular.ttf` |
| Bold (`<b>` or `font-weight: 700`) | `MyFont-Bold.ttf` |
| Italic (`<i>`) | `MyFont-Italic.ttf` |
| Bold italic | `MyFont-BoldItalic.ttf` |

Font weight and style are read from each file's OS/2 table, so filenames do not need to follow any convention.

### Unsupported formats

For **Fontfolder**, use `.ttf` or `.otf` files. Collection extraction is supported
through **Use System Font**, not through folder-font loading.

## Multiple instances

Multiple `GeoTypeAdvanced` COMPs can run simultaneously, in one project or
across several TouchDesigner instances. Each claims its own port from the
9001–9099 range (**Autoport**, on by default) and then *verifies* it owns that
port before using it, so two instances cannot end up sharing one.

Each also carries its own identity, and the component ignores anything arriving
from a different instance. So even in the worst case the failure is "no data"
rather than one instance's text quietly appearing in another's geometry.

Copy-paste, drag-in and reopening a saved project all produce independent
instances.

## Mask-based Text Flow

With Shape Flow enabled and a positive Wrap Width, the input alpha mask marks
obstacles. The layout preserves separate islands and holes, unions obstacles
across the full text-row height, applies Shape Margin, and fits words into the
remaining intervals from left to right. It skips openings too narrow for the
next word. Words stay intact unless a word exceeds the entire wrap width;
those words can break into characters. A single oversized glyph flows below
the mask and may overflow the column rather than stalling layout.

Shape Side anchors the mask box to the left or right of the column. Text can
use free space on both sides of obstacles. Explicit newlines and `<br>` remain
line breaks; whitespace keeps its source indices, with spaces clipped at a
fragment's edge. Shape Max Resolution controls the raster approximation.

Shape Ease now interpolates glyph positions toward the solved layout. The
final layout avoids the mask; moving glyphs may cross it during a transition.
The mask preview uses a fixed layout origin even if its top rows block all text.

## Animation attributes

See the [illustrated POP attribute reference](pop-attributes.md) and
[animation notes](animation-attributes.md) for per-glyph animation and path placement.

[Back to the GTA documentation](README.md)
