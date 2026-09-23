# Delivering a Prompt, and Checking What Came Back

The rest of this skill stops when the prompt is written. This file covers the two
places where a correct prompt still produces a bad outcome: **the wiring between the
prompt and the backend**, and **the moment the image comes back**.

Much of the framing here is adapted from the `qwen-image-gen` skill in
[ZSeven-W/craft-skills](https://github.com/ZSeven-W/craft-skills/tree/main/skills/qwen-image-gen)
(MIT, © Fini.Yang), which works the delivery side of the same problem.

---

## 1. A capability in the model is not a capability in your wiring

The single most useful sentence to borrow: **the model having a feature does not mean
the thing you are talking to exposes it.** `wh_ratio` is a field in a rewriter's output
contract, not a parameter any given workbench understands.

Check the inputs that actually decide this run — not the documentation:

| Decision | What to actually look at |
|---|---|
| t2i or edit | The `mode` / pipeline / workflow in the request, never inferred from the text |
| Reference images | Which images the image node really receives, and how the numbering maps |
| Ratio and pixels | Whether `width`/`height` reach the sampler or latent node at all |
| Edit output size | Follows the reference, or takes an explicit size? Is there a fixed-area rescale? |
| Randomness | Is the seed fixed — or replaced by a wrapper? |
| Transparency | Real RGBA, or a checkerboard painted into RGB? |
| Result | The original-file download, true pixel size, job status, and recoverable settings |

Measured instance of exactly this: on one local setup, text-to-image produced
1152×2048 while the *same machine's* edit path produced 768×1376 and silently ignored
the submitted size. Two paths, one backend, different obedience. Check them separately.

> When an edit workflow ignores `width`/`height`, report the limit. Do not keep
> enlarging the input hoping it takes.

**Never say "generated" when you mean "request prepared".** A built request, a queued
job and a downloaded image are three different states, and only the third can be
checked. Credentials come from the user's existing configuration, are used only in the
request, and never appear in a prompt, a report or a repo.

## 2. One image, a series, and the frame are three separate decisions

A single image gets **one** clear action. When a brief describes several actions, split
it into separate complete prompts rather than appending "only draw one" — a multi-action
prompt with a suppression clause is exactly what produces a 3×3 contact sheet. This is
the action-layer twin of *quantity is a quota* in `text-generation.md`.

For a series of the same character, start **every** image from one approved identity
reference. Do not feed image *i−1* in as the reference for image *i*: the autoregressive
chain measurably degrades (`community-findings.md` — stalls after ~6 frames, background
brightness −42%, sharpness +163% as VAE losses compound).

The frame is an independent parameter. An explicit pixel size or ratio wins; otherwise
choose one from the composition and **push it into the downstream interface**. Saying
"vertical" in prose while submitting a square canvas is a wiring failure, not a prompt
failure.

## 3. Creating from a reference subject is its own task type

Four task types, not three:

```
prompt only      deliver text and a ratio, submit nothing
text-to-image    describe the finished, visible frame
edit this image  lead with the operation, then one preservation clause
from a reference create a new frame, with identity pointed at the image
```

The fourth is the one people collapse into the second. Point identity at the reference
image; do not re-describe the face in adjectives — verbal features make the model
regenerate the likeness. Only what genuinely must carry over (garment, accessory,
style) goes into the preservation clause; the pose, scene, camera and light are new.

A new scene may want a new ratio. Check whether the endpoint can change the frame
during a reference edit — if it can only follow the source, say so. *Suggesting* a new
ratio is not *generating* at one.

If you cannot read the text in a reference image, say so. Do not invent an OCR result.

## 4. Accept the image on the unretouched original

Look at the raw output, not a preview or a thumbnail: target text, the action, limb
connections, identity, the regions meant to be untouched, and the **real** pixel size.
For transparency, inspect the alpha distribution and the edges — a `.png` extension and
an RGBA mode prove nothing (an alpha channel that is uniformly opaque is still RGBA).

For every result keep: the original brief, the text actually submitted, the reference
images, the generation settings, and where the output landed.

**Record a defect and a dislike separately.** "Extra limb" is a defect; "I don't like
the seated pose" is a direction change. Merging them loses both — you can no longer
tell whether a fix worked.

When asking for a correction, state the defect, the target state, and what must stay.
Keep the original alongside the new version so the user can compare. When there is no
authorization for more generations, stop — do not re-roll indefinitely.

## 5. A/B comparisons have to hold everything else still

Fix the model, size, steps, seed and reference images; change one factor. An experiment
that also adopts a new frame is a different experiment — compare it on its own.

```
It generated          ≠  It looks better  ≠  It did what was asked
```

Three separate judgements. Most disappointing results pass the first, argue about the
second, and quietly fail the third.
