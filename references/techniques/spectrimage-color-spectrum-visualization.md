# Spectrimage — Generating a Color Spectrum for an Image

A visualization technique (and tool) for showing the *color composition* of an image as a
2D spectrum: hue along the horizontal axis, lightness stacked vertically, frequency as
column height. Distinct from palette extraction — instead of reducing an image to N swatches,
it shows the full distribution of hue × lightness × frequency in one chart.

- **Author:** Amanda Hinton
- **Published:** 2026-04-14 (tags: color, code, creative)
- **Built at:** Recurse Center "Impossible Stuff Day"
- **Source:** <https://amandahinton.com/blog/generating-a-color-spectrum-for-an-image>
- **Contact:** hello@amandahinton.com

## The Core Idea (Iteration 7 — the breakthrough)

The hard part of "spectrum from an image" is that color is 3D (hue, lightness, chroma) but
a spectrum strip is 1D. Early attempts all tried to **sort** pixels into a single line
(median-cut quantization, hue histograms, pixel-level sorting, lightness sub-sorts) and all
produced banding, striping, or discontinuities — the one-dimensional sorting problem.

The breakthrough was to go 2D: **each hue gets a vertical column, painted as a linear
gradient — tint at top, pure hue at center, shade at bottom.** This displays hue, frequency,
and tonal range simultaneously instead of forcing everything onto one axis.

Within a hue column, the tonal endpoints are derived by slicing that hue's pixels by lightness:

> "The darkest 20% are averaged to produce the shade color. The middle 20% produce the pure
> color. The lightest 20% produce the tint. Using 20% slices reduces outlier noise."

## Reading the Final Spectrum

| Visual property        | Encodes                                              |
| ---------------------- | ---------------------------------------------------- |
| Horizontal position    | Hue (walks the wheel)                                |
| Column height          | Frequency, relative to the most-populous hue         |
| Stacked bands (top→bot)| Lightness distribution: tints up, shades down        |
| Asymmetry about center | Tonal character — dark-dominant vs light-dominant    |

The pure-hue band straddles the horizontal axis; tints stack upward, shades downward, so a
column that bulges upward is a light-leaning hue and one that bulges down is dark-leaning.

## OKLCH Refactor (Iteration 11)

The tool started in HSL and was rebuilt in **OKLCH**, which fixed saturation-perception and
perceptual-uniformity problems:

> "In OKLCH, L is lightness that matches the eye. C is chroma, independent of lightness, so a
> near-white off-white reports near-zero. And H is hue that walks the wheel evenly."

This is the standard argument for OKLCH over HSL: HSL's L is a math average (so equal-L colors
look unequally bright), its S doesn't track perceived saturation, and its hue spacing is
non-uniform. OKLCH's chroma being lightness-independent is what makes the chromatic/achromatic
split (below) clean — an off-white reports near-zero chroma instead of HSL's misleadingly high
saturation.

## Binning Parameters (final implementation)

| Parameter              | Value                                              |
| ---------------------- | -------------------------------------------------- |
| Color space            | OKLCH                                               |
| Chromatic threshold    | chroma **C ≥ 0.02** (below → treated as achromatic)|
| Hue bins               | **180 bins, 2° each**, wrap point at 15°            |
| Achromatic bins        | **60 lightness bins** (greys handled separately)   |
| Lightness bands/column | **11 bands**, ~0.091 L units each; band 5 = pure   |

Black-and-white / near-grey images get special handling: pixels under the chroma threshold are
routed to a separate achromatic (lightness-only) track instead of being forced into a hue column.

## Performance / Architecture

- **Fully client-side** — runs in the browser, no server beyond the Canvas API.
- Images **downsampled to 300px on the longest dimension** before binning.
- **Under 1 second** to process a 4000×3000 photo.

## Why It Matters (technique takeaways)

- **Don't sort 3D color onto 1D.** Give one perceptual axis to position (hue → x) and let the
  others map to height/stacking. Generalizable to any "summarize a color field" problem.
- **Percentile slices (20%) beat min/max** for deriving tint/pure/shade — they reject outlier
  pixels (specular highlights, JPEG noise) that a true-min or true-max would latch onto.
- **A chroma threshold is the right achromatic gate** — in OKLCH, near-greys honestly report
  near-zero chroma, so `C ≥ 0.02` cleanly separates "has a hue" from "is grey." This is exactly
  the kind of thing HSL gets wrong (a dark saturated-looking blue and a near-grey can both show
  high S).
- **Frequency-as-height + asymmetry-as-tone** packs three dimensions into a static chart that's
  readable at a glance.

## Related

- See [Image Color Extraction Tools](image-color-extraction-tools.md) for the *palette
  extraction* counterpart (img-colors.com, okpalette.color.pizza, colorgram-js, Art Palette) —
  those reduce to N swatches; Spectrimage instead shows the full distribution.
- OKLCH rationale: [HSLuv better than HSL](hsluv-better-than-hsl.md), [Culori](culori-color-spaces-api.md).

## Links

- **Article:** <https://amandahinton.com/blog/generating-a-color-spectrum-for-an-image>
- **Author site:** <https://amandahinton.com>
- **Recurse Center:** <https://www.recurse.com>
