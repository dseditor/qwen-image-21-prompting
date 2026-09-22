# qwen-image-21-prompting

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
SKILL.md                     entry point — routing and the one rule worth repeating
references/
  official_rewriter_t2i.txt    official rewriter system prompt (text-to-image)
  official_rewriter_edit.txt   official rewriter system prompt (image editing)
  prompt-writing.*.md          the two specs in brief, plus measured prompting rules
  text-generation.*.md         rendering text, especially Chinese
  community-findings.*.md      claimed capabilities, measured — including what fails
```

Each guide ships in `en`, `zh-Hant` and `zh-Hans`.

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

`community-findings.*.md` records what was measured, when, and how — including
capabilities that are claimed but do not hold up (360° panoramas do not close at the
seam; transparency is clean for objects but degrades text). Re-verify against your own
build before relying on any of it, especially the negative results — a later release
may well fix them.

## Provenance

`references/official_rewriter_*.txt` are the official Qwen-Image 2.1 rewriter system
prompts, included unmodified. Everything else is original notes from hands-on testing,
plus one clearly-attributed third-party benchmark summarized in
`community-findings.*.md`.
