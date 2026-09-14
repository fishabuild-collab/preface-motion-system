# Preface — motion system

How we make films: the visual system, the way of working, and a skill file that
teaches an AI agent to apply both.

Documentation. No footage, no client material, no licensed typefaces.

---

## Start here

| If you are… | Read |
|---|---|
| New to any of this | [`docs/remotion-explainer.md`](docs/remotion-explainer.md) |
| Designing or reviewing a film | [`docs/preface-brand.md`](docs/preface-brand.md) |
| Actually building one | [`docs/remotion-build-runbook.md`](docs/remotion-build-runbook.md) |
| Setting up an AI agent to help | [`skills/preface-video/SKILL.md`](skills/preface-video/SKILL.md) |
| After the logo or the type spec | [`brand/`](brand/) |

### [The explainer](docs/remotion-explainer.md)
What Remotion is (React that renders to video), where it sits next to
Premiere / DaVinci / After Effects, the pipeline we run end to end, the four
practices that make the difference, tips, and a do/don't list.

### [The brand guide](docs/preface-brand.md)
Palette, type scale, motion spec, formats, photo registers, and the honesty
rules. Five rules carry the rest:

1. Two hues, both semantic
2. Three type sizes, two weights
3. Things settle, never bounce
4. One focus point, and the movement points at it
5. Never claim what isn't true

### [The build runbook](docs/remotion-build-runbook.md)
Scaffold to render, from an empty folder. Commands, fonts, browser gotchas.

### [The skill](skills/preface-video/SKILL.md)
Drop-in instructions for a coding agent. On Claude Code, copy the folder to
`.claude/skills/preface-video/` in your project and it loads itself when the
work calls for it. On another platform, load all three documents as system
instructions — and make sure the agent can read files, run a shell, and **look
at rendered stills**. Without that loop, any model is animating blind.

### [Brand assets](brand/)
The Preface wordmark, and which weights of Galano Grotesque the films use.
**The font files are not here** — Galano is a commercial typeface licensed to
Preface, so the binaries stay out of a public repo. Preface staff get them from
the studio; everyone else should license their own seat.

---

## Getting the files

Download the whole thing as a zip: green **Code** button → **Download ZIP**.
Or, if you have git:

```bash
git clone https://github.com/fishabuild-collab/preface-motion-system.git
```

Single file: open it, click **Raw**, then save.

---

## Deliberately not in here

- **Font binaries** — licensed, see [`brand/`](brand/)
- Client photography, VO and renders
- Project-specific handoffs and any unpublished campaign figures
- Andale Mono — it ships with macOS

## Using this

The writing is shared so others can borrow the method — take the ideas, the
easing values, the workflow. The Preface name, wordmark and the visual system
as a whole are ours; please don't present them as your own.

## Keeping it honest

If a value here and the code in the film project disagree, **the code wins** —
then fix the document. Tokens live in `src/brand/light.ts`,
`src/brand/motion.ts` and `src/loops/tokens.ts` in the film repo.
