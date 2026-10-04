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

**Message:** "Can I please have a **green tea**?", with vertical Japanese beside it: お茶をください ("tea, please") and a red seal stamp 茶 ("tea").

**Look:** Warm paper with a faint grain, ink-colored text and matcha green accents. The font is Shippori Mincho (400 and 600), a Japanese Mincho serif from Google Fonts.

| Token | Value | Used for |
|---|---|---|
| `--paper` / `--paper-deep` | `#f2ede3` / `#e6dfd0` | background gradient |
| `--ink` / `--ink-soft` | `#2b2a26` / `#7a766c` | text, steam, vertical Japanese |
| `--matcha` / `--matcha-light` | `#6b8f47` / `#a9c28a` | "green tea", ripple, divider line, leaves |
| `--seal` | `#b8392b` | the 茶 stamp |

**Elements and motion (timeline from page load):**

| When | What | How it's built |
|---|---|---|
| always | Paper grain over the whole screen | Inline SVG noise filter (`feTurbulence`) as a background image |
| 0.3 s | Tea cup soaks in like ink (blur → sharp, 2.6 s). Three steam wisps rise on a staggered 6 s loop | Inline SVG: cup body, tea surface, foot, brush band, steam paths |
| 1.2 s | Headline soaks in | CSS `ink` animation |
| 2.4 s | Vertical Japanese soaks in | same |
| 3 s | A thin green line draws itself under the headline | CSS width animation |
| 4.2 s | The seal stamps down with a little overshoot and settles slightly rotated | CSS `cubic-bezier` bounce |
| 5 s → forever | One tea leaf drifts down every 4.5 s, taking 14–22 s, with random drift and spin | Created and removed by JS |

**Interaction:** Tapping or clicking anywhere sends out a slow green ripple from that point (2.8 s).

**Layout:** Side by side (message and vertical Japanese) on wide screens. Below 600 px wide, the Japanese moves under the message and switches to horizontal text.

## Shared patterns

- **Colors as named tokens.** Each page defines its colors once on `:root` and uses them by name.
- **Fluid sizing.** Text and objects scale with the window through `clamp(min, preferred, max)`.
- **Choreographed entrance.** Elements start hidden and appear in a staged sequence driven by `animation-delay`. Nothing waits on user input.
- **Ambient particles.** JS creates small decorative elements (stars, hearts, leaves) at random positions with random timing, then removes each one when its animation ends.
- **Reduced motion is respected.** When the device's "Reduce Motion" setting is on, every page stops animating, shows its final composed state, and skips particles and ripples. (In `tea/index.html` the variable `calm` is *true* when motion is allowed. The name reads backwards.)
- **Decoration is hidden from screen readers.** Particles, the moon, the cup and the seal are `aria-hidden`. The Japanese line carries `lang="ja"`.
- **Self-contained.** The only outside dependency is Google Fonts.
