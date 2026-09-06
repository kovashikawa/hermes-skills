---
name: markdown-to-manim-video
description: Turn technical Markdown into verified Manim videos.
version: 0.1.0
author: Rafael Kovashikawa (kovashikawa), Hermes Agent
license: MIT
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Manim, Video, Markdown, DataViz, LaTeX]
---

# Markdown to Manim Video

Turn a technical Markdown document and its runnable code into a source-grounded Manim explainer. This is an AI-assisted production workflow, not a one-command converter: the agent writes scenes, renders drafts, reconciles every displayed claim against the source, and iterates from visual review.

## When to Use

- A technical article, notebook, or study needs an animated explainer.
- On-screen numbers and equations must stay synchronized with auditable source material.
- The video needs reproducible data visualizations, LaTeX, and frame-level review.
- Don't use for live-action editing, avatar video, or simple slide exports.

## Prerequisites

- Python managed with `uv`.
- Manim Community Edition.
- `ffmpeg` and `ffprobe` on `PATH`.
- A LaTeX distribution for `Tex` and `MathTex`.

Prefer a project-local environment:

```text
terminal(command="uv init && uv add manim numpy", workdir="<project>", timeout=300)
```

On macOS without administrator access, TinyTeX can be installed in the user library. Add its binary directory to `PATH` before rendering:

```text
export PATH="$HOME/Library/TinyTeX/bin/universal-darwin:$PATH"
```

Install Manim's common TinyTeX dependencies with `tlmgr`: `dvisvgm`, `dvipng`, `latex-bin`, `standalone`, `amsmath`, `amsfonts`, `cm`, `preview`, `xcolor`, and `babel-english`.

## Procedure

1. **Lock the source of truth.** Read the Markdown and any scripts, tables, or notebooks that produced its results. Record which values are source-derived and which are live sample statistics. Completion: every planned on-screen number has a named source.

2. **Define the visual grammar.** Extract the publication's design tokens or ask for a palette. Treat them as replaceable author-site tokens, not universal defaults. Define background, ink, muted text, rules, accent, warning, and fonts once at module level. Completion: all scenes import the same palette constants.

3. **Write the scene plan.** Give each scene one claim and an explicit transition. A reliable technical arc is: motivating failure, concrete examples, expanded benchmark, comparison reveal, compact summary, method details, limitations, closing rule. Completion: `plan.md` lists the scenes in final order and the claim each earns.

4. **Generate deterministic visuals.** Give every scene its own `np.random.default_rng(seed)`. Match the source's stated generating process exactly, including distributions and sample sizes. Plot deterministic functions as clean curves or lines; use scatter points for sampled or noisy data. Completion: rerendering produces the same points and displayed statistics.

5. **Typeset math with LaTeX.** Use `MathTex` for equations and mixed text/math expressions. Use `Tex` only with explicit math delimiters where needed. Put equations above charts rather than across axes or data. Avoid redundant labels when the equation already names the relationship. Completion: no raw LaTeX, tofu glyphs, or text-axis collisions in a draft frame.

6. **Design comparisons as staged reveals.** Register all panels of a small-multiples grid together, then animate comparable layers together. For a classical-versus-modern comparison, reveal the classical statistic first, pause, then grow the modern statistics onto the same persistent grid. Do not fade out and rebuild identical panels. Completion: the contrast lands as a sequence without spoiling the later measure before its reveal.

7. **Render drafts scene by scene.** Use low quality for iteration:

```text
terminal(command="uv run manim -ql script.py Scene1 Scene2", workdir="<project>", timeout=600)
```

Extract frames with `ffmpeg` at meaningful fractions of each scene. Do not use only the last frame because a closing `FadeOut` can make it blank. Completion: every scene has at least one review still at its fully populated state.

8. **Review geometry and claims.** Use `vision_analyze` for overlap, clipping, hierarchy, and legibility. Verify tiny numbers programmatically against the Markdown's table or generated output because vision models misread digits. Compute content bounding boxes when centering matters. Completion: no text touches plots, frames, neighboring panels, or screen edges; every displayed value matches its source.

9. **Run an independent reviewer pass.** Give the reviewer fresh keyframes from the current render, `script.py`, `plan.md`, and the exact Markdown source. Ask for numbered findings with severity, locations, fixes, and a publish verdict. Treat reviewer claims as hypotheses: reproduce each against current files before editing. Completion: every real Major/Critical finding is fixed and rechecked; stale-keyframe findings are documented and dismissed with evidence.

10. **Render and stitch the final.** Render only changed scenes at high quality, maintain a `concat.txt` in the intended order, and stitch without re-encoding when scene codecs match:

```text
terminal(command="uv run manim -qh script.py Scene1 Scene2", workdir="<project>", timeout=600)
terminal(command="ffmpeg -y -f concat -safe 0 -i concat.txt -c copy final.mp4", workdir="<project>", timeout=300)
```

Completion: `ffprobe` confirms resolution, frame rate, duration, and expected scene count.

11. **Verify the deliverable, not the intermediates.** Re-extract the user-flagged timestamps from `final.mp4`. Confirm the current source hash or commit still matches the document used for the render. Completion: checks are against the stitched file and latest source, not cached scene output.

## Visual Rules

- Keep prose short; equations carry mathematical definitions.
- Omit sentence-ending periods in compact on-screen labels when they read as stray marks.
- Frame dense small multiples with subtle rules so panels do not visually merge.
- Use one composed stats line per mini-panel when vertical space is tight.
- Keep captions at least 26px at 1080p when using thin LaTeX or muted colors.
- Center tables by the rendered content block, not hardcoded column coordinates.
- A full table belongs after the visual comparison as the compact reference, not before the audience understands it.

## Pitfalls

- **Stale stitched scene:** a successful render does not update the final MP4. Re-stitch and extract the flagged timestamp from the final file.
- **Shared RNG:** a module-level generator bleeds state across scenes. Instantiate it inside each scene.
- **Vision digit errors:** screenshot review is not numerical verification. Parse and compare values in code.
- **Reviewer stale context:** regenerate keyframes after every structural change and verify findings against current source.
- **`Tex` math mode:** `Tex` does not wrap raw commands in math mode; use `MathTex` or explicit `$...$` segments.
- **TinyTeX missing files:** read `media/Tex/<hash>.log`; `preview.sty` and `babel-english` are common missing packages.
- **Axis-title collision:** `next_to(ax, UP)` ignores separate axis-label objects. Include the label bounds or use a larger buffer and inspect the render.
- **Dense tables:** showing every cell may require a staged comparison first. Do not claim a condensed table is the full benchmark.

## Sources and Attribution

- Production approach inspired by the 3Blue1Brown videos repository (https://github.com/3b1b/videos): scene-based Manim production, draft iteration, keyframe review. No code copied.
- That repository is CC BY-NC-SA 4.0. This skill is an original written workflow, not adapted code, so ShareAlike does not attach to it. Keep it that way: describe techniques, never paste their code.
- Runs on Manim Community Edition (MIT). Name it, do not claim it.

## Verification Checklist

- [ ] Latest Markdown and result-generating code read before final render
- [ ] Every number tied to source output or identified as live sample output
- [ ] Deterministic seed and generator documented
- [ ] Equations render in LaTeX and clear axes/data
- [ ] Small multiples register and animate as coherent comparison layers
- [ ] No overlap, clipping, stale labels, or broken glyphs in review frames
- [ ] Independent reviewer findings reproduced before fixes
- [ ] Final stitched MP4 checked at user-flagged timestamps
- [ ] `ffprobe` confirms resolution, frame rate, duration, and stream integrity
- [ ] Public copy contains no personal paths, credentials, or private project details

[[as_document]]
