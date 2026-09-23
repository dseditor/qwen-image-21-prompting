# Prompt Writing for Qwen-Image 2.1

Qwen-Image 2.1 ships with two official **rewriter system prompts** — one for
text-to-image, one for editing. They were written for a rewriter model. When there
is no rewriter model in the loop, **you are the rewriter**: read the matching file
in full and follow it step by step.

```
no input image  →  official_rewriter_t2i.txt   → {rewritten_prompt, wh_ratio}
has input image →  official_rewriter_edit.txt  → + ratio_follow
```

The t2i spec is **eight ordered steps, and later steps never revise earlier ones**.
Writing from memory collapses eight steps into one — which is exactly what it guards
against.

## The two specs pull in opposite directions

**t2i — you are an observer describing a finished image.**
Present tense, third person, declarative. Roughly twenty sentences and 400–500 words.
Eight to fourteen positional phrases that reach the corners, not just the centre.
Lighting gets its own sentence. Exactly one closing sentence. Skip the text step
entirely when nothing in the image is meant to be read.

> A three-word brief and a three-hundred-word brief produce descriptions of **the
> same size**. A thin brief means you are inventing more of the frame, not writing less.

**edit — you are instructing someone holding the input image.**
Lead with the operation, not with a description of the finished picture.

- **Attribute disentanglement at full strength.** Change exactly what was named,
  push it to an unmistakable degree, hold everything else at input fidelity.
  Two symmetric failures: *leakage* (touching what was not named) and *under-editing*
  (an output that could be mistaken for the input).
- **Preservation locks content, never edit strength.** Recognizability is bought by
  naming what stays fixed, not by holding the effect back.
- **Say what stays without repainting it.** Name untargeted content by type, position
  and role. A preservation description reads to the model as a generation instruction —
  the more concretely you describe something you meant to keep, the more it drifts.
- **Identity: point at the reference image, do not describe features in words.**
  Verbal descriptions make the model regenerate the face and degrade the likeness.
- Multi-image input uses `<image1>`, `<image2>`; single-image input uses none.
  State each image's role — which is the canvas, which supplies material.

### Self-check for edit prompts

*For every element I asked to preserve — did I also remove its cause?*

Asking for "make the window a rainy night" **and** "keep the lighting unchanged" is
physically contradictory: the rim light came from the daylight you just removed. The
model will silently pick one side, and it has no obligation to pick yours.

## Spec layer vs platform layer

`wh_ratio` and `ratio_follow` are **fields the hosting platform reads**. The mapping
table in the official spec (business card 9:5, A4 5:7, phone screen 18:39 …) exists so
*you* can choose a ratio — it is not read by the model.

```
Never put resolution, pixel counts, "4K" or "business card" into rewritten_prompt.
```

On a local backend (ComfyUI and similar) the output size comes from the latent you
allocate, so those fields do nothing unless your own script consumes them.

> **When adopting someone else's spec, first separate the half that is coupled to
> their platform.**

## Three rules that come from measurement

**Use semantic viewpoint words, not angle numbers.**
`seen from directly behind` works. `rotated 180 degrees` does not — in a 24-frame
sweep every frame stayed frontal and the error was uncorrelated with the stated angle.

**When an instruction gets no response with a conditioning image, change the seed
before you change the prompt.** In a controlled diagnostic the same instruction
succeeded on all four viewpoints after a seed change; the failing seed happened to be
the one that generated the anchor image.

**Any quantity you state is executed as a quota.** Asking for "three to five bullet
points" and supplying two leaves a gap that gets filled from the training
distribution — as garbled text. Enumerate every text element explicitly and add a
lock: *no text may appear other than the listed items.* The official spec says the
same thing from the other side: **text you cannot commit to should not be added at all.**

## Transparency

```
open with   This is an RGBA image with transparency.
close with  The image has alpha channel and the background is transparent.
```

Clean for ordinary objects (measured: 59.2% fully transparent, only 0.5% partial).
**Not for text** — glyph quality drops noticeably. See `text-generation.md`.
