# Preface — visual guide for motion

The house style for Preface films, loops and case work. Extracted from
`preface.ai` and hardened across six films.

Code is the source of truth for values: `src/brand/light.ts`,
`src/brand/tokens.ts`, `src/brand/motion.ts`, `src/loops/tokens.ts`. If this
document and a token file disagree, the token file wins — fix this document.

---

## 1. The five rules

Everything below is detail. These are the film.

1. **Two hues, both semantic.** Blue means machine-confirmed. Clay means a human
   must look. Never decorate with colour.
2. **Three type sizes, two weights.** 400 and 600. Nothing heavier, no fourth size.
3. **Things settle, never bounce.** Critically damped. No overshoot anywhere.
4. **One focus point, and the movement points at it.** One centred axis.
5. **Never claim what isn't true.** Placeholders look like placeholders;
   unknown figures render as marked gaps.

---

## 2. Palette

### Light — canonical

`src/brand/light.ts`. Paper-white ground. This supersedes the dark look.

| Token | Hex | Meaning |
|---|---|---|
| `bg` | `#F7F6F3` | the ground; every non-photo frame |
| `surface` | `#FFFFFF` | cards, panels |
| `surfaceDim` | `#F1EFEA` | plate backing, inert fills |
| `line` | `#E4E0D8` | hairlines |
| `lineStrong` | `#C9C3B8` | borders, dot matrices, dashed placeholder edges |
| `ink` | `#1E2B3C` | hero type |
| `body` | `#465666` | secondary type |
| `muted` | `#8A94A2` | struck-through, pending, de-emphasised |
| `accent` | `#2F6D9E` | **machine-confirmed** — focus words, all numbers, rules |
| `accentSoft` | `#E9F0F6` | accent tint |
| `review` | `#A75F49` | **needs a human** — strike-throughs, exceptions |
| `reviewSoft` | `#F5EBE7` | review tint |
| Reversed | `#FFFFFF` | all type over photography |

Clay is deliberately not a warning yellow. It means *a person should look at
this*, not *error*.

### Dark — legacy

`src/brand/tokens.ts` (`c`). The original ink/aqua/signal system, kept for the
earlier films. Do not start new work on it. If you must: on dark, glow carries
presence; on paper there is none, so strokes and nodes need roughly **40% more
weight** to hold the same frame. It is not a re-skin — re-judge every value.

---

## 3. Type

**Galano Grotesque**, weights **400** and **600** only. Andale Mono exists in the
system for data and labels, but the Effie cut uses none — prefer display for
everything unless you need a mono reading.

> Fonts are licensed to Preface. **Never ship the binaries** in a handover —
> name the faces and let the vendor license them.

Three sizes per context, scaled to the canvas:

| Context | Display | Body | Label |
|---|---|---|---|
| Film 1920×1080 (`lt`) | 76 | 32 | 20 |
| Loops 1080×1350 (`ll`) | 64 | 34 | 26 |

A phone-sized loop needs *larger* small text than a downscale would give — 20px
at 1080 wide is unreadable in a feed.

### The Effie case-film scale

A wider scale for a long-form piece with photography. Sizes in px at 1920×1080:

| Element | Size | Weight | Tracking | Leading |
|---|---|---|---|---|
| Focus word | 118 | 600 | −0.035em | 1.02 |
| Hero line | 84 | 600 | −0.02em | 1.1 |
| Body line | 58 | 600 | −0.02em | 1.1 |
| Lead / run-up | 38 | 400 | −0.02em | 1.1 |
| SUPER (Chinese) | 40 | 400 | normal | 1.1 |
| Stat value / label | 96 / 26–28 | 600 / 400 | −0.03em | 1.0 |

Type is centred on one axis throughout. Max text width 1450–1500px; side margins
140px; reversed type sits 132px from the bottom.

---

## 4. Motion

`src/brand/motion.ts`. Import these — never retype a bezier inline.

### Easing

```
ease-out   cubic-bezier(0.16, 1, 0.30, 1)    entrances
ease-in    cubic-bezier(0.55, 0, 1, 0.45)    exits
ease-inOut cubic-bezier(0.65, 0, 0.35, 1)    rare
```

### Springs — all critically damped

| Name | Use | Config |
|---|---|---|
| `settle` | default entrance | damping 200, stiffness 120, mass 0.7 |
| `snap` | something landing in a slot | damping 26, stiffness 340, mass 0.6 |
| `heavy` | a panel, a collapsing stack | damping 200, stiffness 60, mass 1.1 |

**Nothing overshoots.** The films argue that a process is precise and
accountable; a wobble contradicts the copy.

### The speed-ramped cut

The most important single behaviour, and the one most likely to get "fixed" by
someone else:

| | |
|---|---|
| Incoming | opacity 0→1 over **11f**, Y +46px→0, ease-out |
| Outgoing | opacity 1→0 over **7f**, Y 0→−36px, ease-in |

Across a join the velocity curve is `/\` — fastest at the cut, still in the
middle of each shot.

### Everything else

| Move | Spec |
|---|---|
| Word stagger | 1.4f per word, Y +24% of font size, settle |
| Type drift zoom | 1.0 → 1.022 over the card (1.04 on heroes) |
| Ken Burns, full-bleed | 1.0 → 1.07, ease-out |
| Ken Burns, plate | 1.0 → 1.05 |
| Ken Burns, strip | 1.0 → 1.04 |
| Strip frames | hard 1-frame cut, staggered 6–14f |
| Ping ring | scale 0.3 → 1.0 over 40f, fading out |

**Nothing is ever completely still.** A landed frame keeps its slow drift until
it cuts.

---

## 5. Formats

| Use | Canvas | fps | Notes |
|---|---|---|---|
| Films, case work | 1920×1080 | 30 | landscape even for social |
| Wordless loops | 1080×1350 (4:5) | 30 | 8s, seamless |
| Storyboards | contact sheet | — | stills, for picking a direction |

**Loops reserve clear space.** `BAND` keeps the top 230px and bottom 260px empty
so copy can be overlaid later. Every moving part stays inside the band.

---

## 6. Photography

Three registers. The size of the picture carries the phase.

| Register | Geometry | When |
|---|---|---|
| **Plate** | 1080×608, radius 14, on `surfaceDim` | default — one picture with air around it |
| **Full-bleed** | fills frame, bottom scrim, reversed type | peaks only, a handful per film |
| **Strip** | 3–4 frames, 470px tall (400w four-up / 500w three-up), 26px gutters, radius 12 | where the VO lists things |

Full-bleed scrim — bottom third only, so the top half of the photograph stays clean:

```
rgba(12,18,26,0)    at 42%
rgba(12,18,26,0.34) at 66%
rgba(12,18,26,0.76) at 100%
```

Pin a crop (`objectPosition`) when centring would cut the subject — a headline in
the photograph counts as the subject.

---

## 7. Honesty in the frame

Not decoration — these have all been load-bearing decisions.

- **Placeholders look like placeholders.** Dashed grey boxes captioned with what
  belongs there. No invented screenshots, no fake OOH photography, no mock brand
  marks that could be mistaken for shipped work.
- **Unknown figures render as marked gaps.** The Effie results card shows `$$`
  greyed over a dashed rule labelled "TBC" because the business number hasn't
  landed. A plausible-looking invented figure is worse than a visible hole.
- **Abstract-structural over mock product UI.** The workflow films claim nothing
  about a shipped interface.
- **Draw your own geometry.** The Hong Kong map is our own ellipse union, not a
  traced Maps screenshot — that can't be licensed for broadcast.
- **Say when data is placeholder** in the handoff doc, every time.

---

## 8. Quick check before shipping

- [ ] Two hues only, both meaning something
- [ ] Three sizes, weights 400/600 only
- [ ] No overshoot anywhere
- [ ] Every cut speed-ramped; nothing landed is frozen
- [ ] One focus point per frame
- [ ] Text held back where picture can carry it; rests intact
- [ ] Loops: everything inside `BAND`
- [ ] Every number real, or visibly marked TBC
- [ ] Fonts not in the handover; assets downscaled; manifest included
