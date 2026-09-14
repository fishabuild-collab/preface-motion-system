# Remotion: build these films from an empty folder

Assumes nothing exists. Follow top to bottom and you get the same six videos.

---

## 1. The one rule

Remotion is React that renders to video. It mounts your component once per frame, screenshots
it, and stitches the frames with ffmpeg.

> **A frame must be a pure function of its frame number.**

Remotion renders frames **out of order and in parallel** across several browser tabs. Frame 400
may render before frame 12, in a different tab. So a frame can only depend on `useCurrentFrame()`.

Banned inside a composition:

- `Math.random()`, `Date.now()`, `new Date()`
- `useState` / `useEffect` driving anything visible
- accumulating animation (`x += speed`) instead of computing `x` from `frame`
- physics or layout simulation stepped per frame

Need something organic? **Solve it once at module load and cache it.** A force-directed graph
must be solved outside the component, or every render produces a different graph.

---

## 2. Scaffold

```bash
mkdir myfilms && cd myfilms
npm init -y
npm i remotion@4.0.373 @remotion/cli@4.0.373 @remotion/fonts@4.0.373 react@19.2.0 react-dom@19.2.0
npm i -D typescript@5.9.3 @types/react@19.2.0
mkdir -p src/brand src/compositions src/loops public/fonts out
```

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

**`remotion.config.ts`**

```ts
import { Config } from "@remotion/cli/config";

Config.setVideoImageFormat("jpeg");
Config.setOverwriteOutput(true);
Config.setCodec("h264");
Config.setCrf(16);
```

**`src/index.ts`**

```ts
import { registerRoot } from "remotion";
import { RemotionRoot } from "./Root";

registerRoot(RemotionRoot);
```

**`src/Root.tsx`** — every composition is registered here; `id` is what you render by name.

```tsx
import React from "react";
import { Composition } from "remotion";
import "./brand/fonts";
import { Film1 } from "./compositions/Film1";

export const RemotionRoot: React.FC = () => (
  <>
    <Composition
      id="Film1-Tender"
      component={Film1}
      durationInFrames={600}   // 20s
      fps={30}
      width={1920}
      height={1080}
    />
  </>
);
```

**`package.json` scripts** — the ID must match `Root.tsx` exactly or the render fails:

```json
"scripts": {
  "dev": "remotion studio",
  "film:1": "remotion render Film1-Tender out/01-tender.mp4",
  "typecheck": "tsc --noEmit"
}
```

---

## 3. Fonts

Drop the licensed `.otf`/`.ttf` files in `public/fonts/`, then **`src/brand/fonts.ts`**:

```ts
import { loadFont } from "@remotion/fonts";
import { continueRender, delayRender, staticFile } from "remotion";

const handle = delayRender("Loading brand fonts");

const faces = [
  { file: "GalanoGrotesque-Regular.otf", family: "Galano Grotesque", weight: "400" },
  { file: "GalanoGrotesque-SemiBold.otf", family: "Galano Grotesque", weight: "600" },
  { file: "AndaleMono.ttf", family: "Andale Mono", weight: "400" },
];

Promise.all(
  faces.map((f) =>
    loadFont({ family: f.family, url: staticFile(`fonts/${f.file}`), weight: f.weight }),
  ),
)
  .then(() => continueRender(handle))
  .catch((err) => { console.error("Font loading failed", err); continueRender(handle); });
```

`delayRender()` holds the capture until the promise resolves — without it Remotion screenshots
before the fonts arrive and every frame silently renders in a fallback face. Import it once, in
`Root.tsx`.

---

## 4. Motion (`src/brand/motion.ts`)

Four helpers do all the animation. Copy this file verbatim — it is the house style.

```ts
import { Easing, interpolate, spring } from "remotion";

export const FPS = 30;

/** Things settle rather than bounce: the films argue precision. */
export const SPRING = {
  settle: { damping: 200, stiffness: 120, mass: 0.7 },
  snap:   { damping: 26,  stiffness: 340, mass: 0.6 },
  heavy:  { damping: 200, stiffness: 60,  mass: 1.1 },
} as const;

export const EASE = {
  out:   Easing.bezier(0.16, 1, 0.3, 1),
  inOut: Easing.bezier(0.65, 0, 0.35, 1),
} as const;

/** Spring 0 -> 1 starting at `delay`. */
export const rise = (frame: number, delay = 0, config: object = SPRING.settle, fps = FPS) =>
  spring({ frame: frame - delay, fps, config, durationInFrames: undefined });

/** Clamped eased map — the workhorse for timed reveals. */
export const ramp = (
  frame: number, from: number, to: number,
  outFrom = 0, outTo = 1, easing = EASE.out,
) =>
  interpolate(frame, [from, to], [outFrom, outTo], {
    extrapolateLeft: "clamp", extrapolateRight: "clamp", easing,
  });

/** Fade in, hold, fade out — wraps a beat that owns a time window. */
export const window_ = (
  frame: number, inStart: number, inEnd: number, outStart: number, outEnd: number,
) =>
  interpolate(frame, [inStart, inEnd, outStart, outEnd], [0, 1, 1, 0], {
    extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE.out,
  });

/** Deterministic pseudo-random — never Math.random(). */
export const rnd = (seed: number) => {
  const x = Math.sin(seed * 127.1 + 311.7) * 43758.5453;
  return x - Math.floor(x);
};
export const rndRange = (seed: number, min: number, max: number) =>
  min + rnd(seed) * (max - min);
```

---

## 5. Design system

**`src/brand/light.ts`** — deliberately narrow. Two hues, both semantic; three sizes; two weights.

```ts
export const L = {
  bg: "#F7F6F3", surface: "#FFFFFF", surfaceDim: "#F1EFEA",
  line: "#E4E0D8", lineStrong: "#C9C3B8",
  ink: "#1E2B3C", body: "#465666", muted: "#8A94A2",
  accent: "#2F6D9E", accentSoft: "#E9F0F6",   // machine-confirmed
  review: "#A75F49", reviewSoft: "#F5EBE7",   // needs a human
} as const;

export const lt = { display: 76, body: 32, label: 20 } as const;  // three sizes. no fourth.
export const lw = { normal: 400, semi: 600 } as const;            // nothing heavier.
```

Fonts: `"Galano Grotesque"` for display/body, `"Andale Mono"` for labels (uppercase,
`letterSpacing: "0.15em"`).

**Shared beat map** — all films cut on the same frames so the set reads as a series:

| Beat | Frame | Sec |
|---|---|---|
| mess | 0 | 0.0 |
| tension | 105 | 3.5 |
| transform (hero) | 180 | 6.0 |
| proof | 390 | 13.0 |
| end card | 510 | 17.0 |
| end | 600 | 20.0 |

Margin 130px at 1920×1080.

---

## 6. How a film is built

One component per beat, each wrapped in `window_`, each returning `null` when invisible so
off-screen beats cost nothing. The *subject* (a wall of pages, a set of documents, a graph)
lives outside the beats and is only ever transformed — never cut away from. That is what makes
the mechanism feel like it comes out of the thing you opened on.

```tsx
export const Film1: React.FC = () => {
  const frame = useCurrentFrame();
  const actOut = ramp(frame, 498, 512, 1, 0);   // clear before the end card

  return (
    <AbsoluteFill style={{ background: L.bg }}>
      <AbsoluteFill style={{ opacity: actOut }}>
        <Subject />        {/* persists, transforms across beats 1-3 */}
        <MessBeat />
        <TensionBeat />
        <HeroBeat />
        <ProofBeat />
        <Watermark />      {/* inside the fade, so the end card's logo doesn't double up */}
      </AbsoluteFill>

      <Sequence from={510}>
        <EndCard line="Every requirement, traced to its page." />
      </Sequence>
    </AbsoluteFill>
  );
};

const TensionBeat = () => {
  const frame = useCurrentFrame();
  const op = window_(frame, 114, 134, 170, 188);
  if (op <= 0.001) return null;
  return <AbsoluteFill style={{ opacity: op }}>…</AbsoluteFill>;
};
```

To move the subject and have other things follow it (threads, beams), express the transform as
a **shared function of frame**, then compute attachment points from it:

```ts
const wallAt = (frame: number) => ({
  s:  interpolate(frame, [104, 134, 252, 290], [1, 0.62, 0.62, 1], { /* clamp, easing */ }),
  cy: interpolate(frame, [104, 134, 252, 290], [660, 398, 398, 660], { /* … */ }),
});
// threads call wallAt(frame) too, so they stay attached while it moves
```

---

## 7. The three films

Each is: **mess → tension → mechanism (hero) → proof → end card.** The hero beat is ~7s and
carries the argument.

**Film 1 · Tender — "The one you missed."** A dense wall of ~200 page tiles fills the frame.
One tile burns clay: a mandatory clause on p.340 that voids the bid. The wall lifts, and six
requirements pull out onto threads into a ruled register — each tagged, each with its page
reference and confidence. The last to land is a 58% addendum conflict, in clay. *Claim:
coverage and traceability, not speed.*

**Film 2 · Reconciliation — "Two columns that don't agree."** Six incompatible document formats
scatter in, then collapse into one stack somebody has to key. Bank and ledger face each other;
pairs resolve one line at a time. Agreements join with a check; disagreements break into two
stubs that never meet — one a digit transposition (61,200 vs 61,020), one with no counterpart.
Then the exception opens against its source page. *Claim: it matches, and it stops when it can't.*

**Film 3 · Knowledge Base — "It walks out the door."** One person holds two years of context.
They resign on a Friday and it leaves with them — unless it was compiled first. A graph grows
outward from that person into ~35 linked, typed pages. Same resignation, but a new joiner starts
lit from the base. *Claim: knowledge that stays when the person doesn't.*

---

## 8. Wordless loops (1080×1350)

Feed cuts: no copy, no branding beyond a watermark, clear bands top and bottom for overlaid text.
Data stays legible (page refs, amounts, node labels, confidence) because it *is* the mechanism.

**A seamless loop means frame 0 and frame N are identical.** Three strategies:

| Loop | Strategy |
|---|---|
| Reconcile | periodic scroll — content advances exactly one full cycle of N rows over the duration |
| Knowledge | grows from a single node, holds, fades back to that node — which is the frame-0 state |
| Tender | the wall is present in every frame; each source runs its own phase so nothing resets globally |

Any idle drift must use **whole-number cycles per loop**, or the wobble jumps at the seam:

```ts
const wob = (phase: number, cycles: number) =>
  Math.sin((frame / DURATION) * Math.PI * 2 * cycles + phase);
```

A sweep that crosses the frame should be drawn twice, offset by its own width, so its tail
re-enters as its head exits.

---

## 9. Commands

```bash
npm run dev      # Remotion Studio — scrub and tweak live
npm run film:1
npm run typecheck
```

Check one beat without a full render:

```bash
npx remotion still src/index.ts Film1-Tender out/check.png --frame=300
```

Renders are deterministic: same commit, same bytes.

---

## 10. Gotchas

**Many elements: use SVG, not divs.** A 200-tile wall as `<rect>`s in one SVG renders fast; as
nested divs with shadows it crawls.

**Density carries meaning.** 48 large page tiles read as "some documents". 208 small ones read
as "1,847 pages". Same idea, different count, different claim.

**Light is not a re-skin of dark.** On dark, glow carries presence. On paper there is no glow,
so nodes and strokes need ~40% more weight to hold the same frame. Re-judge every value.

**Constraints cost specific things.** Three type sizes and two weights make a set calm and
coherent — and remove the big-number moment. A stat that was 76px on dark may have to drop to
32px. Know which beat you are trading away.

**If the origin is ambiguous, the claim dies.** Threads from a document wall read as floating
wires until the source tile is visibly marked. Then fade landed threads back to ~25%, so the
newest is always the bright one.

**Chrome after a laptop sleep.** A render that hangs at `Opening browser` and dies with
`Timed out after 25000 ms while trying to connect to the browser` means the bundled headless
shell is broken:

```bash
rm -rf node_modules/.remotion && npx remotion browser ensure
```

Until then, point at system Chrome:

```bash
npx remotion render src/index.ts Film1-Tender out/x.mp4 \
  --browser-executable="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
```

**Transient `localhost:3000 gave no response`** happens occasionally mid-batch. Re-run that one
composition, and always confirm a batch finished rather than assuming.
