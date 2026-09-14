# Brand assets

## Typeface — not in this repo

The films are set in **Galano Grotesque**, using **400 (Regular)** and
**600 (SemiBold)** only. Nothing heavier.

> The font files are **not here, and must never be committed.** Galano
> Grotesque is a commercial typeface licensed to Preface; this repo is public,
> and publishing the binaries would be redistribution. `.gitignore` blocks
> `.otf`/`.ttf`/`.woff` outright.
>
> **Preface staff:** get the files from the studio, or from `public/fonts/` in
> the film project.
> **Everyone else:** license your own seat from the foundry, or substitute a
> geometric sans of similar proportion.

| Weight | Used in film work |
|---|---|
| 400 Regular | **yes** |
| 500 Medium | no |
| 600 SemiBold | **yes** |
| 700 Bold | no |
| 800 Heavy | legacy dark films only |
| 900 Black | no |

**Andale Mono** (data labels and system captions in some films) also isn't
included — it ships with macOS at
`/System/Library/Fonts/Supplemental/Andale Mono.ttf`.

## Logos — `logos/`

| File | Size | Use on |
|---|---|---|
| `preface-logo-white.png` | 640×64 | dark grounds and photography only |

Preface's own mark, published here for team use. No dark-ink version exists in
this set — the paper-white films end on the wordmark set in Galano rather than a
logo lockup, so one hasn't been needed. Ask the studio if you need it.

## Loading fonts in Remotion

Through `@remotion/fonts` with a `delayRender` handle, so the first frame cannot
render before the faces are ready — see `src/brand/fonts.ts` in the film project
and §3 of the build runbook.
