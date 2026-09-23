# Composition and Optics — the layer the official spec does not have

The official rewriter puts you in one role: **an observer reporting what is in the
frame**. That role is why a well-formed prompt still produces a templated layout —
an observer reports positions, but never *decides* a grid, a hierarchy or a shared
alignment edge. Nothing in the eight steps asks you to make a design decision,
so the model falls back to the safest arrangement it knows: everything centred,
everything full-width.

This file is the layer that goes **before** Step 3. It changes nothing about the
official spec — because every design decision can be written as an observable
sentence, which is exactly what the spec wants.

```
decide  →  a twelve-column grid, text on one left edge, 4× type jump, 30% empty
write   →  "built on a twelve-column grid… a single left alignment edge one eighth
            in from the left carries every text block; nothing crosses it"
```

---

## 1. Layout: six decisions to make before writing

| Decision | What to settle | How it reads in the description |
|---|---|---|
| **Grid** | How many columns; which element spans how many | "built on a twelve-column grid", "spans eight columns and stops short of the right margin" |
| **Alignment edge** | How many left edges exist (one is strong, two is the limit) | "a single left alignment edge one eighth in from the left carries every text block; nothing crosses it" |
| **Hierarchy** | The size ratio between the first and second thing the eye meets | "about four times the height of any other text, so the eye lands there first" |
| **Negative space** | Which region is deliberately empty, and how big | "the left five columns stay empty; that is the largest quiet area in the frame" |
| **Reading path** | The order the eye travels | "reading top-left to centre-right to bottom-left" |
| **Weight** | Where mass sits, and whether it is symmetric | "asymmetric and top-weighted" |

**Do not use design jargon the model has to decode.** "Good visual hierarchy" and
"professional layout" are not observations. A ratio, a column count and an emptiness
percentage are.

### What this actually buys (measured 2026-09-23, Qwen-Image 2.1, seed fixed)

Same festival poster, identical information — headline, sub-line, four band names as
literal strings, dates, venue, price — generated at the same seed, size and step count.

```
without the layer   every block centred and full-width; a beige footer bar cuts the
                    poster in half; the date floats alone in the top-right; it reads
                    like a template
with the layer      three text groups genuinely share one left edge; the rule stops
                    at eight columns; the player moves right and crops at the knee;
                    the lower third stays open
```

⚠️ **Two honest limits from that run:**
- It obeys the *organisation*, not every *coordinate*. "The date sits directly above
  the venue line" was ignored — the date went to the bottom-right instead. **The
  design layer changes how the frame is organised; it does not place things to the
  pixel.** For exact placement, composite or run an edit pass.
- In that pair the design-layer prompt was 35% longer, so length is not fully
  controlled. An earlier pair matched for length (221 vs 224 words) showed the same
  effect, which is the evidence to trust.

## 2. Optics: decide the lens and the light, then describe the result

The spec welcomes photographic vocabulary but never says *which* to reach for. These
are the four decisions worth making explicitly — and note that in testing, **writing
the visible consequence worked better than naming the gear.**

| Decision | The two ends | Write the consequence, not the number |
|---|---|---|
| **Focal length** | wide, inside the space ↔ long, lifted out of it | wide: "the ceiling, the neighbouring tables and the depth of the room stay visible, and the lines of the room converge strongly behind her" · long: "the room behind her compressed into a narrow band of soft out-of-focus shapes… the window frame reads as a single soft vertical bar rather than a receding wall" |
| **Key light position** | 45° short key ↔ backlight ↔ flat frontal | Rembrandt: "a small inverted triangle of light sits on the cheek away from the window, and the shadow of her nose reaches toward the corner of her mouth without joining it" · backlit: "the window is the brightest thing in the picture and blows out to near white, so her figure reads mainly as a dark shape with a hard bright rim" |
| **Depth of field** | background carries information ↔ background is erased | "falls off quickly into smooth round bokeh" vs "stays clearly legible behind her" |
| **Shutter / texture** | frozen ↔ trailed; clean ↔ grain | "the rising steam is frozen mid-curl", "fine grain sits in the shadows" |

Both descriptions above came from one subject at one seed, and the pair came back
genuinely different: one intimate and flat-backed with round bokeh, the other a wide
contrasty room with hanging lamps, tiled floor and a blown-out window rimming the
subject. **Neither prompt contained a millimetre figure.**

📌 A number like "85mm" is a fact about equipment. The model was trained on pictures,
not on EXIF. Describe what the lens *did* to the frame.

## 3. This layer sets floors, not ceilings

Layout and optics fail differently from the rest of this skill: a badly composed image
throws no error, it is simply **boring**. Rules that guard against breakage should be
tight; rules that guard against dullness should be few, or they produce competence and
nothing else.

So use it as a floor — check these, and stop:

```
□ One reading path you can say out loud
□ A type jump of at least 3× between first and second level
□ One deliberately empty region, at least a quarter of the frame
□ Every text block on one or two alignment edges — not five
□ The light has a stated direction and one consequence you can point at
```

Five boxes. Do not grow this list: each rule added past the floor trades a possible
great image for a guaranteed mediocre one.
