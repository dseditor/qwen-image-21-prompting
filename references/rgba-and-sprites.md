# Native RGBA, Game Assets and Sprite Sheets

Qwen-Image 2.1 outputs a real alpha channel, which removes an entire stage from the
usual game-asset pipeline (generate on a chroma-key background → run a segmentation
model → check it did not over-cut). This file is what that replaces, what it does
**not** replace, and the one line of post-processing it still needs.

All figures measured 2026-09-23 on a local build, 1024×1024, 25 steps, seed 8888.

---

## 1. Turning it on, and reading the result correctly

Open and close the description with the two transparency sentences (see
`prompt-writing.md`). Then **read the alpha distribution, never `im.mode`** — ordinary
output is an RGBA container too, with alpha uniformly 255.

🚨 **A naive histogram will make a clean cutout look broken.** Across six assets,
46–58% of pixels came back as "partially transparent" — which sounds catastrophic and
is not:

```
alpha == 0        34–42%     ← the empty field
alpha 1–31        45–55%     ← ALSO the empty field, just not zeroed
alpha 32–223      under 1%   ← the actual soft edge
alpha == 255      4–17%      ← the subject
```

In the empty corners the mean alpha is **0.6** (max 4). Composited onto pure black the
error is **+0.3 / 255**; onto pure white, −0.3. It is invisible. The bimodal shape —
a spike near 0 and a spike at 255 with nearly nothing between — is what a *good* matte
looks like.

✅ **The fix is one line**, and it is worth doing before handing assets to an engine
that premultiplies alpha or tests `alpha > 0`:

```python
alpha[alpha < 32] = 0     # subject pixels are untouched: verified identical count
```

After thresholding, the fully-transparent share goes 34.5% → **81.5%**, versus **82.6%**
for the same character generated on a magenta field and cut with RMBG-2.0. The two
routes land in the same place.

## 2. What it replaces outright

Same character, same seed, same model — one generated with native transparency, one
generated on flat magenta and segmented with RMBG-2.0. Silhouettes are equivalent down
to the hair strands; the native route simply skips the segmentation step and, with it,
**both of that step's known failure modes**: over-cutting into a hollow outline, and
chroma spill along the edge.

```
✅ single props          sword, potion, coin — precise outline, no halo
✅ UI icons in one sheet  four icons, fully separated, no icon touching another
✅ character stand-ins    full body, clean hair edges, nothing touching the frame edge
✅ UI buttons WITH text   "START" on a teal button rendered correctly
```

🔑 **Why the button's text survived, when `community-findings.md` says transparency
wrecks text:** the glyphs sat on an **opaque button body**. The alpha boundary ran
around the button, not around the letters. The old finding still holds for its own
case — text as the subject of the cutout — and the distinction is the useful part:

```
text ON an opaque object   → alpha never touches the glyph edges   → fine
text AS the cut-out object → alpha traces every stroke             → glyphs collapse
```

## 3. What it does not replace

A sprite **sheet** is not a transparent image; it is a set of frames that must agree
with each other. Native alpha solves neither remaining problem.

**Cell geometry — close, but sprites need exact.** Asking for an explicit two-by-two
cell structure did measurably improve it (frame-height spread 4.4% → 2.2%, and the
figure fills the cell instead of floating in it), but the feet landed 5, 10, 14 and
18 px from their own cell floors. **A 13 px vertical wobble is visible as bounce in a
walk cycle.** Snapping frames to a shared baseline and scale is still a post-process —
`generate2dsprite.py process --threshold 1` and its edge-touch QC do exactly that, and
that part of the old pipeline stays.

**Frame-to-frame identity — this is the real ceiling.** Across four cells of one sheet
the costume drifted: sleeve shape, the number of frill layers, hair volume. Played back
it reads as flicker. **One text-to-image call cannot hold a character identical across
frames**, because each cell is generated in the same pass with no constraint tying them
together.

→ Two ways out, and **the second one wins for continuous action**:

**(a) Edit route.** Generate one approved frame, then produce the rest through the
edit/reference route with that frame as the identity source — never chain frame *i−1*
into frame *i* (`community-findings.md`: stalls at ~6 frames, background brightness
−42%, sharpness +163%).

**(b) 🏆 Take the frames from a video model instead.** A swing, a walk, a cast — these
are *motion*, and a video model holds identity across frames by construction. Measured
2026-09-23 with MiniMax H3 text-to-video, 480×480, 124 frames, 20 steps:

```
47 seconds          124 usable frames of an 8-bit warrior swinging a sword twice
foot line           standard deviation 0.0 px across the extracted cells
                    (versus 13 px from a four-cell Qwen sheet)
identity            costume, palette, proportions hold for all 124 frames;
                    background corner colour drifts by 0.3 / 255
style               chunky 8-bit pixel look renders fine — it is a style, not a format
```

📌 **Why the alignment is free:** every frame comes from one locked-off shot, so they
share a coordinate system already. Qwen's four cells have to be *made* to agree;
H3's frames never disagreed in the first place.

**Three things that route needs, and a skill should say all three:**
1. **Save a PNG sequence, not an mp4.** h264 chroma subsampling (4:2:0) smears the
   exact colour you are about to key out. Cut `CreateVideo`/`SaveVideo`, put
   `SaveImage` straight after `VAEDecode`.
2. **Key + despill, because there is no alpha.** Background keys at a measured colour
   (here 228,4,143 — *not* pure magenta; sample it, do not assume it). Soft mask on
   colour distance, then pull R and B back toward G on partial pixels: 26% of edge
   pixels carried spill before despill.
3. **Find the cycle from the frames, not by eye.** The top-most foreground row per
   frame is a clean motion signal — the two swings showed up as dips from y=196 to
   y=19 around frames 24–48 and 76–92. Sample the cells from one dip.

⚠️ Costs of the video route: the frame count is fixed by the `17n+5` rule (124 is the
floor, ≈5 s), you pay for frames you throw away, and **the pixel grid is not a real
lattice** — it is pixel-*look*, so down-sampling to a true 32×32 still needs its own
step. Also state in the prompt that the whole body plus margin stays inside the frame:
this test had the boots landing on the bottom row, which is exactly the edge-touch
failure `generate2dsprite`'s QC exists to catch.

**Pixel art is a style, not a resolution.** The model draws chunky pixel-look shapes
with clean stepped alpha, but it is not producing a true 32×32 grid; nothing snaps to a
pixel lattice. Down-sampling to a real sprite resolution remains a separate step.

## 3.5 The RPG case: one reference image, one clip, every direction

Measured 2026-09-23. Reference image drawn by **codex** (transparent 8-bit chibi), composited
onto magenta by hand because H3 does not read alpha, then fed to H3 **ref2v** with four shots
in one prompt: walk facing the viewer → facing right → facing away → facing left.

```
78 seconds          209 frames covering all four directions
foot line           standard deviation 0.9 px across 16 cells spanning FOUR directions
identity            dress, star clips, gloves, boots hold through every turn
edge touch          none
```

📌 **Four shots in one clip beats four separate clips**: one shot means one coordinate
system and one identity draw. Separate clips have to be *made* to agree afterwards.

🚨 **But ref2v does not hold the background the way t2v does.** This is the one real
defect found, and it is invisible until you try to key it:

```
pure t2v      background corner colour drifts   0.3 / 255   ← rock solid
ref2v         background corner colour drifts  42.6 / 255   ← 78% of frames turned
                                                              neutral grey instead of magenta
per-frame     corner spread WITHIN a frame      1.1         ← each frame is still flat
```

The retention clause protects the *subject*; the background rides along and wanders.

✅ **Fix: sample the key colour per frame, not once for the clip.** Each frame is
internally flat, so its own corner median is a valid key. With that, the four-direction
sheet keys cleanly. The handful of frames that *did* stay magenta still need despill —
skip it and those cells come out with a violet fringe (visible in this test's run).

📌 **Generalised: a "constant" that holds in one mode can wander in another.** Measure the
thing you are about to rely on, in the mode you will actually run.

## 3.6 Where each generator sits (2026-09-23)

```
codex          finest art, real RGBA (corner alpha 0, no edge touch) — the identity source
               ⚠️ subject alpha averages 252, not 255: run alpha[alpha>200]=255 before
               handing it to an engine that multiplies alpha
H3             motion and cross-frame identity — the only one that gives continuous action
Qwen 2.1       local, free, ~6 s, native alpha; an occasional local route rather than the
               default, until community tuning closes the quality gap with codex
```

## 4. Deciding the route

```
one asset, any style, needs transparency      → native RGBA, threshold, done
a set of unrelated icons on one sheet         → native RGBA, then split on empty space
a character used once (portrait, web hero)    → native RGBA, done
an animation set / walk cycle / any motion    → video model (H3 t2v), magenta field,
                                                 PNG sequence, key + despill, pick
                                                 cells from one motion cycle
a named character in many directions／poses   → codex draws the reference, H3 ref2v
                                                 walks it through four shots in ONE clip,
                                                 then key PER FRAME (background wanders)
the same pose in several unrelated outfits    → one approved frame, then edit-route
true pixel-art at a fixed lattice             → native RGBA for the art, then downscale
text as the transparent object itself         → do not; composite with a real font
```

📌 One line to keep: **native alpha removed the cut-out stage, not the consistency
stage.** The hard part of sprite work was never the background.
