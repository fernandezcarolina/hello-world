# Project map

A map of the `hello-world` web codebase as it stands today, written as the starting point for the iOS port (see [ios.md](ios.md)).

## At a glance

| | |
|---|---|
| What it is | Two short, animated, single-screen notes, each written to one person |
| Tech | Plain HTML + inline CSS + a little inline JavaScript. No framework, no build step, no dependencies |
| Hosting | Vercel, deployed from `main` on GitHub. https://hello-world-delta-dun.vercel.app |
| Data | None. No backend, no storage, no accounts, no network calls besides loading fonts |

## Files

```
hello-world/
├── index.html        →  /       "Hello, Sam"  (bedtime note)
├── tea/index.html    →  /tea    "Green tea, please"
├── README.md         learning-repo intro
├── CLAUDE.md         guidance for Claude Code
├── PROJECT_MAP.md    this file
└── ios.md            iOS port plan
```

Each page is one self-contained file. The pages don't share CSS or JS, and they don't link to each other. You reach each one only by its URL.

## Screen 1: "Hello, Sam" (`index.html`)

**Message:** "Hello, *Sam.*" and "Come to bed at **10 pm** ♡"

**Look:** A night sky made of a radial gradient from deep plum (`#3a1631`) to near-black violet (`#0f0a1f`), with rose text and a red accent. The font is Instrument Serif (regular and italic) from Google Fonts.

| Token | Value | Used for |
|---|---|---|
| `--night-top` | `#0f0a1f` | sky, top |
| `--night-bottom` | `#3a1631` | sky, horizon glow |
| `--rose` | `#ffc2cf` | body text |
| `--red` | `#ff4d6d` | "Sam.", hearts, title glow |
| `--moon` | `#fff4e0` | moon |

**Elements and motion (timeline from page load):**

| When | What | How it's built |
|---|---|---|
| 0.3 s | Moon fades in (top right), then pulses its glow every 6 s | CSS `div` with `box-shadow` glow |
| 0 s | 90 stars scattered at random, each twinkling on a 3 s loop with a random delay | Created by JS, 2 px dots |
| 0.8 s | "Hello, Sam." un-blurs and rises into place (1.8 s) | CSS `reveal` animation |
| 2.4 s | "Come to bed at 10 pm ♡" un-blurs and rises | same |
| 3 s → forever | A heart floats up every 0.45 s from a random spot along the bottom, at a random size, drift and spin. Each one rises for 7–13 s | Created and removed by JS |
| 5 s → forever | "10 pm" does a heartbeat pulse every 1.6 s | CSS `scale` keyframes |

**Interaction:** None.

## Screen 2: "Green tea, please" (`tea/index.html`)

**Message:** "Can I please have a **green tea**?", then in italic "Two cups, one slow afternoon, with you." Beside it, vertical Japanese お茶をご一緒に ("shall we have tea together") and a red seal stamp 茶 ("tea").

**Look:** Soft and a little romantic. The fonts are Shippori Mincho (400 and 600, a Japanese Mincho serif) and Cormorant Garamond italic for the second line, both from Google Fonts. The page follows the device's light/dark setting:

- **Light:** blush paper in late-afternoon sun, with a breathing rose glow behind the cups.
- **Dark:** the earlier dark tea palette, lit by a low rose candle glow.

| Token | Light | Dark | Used for |
|---|---|---|---|
| `--paper` / `--paper-deep` | `#f7eee8` / `#ecdcd4` | `#23211c` / `#121110` | background gradient |
| `--glow` | `#f4cfcf` | `#4a2a2c` | breathing glow behind the cups |
| `--ink` / `--ink-soft` | `#3a2c2c` / `#8a7470` | `#efe9dc` / `#a39d90` | text, cup outlines, steam, italic line |
| `--matcha` / `--matcha-light` | `#6b8f47` / `#a9c28a` | `#9cc075` / `#5f7d42` | "green tea", leaves |
| `--rose` / `--petal` | `#c46a74` / `#f3c6cb` | `#e0939b` / `#d99aa2` | heart, cup band, divider, ripple, petals |
| `--seal` / `--seal-text` | `#b8473f` / paper | `#c4473a` / ink | the 茶 stamp |
| `--cup` | `#fdf9f5` | `#2e2b25` | cup body |
| `--grain` | warm grain | light grain | paper texture |

**Elements and motion (timeline from page load):**

| When | What | How it's built |
|---|---|---|
| always | Paper grain over the whole screen, and a rose glow that slowly breathes (9 s) | SVG noise filter as a background image; `body::after` radial gradient |
| 0.3 s | Two cups leaning together soak in like ink (blur → sharp, 2.6 s) | Inline SVG: one cup drawn in `<defs>`, used twice and tilted |
| 2 s → every 8 s | A steam wisp rises from each cup, they curl toward each other and a small heart appears where they meet, then drifts up and fades | SVG paths drawn with `stroke-dashoffset`; heart synced on the same 8 s loop |
| 1.2 s | Headline soaks in | CSS `ink` animation |
| 2.2 s | Italic line soaks in | same |
| 3 s | Vertical Japanese soaks in | same |
| 3.6 s | A thin rose line draws itself under the message | CSS width animation |
| 4.8 s | The seal stamps down with a little overshoot and settles slightly rotated | CSS `cubic-bezier` bounce |
| 4 s → forever | A cherry blossom petal (or, one time in four, a tea leaf) drifts down every 3 s, taking 13–21 s, with random drift and spin | Created and removed by JS |

**Interaction:** Tapping or clicking anywhere sends out a slow rose ripple (2.8 s) and a small heart that floats up and fades (2.6 s).

**Layout:** Side by side (message and vertical Japanese) on wide screens. Below 600 px wide, the Japanese moves under the message and switches to horizontal text.

## Shared patterns

- **Colors as named tokens.** Each page defines its colors once on `:root` and uses them by name.
- **Fluid sizing.** Text and objects scale with the window through `clamp(min, preferred, max)`.
- **Choreographed entrance.** Elements start hidden and appear in a staged sequence driven by `animation-delay`. Nothing waits on user input.
- **Ambient particles.** JS creates small decorative elements (stars, hearts, leaves) at random positions with random timing, then removes each one when its animation ends.
- **Reduced motion is respected.** When the device's "Reduce Motion" setting is on, every page stops animating, shows its final composed state, and skips particles and ripples. (In `tea/index.html` the variable `calm` is *true* when motion is allowed. The name reads backwards.)
- **Decoration is hidden from screen readers.** Particles, the moon, the cup and the seal are `aria-hidden`. The Japanese line carries `lang="ja"`.
- **Self-contained.** The only outside dependency is Google Fonts.
