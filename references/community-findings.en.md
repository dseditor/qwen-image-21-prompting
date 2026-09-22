# Qwen-Image 2.1 — What Actually Works

Claimed capabilities, checked. This file exists because the README and most community
posts describe what the model *can* do; the expensive knowledge is what it **cannot**.

Each entry says how it was measured so you can re-check it on your own build.

---

## ❌ 360° panorama — does not close

Prompting for a seamless equirectangular environment map produces something that
*looks* panoramic but whose left and right edges do not join.

**Method**: mean absolute difference between the leftmost and rightmost 8-pixel
strips, against a baseline of two randomly chosen 8-pixel strips from the same image.
A closed seam approaches 0.

```
seed A   seam 32.99   random baseline 48.62
seed B   seam 40.07   random baseline 48.56
```

An independent report measured 33.71 against a baseline of 70.53. Both agree: the
seam is barely more similar than two unrelated columns.

**Usable for**: wide establishing images, backdrops.
**Not usable for**: VR environment maps, anything that wraps.

## ⚠️ Frame-by-frame rotation as video — keyframes only

Generating a turnaround one frame at a time works, with caveats:

- **Angle numbers do not steer it.** A 24-frame sweep using
  `rotated N degrees clockwise` stayed frontal throughout; error against the first
  frame was uncorrelated with the stated angle.
- **Semantic viewpoint words do work.** `seen from directly behind` produced a correct
  eight-position turnaround, with near-perfect symmetry between mirrored views
  (12.32 vs 12.36).
- **Rotation speed is uneven** — a nominal 22.5° step snaps to the nearest semantic
  position, and frames carry slight scale and translation drift.
- **Autoregressive chaining degrades.** Feeding frame *i−1* back as the conditioning
  image for frame *i* stalled after about six frames; background brightness fell 42%
  and sharpness rose 163% as VAE losses accumulated.

**Treat it as a keyframe generator.** Hand in-between frames to an interpolator or a
video model.

## ✅ / ❌ Transparency — objects yes, text no

```
ordinary object   59.2% fully transparent, 0.5% partial  → clean matte, precise outline
text as text      alpha roughly inverted, glyphs collapse
text as sticker   alpha correct, but glyph quality still drops
```

Note that a saved PNG may be RGBA even when transparency was not requested — the
alpha channel is simply all-opaque. **Check the alpha distribution, not the file mode.**

## ✅ Text rendering — see `text-generation.en.md`

Summary: minimum glyph size matters; failures are systematic per-string rather than
random; description language and type style have no effect; and **no automated
verifier can check glyph correctness**.

## 📊 Performance characteristics

From an independent benchmark (single RTX 4090 48GB, bf16, diffusers). **Absolute
numbers will not transfer to a different precision or framework — the shape of the
relationships will.**

| Property | Finding |
|---|---|
| Step cost | Strictly linear. 24–30 steps is the useful range; 40 adds little over 24 |
| Resolution cost | Time scales as roughly pixels^1.18 |
| `true_cfg_scale` > 1 | Exactly 2× cost — it runs a second full forward pass. Default is 1.0 |
| Batching | No economy of scale: 1 / 2 / 4 images cost 10.9 / 22.2 / 44.2 s |
| Determinism | Same seed reproduces bit-for-bit |
| KV cache | Caches the text + conditioning-image prefix. Pure text-to-image gains ~2%. Each conditioning image adds ~2 GB; enable it for ≤4 reference images, disable beyond that |
| VAE tiling | Default tile is small; enlarging the tile with overlap cost ~1% time and saved ~8 GB |

## Untested

Independent masking, multi-image compositing beyond garment swap, three-view and grid
layouts, and the refusal behaviour of the bundled rewriter model.

---

*Findings dated 2026-09. Re-verify against your own build before relying on any of them —
especially anything marked ❌, since a later release may fix it.*
