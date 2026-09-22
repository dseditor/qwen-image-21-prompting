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
| `prompt-writing.*.md` | The two specs in brief · spec layer vs platform layer · viewpoint wording · seed-first debugging · quantity-as-quota · transparency |
| `text-generation.*.md` | Minimum glyph size · fixed seed + word substitution · why no automated verifier can check glyphs · Traditional vs Simplified |
| `community-findings.*.md` | Claimed capabilities that were measured — including the ones that **don't** work |

   Available in `en`, `zh-Hant` and `zh-Hans`.

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
