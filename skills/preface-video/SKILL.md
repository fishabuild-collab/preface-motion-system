---
name: preface-video
description: Build, edit or extend a Preface film, loop, teaser or case video in Remotion — applying the house visual system (paper-white palette, two semantic hues, three type sizes, speed-ramped cuts) and the VO-driven beat-sheet workflow. Use when the request involves making or changing a video, animation, motion graphic, kinetic-type sequence, social loop, storyboard or alpha overlay in a Remotion project; when cutting picture to a voiceover; when preparing a vendor handover package; or when someone asks for "our style", "on brand" motion, or a new clip for an existing film.
---

# Preface video

Build films the way this shop builds films. Two references, both required
reading before you write animation code:

- `docs/preface-brand.md` — the visual system (palette, type, motion spec, photo registers)
- `docs/remotion-explainer.md` — how we work and why; do/don't list
- `REmotion.md` (repo root) — build runbook, commands, browser gotchas

Values live in code and code wins: `src/brand/light.ts`, `src/brand/motion.ts`,
`src/loops/tokens.ts`.

---

## The rule that breaks everything else

**A frame must be a pure function of `useCurrentFrame()`.** Frames render out of
order in parallel tabs. No `Math.random()`, `Date.now()`, `useState`/`useEffect`
driving visuals, accumulated values, CSS `transition`/`@keyframes`, or
`setTimeout`. Use `random("seed")` or the seeded `rnd()` in `motion.ts`.

---

## Workflow

### 1. Gather context before designing

Read the real files, don't summarise them:

- the script (and mark what is cut, and what is an unresolved `XXXX` / `$$`)
- the VO's `timeline.json` if one exists — **this owns the timing**
- the asset folder: inventory it, look at contact sheets, pick selects
- `src/brand/*` tokens

Never guess a duration that a file already knows. Word-count timing drifted
8.5s on a 2:30 film; timeline-derived timing survived a full VO re-record.

### 2. Prepare assets

Select, downscale to ~2400px wide, flatten EXIF, rename to meaning, and write a
manifest mapping back to originals.

```bash
magick "$SRC" -auto-orient -resize '2400x2400>' -strip -quality 82 "public/assets/<phase>/<name>.jpg"
```

`public/` is copied on **every** render — keep it small.

### 3. Write the edit as data

The beat sheet is the film; the renderer is dumb. Follow `src/effie/script.ts`:
a `BEATS` array anchored to VO segment numbers, each carrying text, shot and
register. Benefits: re-timing is free, the EDL export is free, and a human can
review the edit without reading JSX.

Hold text back. On the Effie film only **14 of 42** spoken lines carry type;
the rest is picture under voice. Leave real rests (~1s, frame empty) between
phases — `hold: "segEnd"` is how they're made.

### 4. Build clip by clip

One mechanism, one composition, rendered and judged alone before it joins the
film. Register standalone compositions for anything reusable (see `HK-Map` /
`HK-Map-Alpha`). Assemble only when each clip stands up.

### 5. Check frames, then render

```bash
npx remotion still <Comp> out/check.png --frame=1300   # ~1 min
npx remotion render <Comp> out/film.mp4                # 10–15 min
npm run typecheck
```

Look at the still. Actually look at it. Never render the film to discover the
type is too big.

For geometry, preview in the cheapest medium first — an ASCII grid in Python
beats a seven-minute render for tuning shapes. To inspect the edit as data
without a browser, bundle the module with `npx esbuild --bundle
--loader:.json=json` and run it in Node.

### 6. Alpha and handover

```bash
npx remotion render <Comp>-Alpha out/x.mov --config=remotion-alpha.config.ts \
  --codec=prores --prores-profile=4444 --pixel-format=yuva444p10le
```

(ProRes rejects the main config's CRF — hence the alternate config.)

A vendor package contains: reference cut, VO + timings, prepared assets +
manifest, EDL CSV with timecode, one frame per beat, the style/motion spec, and
the source. **No font binaries** — they are licensed to Preface.

---

## Non-negotiables

**Visual**
- Two hues, both semantic: `accent` = machine-confirmed, `review` = needs a human
- Three type sizes, weights 400/600 only
- One centred axis; generous margins; nothing crowded
- Photography as plate (default), full-bleed (peaks only), or strip (lists)

**Motion**
- Ease everything: entrances `ease-out`, exits `ease-in`
- Speed-ramp every cut: out 7f accelerating, in 11f decelerating
- Critically damped springs — things settle, never bounce
- Nothing fully still: landed cards keep a 1.022–1.04 drift
- One focus point; movement points at it

**Honesty**
- Placeholders look like placeholders (dashed grey, captioned)
- Unknown figures render as a marked gap, never an invented number
- Draw your own geometry; no traced Maps screenshots
- Say in the handoff which data is still placeholder

---

## When you finish

- [ ] `npm run typecheck` clean
- [ ] Stills checked at each new register before the full render
- [ ] Every number real or visibly TBC
- [ ] Loops: all movement inside `BAND`
- [ ] Handoff doc updated with what's open

---

## Portability

This skill assumes filesystem access, a shell for `npx remotion`, and the
ability to **look at rendered stills**. Without that loop — render, look, fix —
any model is animating blind. On a platform other than Claude Code, load these
same three documents as system instructions and make sure image reading and
shell execution are wired up.
