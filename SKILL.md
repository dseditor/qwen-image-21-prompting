---
name: qwen-image-21
description: Write prompts for Qwen-Image 2.1 (text-to-image and image editing) following the official rewriter specification, and handle Chinese text rendering correctly. Use when generating or editing images with Qwen-Image, or preparing Qwen-Image prompts for a cloud platform, an API, or a local ComfyUI backend. This is the prompt layer — it does not cover model files, weights or runtime configuration.
---

# Qwen-Image 2.1 Prompting

## Scope

```
✅ here      turning a request into a prompt Qwen-Image 2.1 reads correctly,
             plus a suggested aspect ratio, plus Chinese text handling
❌ not here  model files, weights, quantization, steps, cfg, VRAM, how the backend runs
```

The output is **a prompt**. Paste it into a cloud platform, send it to an API, or
feed it to a local backend — this layer should not know which.

## How to use it

0. **Which of the four tasks is this?** They are not variations of one another.

   ```
   prompt only       deliver text and a ratio, submit nothing
   text-to-image     describe the finished, visible frame
   edit this image   lead with the operation, then one preservation clause
   from a reference  new frame, identity pointed at the image — not re-described in words
   ```

   The fourth collapses into the second if you are not watching for it.

1. **Is there an input image?** That decides everything else.
   - No → read `references/official_rewriter_t2i.txt` in full, follow it step by step.
   - Yes → read `references/official_rewriter_edit.txt` in full instead.

   These two files are the official rewriter system prompts, unmodified. They were
   written for a rewriter model; when there isn't one in the loop, **you are it**.
   The t2i spec is eight ordered steps and later steps never revise earlier ones —
   writing from memory collapses them into one.

2. **Read the matching guide** for the working rules and the traps:

| Guide | Covers |
|---|---|
| `prompt-writing.md` | The two specs in brief · spec layer vs platform layer · viewpoint wording · seed-first debugging · quantity-as-quota · transparency |
| `text-generation.md` | Minimum glyph size · fixed seed + word substitution · why no automated verifier can check glyphs · Traditional vs Simplified |
| `community-findings.md` | Claimed capabilities that were measured — including the ones that **don't** work |
| `delivery-and-verification.md` | Wiring checks before you submit · accepting the image afterwards · series splitting · A/B discipline |
| `composition-and-optics.md` | The design decisions the official spec never asks for: grid, alignment edge, type jump, negative space, reading path · lens and key-light decisions written as consequences |
| `rgba-and-sprites.md` | Native transparency for game and web assets · reading the alpha histogram correctly · what it replaces in a sprite pipeline and what it does not |

3. **If the image needs text**, do two things and no more:
   - State a minimum text size in the prompt, as a proportion of the frame.
   - Tell the caller: *if any text comes out garbled, say so and I will substitute
     the wording and regenerate.*

   Fixed seed plus a substituted word is the only text technique that reliably pays
   for itself. Anything heavier — cutting, scaling and compositing glyphs — costs
   more than switching to a tool with stronger text rendering.

## The one rule worth repeating here

**You cannot verify Chinese glyph correctness, and neither can OCR or a VLM.**
All of them resolve ambiguity with context, which is the same mechanism that hides
the error. A wrong character that fits the expected word reads as correct every time.
When text has to be right, a person must look at it — or it should be composited with
a real font.

## If you are also submitting it

A feature existing in Qwen-Image does not mean the endpoint you are calling exposes it.
Before submitting, check what actually reaches the backend — mode, reference numbering,
whether `width`/`height` are read at all, whether the seed survives the wrapper.
Afterwards, accept on the **unretouched original**: real pixel size, alpha distribution,
the untouched regions. A prepared request is not a generated image, and never report it
as one. Details in `references/delivery-and-verification.md`.
