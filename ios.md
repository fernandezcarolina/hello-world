# iOS port plan

A plan for bringing the two web notes ("Hello, Sam" and "Green tea, please") to iPhone as a native Swift/SwiftUI app. For what exists today, see [PROJECT_MAP.md](PROJECT_MAP.md).

> **Status: draft for discussion.** The scope choices below, especially "What we're not building", are proposals. Confirm or change them before work starts. Research was updated on 2026-10-04: see [Research findings](#research-findings-2026-10-04).

## Proposed shape of the app

- One SwiftUI app with the two notes as full-screen pages that you swipe between horizontally.
- **Targets iPhone on iOS 26 or later**, built with Xcode 27 (already installed on this Mac).
  - Building with Xcode 27 means Liquid Glass is always on; the old opt-out is ignored. iOS 26 is the first version with the glass APIs, so targeting it means no "is this iOS version new enough?" checks in the code.
  - It also gives us every animation API we need.
  - iOS 27 is current and about 80% of iPhones were on iOS 26 by mid-2026. Targeting iOS 26 costs us nothing.
  - (The earlier draft said iOS 17. iOS 18 would also work, but it would need fallbacks for the glass APIs and for `@Animatable`.)
- **No app chrome.** No navigation bar, no toolbar, no tab bar. Each page is full-bleed art.
- Lives in an `ios/` folder in this same repo, so the web and app versions sit side by side.

## What carries over

These ideas move across almost one-to-one. Only the syntax changes.

| Web | SwiftUI | Notes |
|---|---|---|
| Color tokens on `:root` | Named colors in the Asset Catalog (or a `Color` extension) | Same names, same hex values. See the token tables in PROJECT_MAP.md |
| Instrument Serif, Shippori Mincho (Google Fonts) | Bundled `.ttf` files listed under `UIAppFonts` in Info.plist, used by PostScript name with `.font(.custom("InstrumentSerif-Regular", size:, relativeTo:))` | Both are OFL-licensed, so bundling is allowed. Ship only the weights we use (Instrument Serif regular + italic, Shippori Mincho 400 + 600). Shippori Mincho is a large font |
| `radial-gradient` backgrounds | `RadialGradient` / `EllipticalGradient`, or `MeshGradient` for a richer night sky | `MeshGradient` needs iOS 18+ |
| Text glow (`text-shadow`), moon glow (`box-shadow`) | `.shadow(color:radius:)`, stacked for a double glow | |
| Blur-in "reveal" and "ink" entrances | `KeyframeAnimator` animating opacity, blur and offset together. A custom `Transition` if we reuse it | `TextRenderer` (iOS 18) can soak the text in line by line or glyph by glyph, if we want it finer than the web version |
| Heartbeat on "10 pm", steam wisps, moon pulse | A looping `PhaseAnimator`, or `keyframeAnimator(repeating: true)` | |
| Seal stamp with overshoot | `.spring(bounce:)` on scale and rotation | The spring stands in for the web's `cubic-bezier` |
| Divider line drawing itself | `.trim(from:to:)` on a line shape, or an animated `.frame(width:)` | |
| Inline SVG tea cup | A SwiftUI `Shape` built by translating the SVG path commands into `Path` | Small drawing, easy to port by hand |
| Reduce Motion support | `@Environment(\.accessibilityReduceMotion)` | Same rule as the web: when it's on, show the final still frame, pause particles, turn rises and springs into simple fades, drop the heartbeat, keep the static grain |
| `aria-hidden` decoration | `.accessibilityHidden(true)` on stars, hearts, leaves, steam, grain, moon, cup and seal | Group each page's words with `.accessibilityElement(children: .combine)` so VoiceOver reads the note as one sentence |
| `lang="ja"` on the Japanese line | Give the Japanese an accessibility label with the full phrase, spoken in Japanese (the `AttributedString` speech-language attribute) | Confirm the exact attribute name when building |
| Tap-anywhere ripple | `SpatialTapGesture` gives the tap point; draw an expanding `Circle().stroke` there | A shader-based water ripple is an optional upgrade (see "What changes") |

## What changes

Here the web approach doesn't translate directly, so the app does it differently.

| Area | On the web | In the app |
|---|---|---|
| **Particles** (90 stars, a stream of hearts, falling leaves) | JS adds a DOM element per particle and removes it after its animation | **One `Canvas` inside `TimelineView(.animation)` per layer.** Seed the particles once with a fixed random seed. Compute each particle's position from the clock, so nothing gets added or removed. Draw the heart/leaf shape once and reuse it. Add `.drawingGroup()` if it gets dense. Separate views per particle are much slower, and SpriteKit or Metal are overkill at these numbers |
| **Off-screen pages** | The browser only runs the page you're on | Both pages exist at once, so **pause the off-screen page's `TimelineView`** (`.animation(minimumInterval:paused:)`). Pause it under Reduce Motion too |
| **Sizing** (`clamp()`, `vw`/`vh`) | Sizes scale with the window width | Fixed sizes tuned for iPhone, scaled with Dynamic Type (`relativeTo:`) and `@ScaledMetric` for spacing. Use `GeometryReader` / `containerRelativeFrame` only for things tied to the screen (moon position, particle area) |
| **Vertical Japanese** (`writing-mode: vertical-rl`) | Built-in CSS | **iOS still has no vertical-text API, iOS 27 included.** Stack the characters in a `VStack`, one per line, upright. Hand-adjust small kana and punctuation if any appear (お茶をください has none). Core Text vertical glyph forms in a `UIViewRepresentable` are the heavier fallback |
| **Narrow-screen layout** (`@media (max-width: 600px)`) | Side by side, stacked when narrow | iPhone portrait is always the "narrow" case, so stack by default. Use `ViewThatFits` if we later want side by side in landscape |
| **Paper grain** (SVG `feTurbulence`) | Generated noise in the browser | Start with a small tiling noise PNG in the Asset Catalog with a blend mode. It's cheaper than a shader. A Metal `colorEffect` noise shader is the alternative. iOS has no built-in grain effect |
| **Water ripple** (optional upgrade) | Expanding CSS ring | A Metal `layerEffect` that actually distorts the page, driven by a keyframe-animated time value, following Apple's WWDC24 visual-effects sample. Precompile it with `Shader.compile(as:)` to avoid a hitch on the first tap |
| **Paging** (two separate URLs) | You only reach a page by knowing its address | `TabView` with `.tabViewStyle(.page)` is simplest and gives page dots for free. A paging `ScrollView` (`.scrollTargetBehavior(.paging)`, `.scrollPosition(id:)`) gives finer control and makes it easy to know which page is showing, for pausing. **Leaning towards the `ScrollView`** |
| **Fonts** | Loaded from Google's servers on every visit | Shipped inside the app, so they work offline |
| **Shipping updates** | Merge to `main` and it's live within seconds | Rebuild in Xcode and reinstall on the phone. TestFlight or App Store updates go through Apple review |
| **Timing** | `setTimeout` / `setInterval` and CSS delays | Animation delays, `.task { try await Task.sleep(...) }` (cancelled automatically when the page goes away) and `TimelineView` |

## Liquid Glass: how it affects this app

Liquid Glass is Apple's translucent material for controls that float over content. It arrived in iOS 26 and was refined in iOS 27. iOS 27 also added a system-wide slider that runs from very clear to fully tinted.

- **It can't be avoided.** Built with Xcode 27, every standard bar, sheet, alert, toggle and tab bar renders as glass. The `UIDesignRequiresCompatibility` opt-out is ignored when building for iOS 27.
- **Our approach: almost no glass.** Apple's guidelines say glass belongs only on functional controls, never on the content itself. Our screens are all content. So:
  - Full-bleed pages with `.ignoresSafeArea()`, and no `NavigationStack` or toolbars, so no glass bars appear.
  - Never put glass on the art, the text or the cup.
  - Page indicator: hide the system dots or draw a tiny custom one with plain fills. (Unverified: whether iOS 27's page dots render as glass.)
  - **If** we add a button later (share, replay): `.buttonStyle(.glass)` with an SF Symbol. Put neighbouring buttons in one `GlassEffectContainer`. Use the `.clear` variant only over bright art, with a 35% dark dim behind it. The night-sky page doesn't need the dim.
- **Accessibility is handled for the glass**, which adapts to Reduce Transparency, Increase Contrast and Reduce Motion by itself. **Our own art animations are not**: they must check Reduce Motion themselves (see above).
- **App icon:** build it in Icon Composer (comes with Xcode) from layers: one background plus foreground shapes on a 1024×1024 canvas. iOS renders six looks: default, dark, clear light/dark, tinted light/dark. Use simple solid shapes kept near the centre. A moon or a tea cup would work well.
- **Test pass before sharing:** Reduce Transparency, Increase Contrast, Reduce Motion, and both ends of the iOS 27 Liquid Glass slider.

Key APIs, all iOS 26+: `glassEffect(_:in:)`, `Glass` (`.regular`, `.clear`, `.identity`, plus `.tint()` and `.interactive()`), `GlassEffectContainer`, `glassEffectID(_:in:)`, `.buttonStyle(.glass)` / `.glassProminent`, `backgroundExtensionEffect()`.

## Unsplash: findings and recommendation

Neither web page uses photos today. This section is here in case we want photo backgrounds in the app.

- **Don't use the official "native SDK" (UnsplashPhotoPicker).**
  - It's UIKit-only, with no SwiftUI API.
  - Its last release was in April 2022 and its last code change in March 2023.
  - It requires putting the secret key inside the app, which breaks Unsplash's own rule that keys stay confidential.
  - The community Swift packages are small and haven't been updated since 2018–2022.
- **Using the live Unsplash API means adding a backend.**
  - The access key has to stay off the phone, which means a small proxy (for example, a Vercel function) that adds it to each request.
  - Rate limits are shared by every user of the app: 50 requests/hour until Unsplash approves a production review. Their docs say 1,000/hour after approval, though older sources say 5,000.
  - The app would also need:
    - visible "Photo by *Name* on Unsplash" credit, linking to both, with `utm_source`/`utm_medium` tags;
    - images shown only from the URLs the API returns (no re-hosting);
    - a call to the photo's `download_location` whenever a photo is used;
    - a network disclosure on the App Privacy label;
    - offline handling.
  - Unsplash also bans "wallpaper applications" and anything copying Unsplash's core experience. A full-screen photo-background app could be read that way, so ask them first if we go this route.
  - Nice extras if we do: imgix size parameters (`w`, `h`, `fit`, `q`, `fm`, `dpr`; keep `ixid`) and a `blur_hash` on every photo for soft placeholders while loading.
- **Recommendation:** if we want photos, **hand-pick a few and bundle them in the app**, credited on screen. That needs no API, no keys and no backend.
  - Confirm the Unsplash License allows bundling a small fixed set (https://unsplash.com/license). It forbids compiling photos into a competing service, which a few backgrounds shouldn't trigger, but this is unverified.
  - Use the live API only if photos need to change without an app update.

## What we're not building

Proposed scope limits for the first version:

- **No web view wrapper.** We won't load the existing site inside a `WKWebView`. The point is a native SwiftUI port.
- **No shared code with the website.** The two versions are kept in sync by hand, using PROJECT_MAP.md as the reference. No cross-platform framework (React Native, Flutter, etc.).
- **No new features.** The app shows the same two notes with the same words, colors and motion. No editing messages, no adding notes, no settings screen.
- **No live Unsplash API, and no Unsplash SDK.** See above. Bundled, credited photos are the only photo option in scope, and only if we decide we want photos at all.
- **No backend, accounts, sync or analytics.** The web version has none of these either, and keeping it that way is why we're not using the Unsplash API.
- **No glass chrome.** No navigation bars, toolbars or tab bars, and glass only on a button if one is ever added.
- **No notifications, widgets or Live Activities.** A 10 pm bedtime reminder or a home-screen widget would be natural follow-ups, but they're out of scope for now.
- **No iPad-specific layout, Mac, Apple Watch or Android.** SwiftUI will run on iPad, but we won't design for it.
- **No App Store release in this phase.** The target is running on our own iPhone from Xcode, then TestFlight if we want to share it. Publishing is a separate decision.
- **No changes to the website** as part of this work.

## Open questions

1. Should the app open on "Hello, Sam" every time, or pick a note by time of day (for example, the bedtime note after 9 pm)?
2. Page indicator: none, or a tiny custom one?
3. Should the tea ripple add a light haptic tap? Should it be the simple ring or the shader ripple?
4. Do we want photos at all? If yes, which ones, and on which page?
5. Is an App Store release ever the goal? That affects naming, the icon and the Apple Developer account ($99/year).
6. The tea page has a pending change to a dark background (PR #4). The app should follow whichever version is merged.

## Suggested order of work

1. Create the Xcode project in `ios/` (iOS 26 target), bundle both fonts, add the color tokens to the Asset Catalog.
2. Port "Green tea, please" first, since it covers the most techniques: layout, cup shape, vertical Japanese, entrance timeline, seal stamp, then the ripple and leaves.
3. Port "Hello, Sam": gradient (try `MeshGradient`), moon, text reveal, heartbeat, then stars and hearts in `Canvas`.
4. Add paging between the two pages and pause the off-screen page.
5. Accessibility pass on both: Reduce Motion, VoiceOver, Reduce Transparency, Increase Contrast.
6. Design the icon in Icon Composer.
7. Run it on a real iPhone and compare it side by side with the website.

## Research findings (2026-10-04)

Summary of the three research passes behind this plan. Items marked *unverified* weren't confirmed against an official source.

**Platform**
- iOS 27.0.1 is current (released Sept 2026). About 79% of active iPhones were on iOS 26 in June 2026.
- App Store uploads have had to use Xcode 26 / the iOS 26 SDK since April 2026. Press reports say Xcode 27 becomes required around April 2027 (*unverified*).
- Sources: https://developer.apple.com/news/upcoming-requirements/ and https://www.macrumors.com/2026/06/09/ios-26-adoption-stats-wwdc/

**SwiftUI patterns**
- `Canvas` + `TimelineView` for particles (iOS 15).
- `KeyframeAnimator` / `PhaseAnimator` and the custom `Transition` protocol (iOS 17).
- `TextRenderer`, `MeshGradient`, `Shader.compile` (iOS 18).
- `@Animatable` (iOS 26).
- WWDC26 added no new animation APIs.
- Third-party speed figures for Canvas vs. many views are *unverified*. A reported iOS 27 vertical-text change in TextKit (`NSTextLayoutOrientationProvider` on iOS) is *unverified*, so plan on the `VStack` approach.
- Sources: https://developer.apple.com/videos/play/wwdc2024/10151/ ("Create custom visual effects with SwiftUI"), https://developer.apple.com/videos/play/wwdc2023/10157/, https://developer.apple.com/documentation/swiftui/applying-custom-fonts-to-text and https://developer.apple.com/documentation/swiftui/scrolltargetbehavior

**Liquid Glass**
- The APIs listed above, the HIG materials guidance, the `UIDesignRequiresCompatibility` key being ignored from iOS 27, and Icon Composer.
- *Unverified:*
  - the minimum OS for `.buttonStyle(.glass(_:))`;
  - whether iOS 27 added new `Glass` variants;
  - how the paging dots look on iOS 27;
  - press claims about iOS 27 contrast changes.
- Sources:
  - https://developer.apple.com/documentation/swiftui/view/glasseffect(_:in:)
  - https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass
  - https://developer.apple.com/documentation/bundleresources/information-property-list/uidesignrequirescompatibility
  - https://developer.apple.com/design/human-interface-guidelines/materials
  - https://developer.apple.com/design/human-interface-guidelines/app-icons
  - https://developer.apple.com/videos/play/wwdc2026/269/

**Unsplash**
- SDK status, API terms, attribution, hotlinking, download tracking and rate limits are covered above.
- *Unverified:* the production rate limit (1,000 vs. 5,000/hour), how the production review works, and whether bundling a few photos fits the license.
- Sources:
  - https://github.com/unsplash/unsplash-photopicker-ios
  - https://unsplash.com/documentation
  - https://help.unsplash.com/en/articles/2511245-unsplash-api-guidelines
  - https://help.unsplash.com/en/articles/2511315-guideline-attribution
  - https://unsplash.com/license
