# Remotion, and how we make films with it

For anyone joining these projects — human or model. Read this first, then
`REmotion.md` at the repo root when you actually need to build (that one is the
step-by-step runbook: scaffold, fonts, commands, gotchas).

---

## 1. What Remotion is

**Remotion is React that renders to video.** You write a component; Remotion
mounts it, seeks it to frame 0, screenshots, seeks to frame 1, screenshots —
then hands the frames to ffmpeg.

That's the whole idea. There is no timeline UI, no keyframes, no scrubber that
owns the truth. The truth is code:

```tsx
const frame = useCurrentFrame();          // 0, 1, 2 … which frame am I?
const opacity = interpolate(frame, [0, 15], [0, 1]);
return <AbsoluteFill style={{ opacity }}>…</AbsoluteFill>;
```

### Why that matters

| | After Effects | Remotion |
|---|---|---|
| Source of truth | a binary project file | text you can diff and review |
| Repeating a design | copy/paste layers | a component, used 38 times |
| Timing | dragging keyframes | derived from data (a VO timeline, a CSV) |
| Re-cut after a change | re-drag everything | change one number, re-render |
| Versioning | `final_v7_REAL.aep` | git |
| A model can edit it | no | **yes — this is the point** |

The last row is why we use it. A film in Remotion is a thing Claude can read,
reason about, and change precisely. "Make card 12 two frames longer" is a real
instruction, not a request to open a GUI.

### Where it sits next to the tools you know

Remotion doesn't replace the edit suite — it replaces the *motion design* seat,
and only for work that is systematic.

| Tool | What it's for | Where it beats Remotion |
|---|---|---|
| **Premiere / DaVinci Resolve** | cutting footage, sound, grade, delivery | anything with real footage, an editor's judgement, or a colourist |
| **After Effects** | motion design, compositing, VFX | one-off hero animation, real motion blur, 3D, plugins, a designer's hand |
| **Remotion** | motion design as code | 38 cards that must behave identically; timing derived from data; a re-cut after the VO changes; a model doing the work |

Rule of thumb: **if the piece is a system, code it. If it's a performance, hand
it to a designer.** The Effie film was a system — one card component used 38
times, timing read from a VO timeline — so it was built here and then handed to
an After Effects vendor as a reference cut for them to add the craft Remotion
can't: real motion blur, depth, grain, transitions with a hand on them.

### The one rule you cannot break

**A frame must be a pure function of its frame number.**

Frames render out of order, in parallel, across several browser tabs. Frame 400
may render before frame 12. So anything visible must be computed from
`useCurrentFrame()` and nothing else. `Math.random()`, `Date.now()`, `useState`,
accumulating values, CSS transitions and CSS keyframes are all banned inside a
composition — they make frame N depend on something other than N, and you get
flicker that only shows up in the final render.

`REmotion.md §1` has the full list and the escape hatches.

---

## 2. How we actually use it

Six films and counting have come out of this one repo. The pipeline that
emerged is the same every time:

```
script (md)  →  VO (local TTS)  →  timeline.json  →  beat sheet (data)
                                                          ↓
   assets (selected, downscaled) → public/ → compositions → stills check
                                                          ↓
                                      render → out/*.mp4 → handover package
```

**1. The script becomes data.** A messy table from the client gets sorted into a
clean `SCRIPT.md` with cuts marked and open items flagged (`XXXX`, `$$`).

**2. The voice is rendered first, and it owns the timing.** Local TTS emits
`effie.wav` plus `timeline.json` — 42 segments, each with `start`, `dur`, `text`.
That file is the spine of the edit.

**3. The edit is a data structure, not layout code.** `src/effie/script.ts` is a
`BEATS` array: which VO segment a beat opens on, what text it carries, which
photo, which register. The renderer is dumb; the beat sheet is the film. This
is what makes a shot-list CSV export free, and what let the film re-time itself
when the VO was re-recorded 14 seconds shorter.

**4. Assets get prepared, not dumped.** 325 originals up to 9504×6320 → 50
selects at 2400px, EXIF flattened, renamed to what they mean
(`ooh-sogo.jpg`, not `SogoTV.jpeg`), with a manifest mapping back to the source.
3.3 GB → 23 MB.

**5. Check with stills, then render.** A full render is 10–15 minutes; a single
frame is one. Never burn a render to find out the type is too big.

**6. Package for whoever is next.** EDL CSV with timecode, one frame per beat,
the style spec, the VO and its timings.

---

## 3. Four practices that make or break it

These are the ones learned the expensive way.

### 3.1 Build clip by clip

**One mechanism, one composition, rendered on its own before it joins anything.**

The Hong Kong map is a component, exposed as `HK-Map` and `HK-Map-Alpha`
compositions, and also used inside the film at 56.5s. Same code, three outputs.
The loops are separate compositions. Each storyboard is its own composition.

Why it matters:

- A 5-second clip renders in ~1 minute. The 2:48 film takes 15. You will iterate
  20 times on the clip and twice on the film.
- A broken clip is isolated. A broken film is a haystack.
- Clips are reusable deliverables on their own — social cutdowns, alpha overlays
  for an editor, a loop for a slide.
- You can judge a clip. Nobody can judge 38 beats at once.

Assemble only when every clip stands up alone.

### 3.2 Feed in as much context as possible

**Give the model the actual files, not a description of them.**

The difference is stark. Timing guessed from word counts drifted **8.5 seconds**
out by the end of the film. Timing read from `timeline.json` — the real file,
handed over — was frame-accurate and survived a complete VO re-record.

So: hand over the VO timeline, the script, the asset manifest, the brand tokens,
the palette file, the previous render's shot list. Context that exists as a file
should never be summarised into a prompt. Summaries drift; files don't.

Corollary: **make your own work legible.** The beat sheet is data and the tokens
are constants precisely so the next session (or the next person) can read the
film instead of reverse-engineering it.

### 3.3 Animate on principle, not by feel

Four rules carry almost all of it:

- **Easing, always.** Nothing linear except a progress bar. Entrances decelerate
  (`ease-out`), exits accelerate (`ease-in`). Our beziers live in
  `src/brand/motion.ts` and never get retyped inline.
- **Match the speed ramp across a cut.** Outgoing accelerates away over 7
  frames, incoming decelerates in over 11. Read across the join, velocity is
  `/\` — fastest *at* the cut, still in the middle of each shot. This single
  choice is what stops a sequence of cards feeling like a slideshow.
- **Always on the move.** Every card creeps 1.022× (1.04 on heroes) for its
  whole life; photographs push 1.04–1.07. Small enough to be felt, not seen.
  A frame that has landed and stopped reads as a stall.
- **One focus point at a time, and movement points at it.** One centred axis, one
  thing entering, one thing the eye is being handed. If two things move for
  attention, neither gets it.

And the fifth, which is about restraint rather than motion: **rests are
composed too.** Eight ~1s holds where the frame empties between phases. Silence
in the picture is as authored as a cut.

### 3.4 Platform

**Claude is the right tool for this**, and it isn't close. The work is long-form
codebase reasoning — a 38-beat data structure, a shared token system, a renderer
that has to stay consistent across six films — plus reading the actual asset
files and running renders. It wants a coding agent with real file and shell
access, not a chat window.

**GPT can do it with the right skill integration** — the model isn't the
constraint, the harness is. What it needs to match this workflow: filesystem
read/write, a shell for `npx remotion`, image reading for checking stills, and
these documents loaded as instructions. Without the loop of *render a still →
look at it → fix it*, any model is animating blind.

---

## 4. Tips and tricks

**Derive timing; never count it.** If a file knows the answer — a VO timeline, a
CSV, an audio duration — read the file.

**Preview geometry without rendering.** The Hong Kong coastline was tuned as
ASCII art in a Python script: same ellipse maths, printed as a 76×42 grid of
dots. Three iterations in about ten seconds each, versus seven minutes a render.
Get the shape right in the cheapest medium that shows the problem.

**Run your data modules in Node.** `npx esbuild --bundle --loader:.json=json`
your `script.ts` and you can print the whole EDL — every beat's in/out timecode,
duration, asset, text — without opening a browser. That's how the vendor
shot-list got made, and how gaps and too-short cards get caught.

**Stills before renders.**

```bash
npx remotion still Effie-Text out/check.png --frame=1300
```

**Keep `public/` small.** It is copied on every single render. Ours is 40 MB and
that copy is the slowest part of the loop on iCloud Drive. 9504px originals
would make it unusable.

**iCloud Drive is a tax.** This project lives in `~/Library/Mobile Documents/…`,
and bundling plus the public-dir copy pays for it every run. If a render seems
hung, it's usually the copy. A local working copy is faster if you ever need the
iteration speed.

**Determinism has a helper.** `random("seed")` from Remotion, or a seeded sine
hash like `rnd()` in `motion.ts`. Never `Math.random()`.

**Use Remotion's `<Img>` and `<Audio>`, and `staticFile()`** — `<Img>` blocks the
frame until the image has actually decoded. A raw `<img>` will render you a
half-loaded frame at random.

**Many elements: SVG, not divs.** 200 tiles as `<rect>` in one SVG is fast; as
nested divs with shadows it crawls.

**Alpha renders need their own config.** ProRes rejects the CRF in the main
`remotion.config.ts`:

```bash
npx remotion render HK-Map-Alpha out/map.mov --config=remotion-alpha.config.ts \
  --codec=prores --prores-profile=4444 --pixel-format=yuva444p10le
```

**Light is not dark re-skinned.** On dark, glow carries presence. On paper there
is none, so strokes and nodes need roughly 40% more weight to hold the frame.
Re-judge every value rather than swapping the palette.

**Density is an argument.** 48 large tiles read as "some documents"; 208 small
ones read as "1,847 pages". Pick the count that makes the claim you mean.

---

## 5. Do and don't

### Do

| | |
|---|---|
| Compute every visible value from `useCurrentFrame()` | the only real rule |
| Put the edit in a data structure | beats as data, renderer dumb |
| Drive timing off a file | VO timeline, CSV, measured duration |
| Build and render clips individually | then assemble |
| Check single frames before full renders | one minute vs fifteen |
| Keep easing and spacing in tokens | `motion.ts`, `light.ts` — never inline beziers |
| Give photographs air, or the whole frame | plate or full-bleed, not crammed |
| Make placeholders look like placeholders | dashed grey boxes captioned with what belongs there |
| State when a number is invented | a dashed empty slot beats a plausible lie |
| Name assets for what they mean | `ooh-sogo.jpg`, not `DSC08659.jpg` |
| Read files over describing them | to yourself, to the model, to the vendor |

### Don't

| | |
|---|---|
| `Math.random()`, `Date.now()`, `useState`/`useEffect` for anything visible | non-deterministic frames, flicker in the final only |
| CSS `transition` / `@keyframes` / `setTimeout` | the renderer seeks; CSS doesn't follow |
| Guess timing from word counts | drifted 8.5s on a 2:30 film |
| Linear interpolation for anything physical | reads as machinery |
| Overshoot springs "for life" | ours are critically damped: things settle, never bounce |
| Let a landed frame go completely still | dead air; keep the slow drift |
| Move two things for attention at once | neither wins |
| Fill every silence with type | ~1/3 of lines get text; the rests are the design |
| Put 9504px originals in `public/` | copied every render |
| Trace a Google/Apple Maps screenshot | cannot be licensed; draw your own geometry |
| Invent a figure to fill a layout | mark it TBC and say so |
| Ship licensed fonts in a handover zip | name them; let the vendor license them |
| Add a third hue | two, both semantic |
| Render the film to find out the type is too big | still, look, fix, then render |

---

## 6. Commands you'll actually use

```bash
npm run dev                                   # studio, live preview
npx remotion still <Comp> out/x.png --frame=N # one frame
npx remotion render <Comp> out/x.mp4          # the whole thing
npm run typecheck                             # tsc --noEmit
```

Everything else — scaffolding, fonts, browser troubleshooting — is in
`REmotion.md`.
