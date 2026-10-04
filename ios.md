# iOS port plan

A plan for bringing the two web notes ("Hello, Sam" and "Green tea, please") to iPhone as a native Swift/SwiftUI app. For what exists today, see [PROJECT_MAP.md](PROJECT_MAP.md).

> **Status: draft for discussion.** The scope choices below, especially "What we're not building", are proposals. Confirm or change them before work starts.

## Proposed shape of the app

- One SwiftUI app with the two notes as full-screen pages. You swipe between them horizontally (`TabView` with `.tabViewStyle(.page)`).
- Targets iPhone on **iOS 17 or later**. That gives us `PhaseAnimator`, `KeyframeAnimator` and Metal shader effects, which cover everything the web pages animate.
- Built in Xcode, which is already installed on this Mac. It lives in an `ios/` folder in this same repo, so the web and app versions sit side by side.

## What carries over

These ideas move across almost one-to-one. Only the syntax changes.

| Web | SwiftUI | Notes |
|---|---|---|
| Color tokens on `:root` | Named colors in the Asset Catalog (or a `Color` extension) | Same names, same hex values. See the token tables in PROJECT_MAP.md |
| Instrument Serif, Shippori Mincho (Google Fonts) | Bundled `.ttf` files registered under `UIAppFonts` in Info.plist, used with `.font(.custom(...))` | Both are free, open-license fonts that may be bundled. Include the italic for "Sam." and the 600 weight for "green tea" |
| `radial-gradient` backgrounds | `RadialGradient` / `EllipticalGradient` | |
| Text glow (`text-shadow`), moon glow (`box-shadow`) | `.shadow(color:radius:)`, stacked for a double glow | |
| Blur-in "reveal" and "ink" entrances | `.opacity`, `.blur(radius:)`, `.offset` driven by `withAnimation(.easeOut(duration:).delay(...))` | Same delays as the web timeline |
| Heartbeat on "10 pm", steam wisps, moon pulse | `PhaseAnimator` or `.repeatForever` animations | |
| Seal stamp with overshoot | `.spring(bounce:)` on scale and rotation | The spring stands in for the web's `cubic-bezier` |
| Divider line drawing itself | `.frame(width:)` animated from 0, or `.trim(from:to:)` on a line shape | |
| Inline SVG tea cup | A SwiftUI `Shape` (or a few) built by translating the SVG path commands into `Path` | Small drawing, easy to port by hand |
| Reduce Motion support | `@Environment(\.accessibilityReduceMotion)` | Same rule: when on, show the final still frame with no particles and no ripples |
| `aria-hidden` decoration | `.accessibilityHidden(true)` | |
| `lang="ja"` on the Japanese line | Set the text's language (e.g. `.environment(\.locale, Locale(identifier: "ja"))` or a language attribute on the `AttributedString`) so VoiceOver reads it in Japanese | |
| Tap-anywhere ripple | `SpatialTapGesture` gives the tap location. Spawn an expanding `Circle().stroke` there | |

## What changes

Here the web approach doesn't translate directly, so the app does it differently.

| Area | On the web | In the app |
|---|---|---|
| **Particles** (90 stars, a stream of hearts, falling leaves) | JS adds a DOM element per particle and removes it after its animation | Draw them all in one `Canvas` inside a `TimelineView(.animation)`. Each particle's position is computed from the clock and a random seed, so nothing gets added or removed. This is much lighter than one SwiftUI view per particle |
| **Sizing** (`clamp()`, `vw`/`vh`) | Sizes scale with the window width | Fixed sizes tuned for iPhone, scaled with `@ScaledMetric` so they respect the user's text size. Use `GeometryReader` or `containerRelativeFrame` only where something has to track the screen (moon position, particle area) |
| **Vertical Japanese** (`writing-mode: vertical-rl`) | Built-in CSS | SwiftUI has no vertical text mode. Stack the characters in a `VStack`, one per line, keeping them upright. That's how vertical Japanese is set, and it works for a short phrase |
| **Narrow-screen layout** (`@media (max-width: 600px)`) | Message and Japanese side by side, stacked when narrow | iPhone portrait is always the "narrow" case, so stack by default. Use `ViewThatFits` or size classes if we later want side by side in landscape |
| **Paper grain** (SVG `feTurbulence` filter) | Generated noise in the browser | Either a small tiling noise image in the Asset Catalog (simplest) or a Metal `colorEffect` shader. Start with the image |
| **Navigation** (two separate URLs) | You only reach a page by knowing its address | Swipeable pages inside one app. There are no URLs |
| **Fonts** | Loaded from Google's servers on every visit | Shipped inside the app, so they work offline |
| **Shipping updates** | Merge to `main` and it's live within seconds | Rebuild in Xcode and reinstall on the phone. A TestFlight or App Store update goes through Apple review |
| **Timing** | `setTimeout` / `setInterval` and CSS delays | Animation delays, `.task { try await Task.sleep(...) }`, and `TimelineView` |

## What we're not building

Proposed scope limits for the first version:

- **No web view wrapper.** We won't load the existing site inside a `WKWebView`. The point is a native SwiftUI port.
- **No shared code with the website.** The two versions are kept in sync by hand, using PROJECT_MAP.md as the reference. No cross-platform framework (React Native, Flutter, etc.).
- **No new features.** The app shows the same two notes with the same words, colors and motion. That means no editing messages in the app, no adding new notes, no settings screen.
- **No backend, accounts, sync or analytics.** The web version has none of these either.
- **No notifications, widgets or Live Activities.** A 10 pm bedtime reminder or a home-screen widget would be natural follow-ups, but they're out of scope for now.
- **No iPad-specific layout, Mac, Apple Watch or Android.** SwiftUI will run on iPad, but we won't design for it.
- **No App Store release in this phase.** The target is running on our own iPhone from Xcode, then TestFlight if we want to share it. Publishing is a separate decision.
- **No changes to the website** as part of this work.

## Open questions

1. Should the app open on "Hello, Sam" every time, or pick a note by time of day (for example, the bedtime note after 9 pm)?
2. Should page swipes show dots, or stay bare like the website?
3. Should the tea ripple add a light haptic tap?
4. Is an App Store release ever the goal? That affects naming, icons and the Apple Developer account ($99/year).

## Suggested order of work

1. Create the Xcode project in `ios/` and bundle both fonts.
2. Port "Green tea, please" first: static layout, cup shape, entrance timeline, seal stamp, then the ripple and leaves.
3. Port "Hello, Sam": gradient, moon, text reveal, heartbeat, then stars and hearts in `Canvas`.
4. Add the swipe between the pages, then do a Reduce Motion and VoiceOver pass on both.
5. Run it on a real iPhone and compare it side by side with the website.
