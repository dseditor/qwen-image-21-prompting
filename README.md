# qwen-image-21-prompting

[English](README.md) | [繁體中文](README.zh-Hant.md) | [简体中文](README.zh-Hans.md)

A skill for writing prompts for **Qwen-Image 2.1** — text-to-image and image editing —
following the official rewriter specification, with practical handling for Chinese text
rendering.

This is the **prompt layer**. It produces a prompt and a suggested aspect ratio.
Paste that into a cloud platform, send it to an API, or feed it to a local backend —
the skill does not care which, and it deliberately says nothing about model files,
weights, quantization or runtime settings.

## Install

Drop the folder into your skills directory:

```
~/.claude/skills/qwen-image-21/
```

## What's inside

```
SKILL.md                          entry point — routing and the one rule worth repeating
references/
  official_rewriter_t2i.txt         official rewriter system prompt (text-to-image)
  official_rewriter_edit.txt        official rewriter system prompt (image editing)
  prompt-writing.md                 the two specs in brief, plus measured prompting rules
  text-generation.md                rendering text, especially Chinese
  community-findings.md             claimed capabilities, measured — including what fails
  delivery-and-verification.md      wiring checks, accepting the output, A/B discipline
```

**The guides are English-only, on purpose.** They used to ship in three languages, and
three copies of a finding drift apart the moment one of them is corrected — silently,
because nobody diffs a translation. This README is translated instead: it is the part a
human reads once, while the guides are the part an agent reads every run.

## The short version

**Pick a spec by one question — is there an input image?**
No input image → the text-to-image rewriter. Input image → the editing rewriter.
Read the matching file in full and follow it in order; the spec is a sequence of
steps where later steps never revise earlier ones.

**For text in the image, do two things.** State a minimum text size as a proportion
of the frame, and offer to substitute wording if anything comes out garbled. Fixed
seed plus a substituted word is the only text technique that reliably pays for itself.

**Nobody automated can check Chinese glyphs.** An LLM, OCR and a VLM all resolve
ambiguity using context — which is exactly the mechanism that hides a wrong character
that happens to fit the expected word. When text has to be correct, a person must
look at it, or it should be composited with a real font.

## Findings are dated

`community-findings.md` records what was measured, when, and how — including
capabilities that are claimed but do not hold up (360° panoramas do not close at the
seam; transparency is clean for objects but degrades text). Re-verify against your own
build before relying on any of it, especially the negative results — a later release
may well fix them.

## The prompt is not the whole job

`delivery-and-verification.md` covers what happens on either side of the prompt: a
feature existing in the model does not mean your endpoint exposes it (one local setup
produced 1152×2048 from text-to-image while its own edit path silently ignored the
submitted size), and an image has to be accepted on the unretouched original rather
than a preview. A prepared request is not a generated image.

## Provenance

`references/official_rewriter_*.txt` are the official Qwen-Image 2.1 rewriter system
prompts, included unmodified. Everything else is original notes from hands-on testing,
plus one clearly-attributed third-party benchmark summarized in
`community-findings.md`.

`delivery-and-verification.md` adapts the delivery-side framing of the
[`qwen-image-gen`](https://github.com/ZSeven-W/craft-skills/tree/main/skills/qwen-image-gen)
skill from Craft Skills (MIT, © Fini.Yang). The measurements in it are our own.
