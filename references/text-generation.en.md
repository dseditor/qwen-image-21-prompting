# Rendering Text with Qwen-Image 2.1

## The short version

1. **State a minimum glyph size in the prompt.** Below it, characters collapse.
2. **If the output is garbled, re-roll with a substituted word — same seed, different
   wording.** This is the only technique that reliably pays for itself.
3. **A human has to look.** Neither the model nor an LLM/OCR/VLM can verify glyphs.

Everything else in this file is detail behind those three.

## Minimum glyph size

Community testing converges on:

```
Simplified Chinese   ≥ 20 px of ink
Traditional Chinese  ≥ 50 px of ink
```

Put it in the prompt as a proportion of the frame rather than a pixel count — the
model has no notion of your output resolution. "Filling about eighty percent of the
frame width" is actionable; "50px" is not.

## Fixed seed, substituted word

Failures are **systematic, not random**: a bad string reproduces across seeds,
description languages and type styles. Re-rolling the same string is wasted time.

```
generate → a human reads it → swap only the word that broke → regenerate
```

**Do not rewrite the whole line.** One bad word costs one substitution.
`處理中` failed 9 out of 9 attempts; `作業中`, `進行中`, `執行中`, `運作中` all
worked on the first try. The fix cost one word, not a redesign.

For a caller-facing tool, the useful offer is simply:

> *If any text comes out garbled or malformed, tell me and I will substitute the
> wording and regenerate.*

## What does NOT affect glyph correctness

Measured, with controls:

| Dimension | Result |
|---|---|
| Description language (Traditional / Simplified / English prose) | No effect — identical outcomes. Pixel difference between Traditional and Simplified prompts was **18.9%** of the difference caused by changing the seed |
| Type style | Same word rendered in bold sans, extruded signboard, neon tube, calligraphic brush, floral and cartoon lettering — **all six correct** |
| Glyph size, once above threshold | No effect |

**So the risk lives in the string itself.** A word verified once can be reused across
styles, prompt languages and sizes — which makes a list of verified strings worth
keeping.

## Traditional Chinese is less reliable — not "always wrong"

A common overstatement. In a controlled test, four high-frequency words with distinct
Traditional forms (開發 / 學習 / 體育 / 鐵路) all rendered correctly; only strings
containing 處 failed.

```
✓  Traditional is measurably less reliable than Simplified
✗  "Traditional always fails"
```

Failures substitute a *plausible neighbour* rather than producing noise — 處 came out
as 虛 (shares the 虍 component) and, in other reports, as the Japanese form 処. A
consistent substitution across seeds is the signature of a systematic pull, not a
rendering wobble.

## 🚨 You cannot verify glyphs — a human must

This is the most important rule in this file, and it is easy to violate without
noticing.

```
Eye-balling it        reads the radical, assumes the character  → wrong
OCR / VLM             uses context, reads the intended word     → wrong
Pixel similarity      scored the wrong character higher         → wrong
A literate human      "that's the wrong character"              → right
```

**The error source and the verifier are the same thing: a context prior.** Any reader
that uses context to resolve ambiguity is structurally blind to this failure, and a
finer metric does not help — the metric shares the failure mode.

Route accordingly:

| The text is | Do this |
|---|---|
| Required to be correct (signage, titles, product names, teaching material) | Composite it with a real font, or use a model with stronger text rendering |
| Atmosphere only (distant signs, background lettering) | Let the model generate it and accept occasional errors |
| In between | Generate, then **have a person look**, and say plainly that you did not verify it |

Being honest about this is cheaper than the alternative: an elaborate pipeline that
cuts, scales and composites glyphs is slower than simply using a tool that gets text
right the first time.

## Quantity is a quota

Naming a count creates slots that will be filled whether or not you supplied content.
Twelve unspecified label rows come back as twelve rows of gibberish. Enumerate every
string that should appear, and add an explicit lock that nothing else may be rendered.

## Transparency does not help text

RGBA output is clean for ordinary objects, but asking for transparency degrades glyph
quality. Describing the text as a die-cut sticker fixes the alpha matte but the letter
forms still come out worse than an opaque render.
