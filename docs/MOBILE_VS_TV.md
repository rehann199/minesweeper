# Mobile vs Android TV vs Apple TV

Minesweeper ships on four platforms from three codebases. This document compares them.

## Relationship between the apps

| Codebase | Platforms | Technology |
|---|---|---|
| Mobile app | Android phone and tablet, iOS | Flutter / Dart, with Kotlin and Swift shims |
| Android TV app | Android TV | A separate Flutter project, with Kotlin shims |
| Apple TV app | Apple TV | Native Swift and SwiftUI, with a small UIKit bridge |

The two Flutter apps share a lineage rather than a package. Of the 59 Dart source paths that exist in both (generated localization excluded), 10 are byte-identical, including the game-rules class, the cell and game-state models and the high-score model and provider. The other 49 diverge. The Android TV app is smaller (about 15,700 lines of Dart against about 30,000 for mobile), because it omits consent, notifications, in-app update, localization and the ad-mediation stack. The Apple TV app (about 4,400 lines of Swift) re-implements the game logic as a hand port.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/mobile/mobile-gameplay-beginner.webp" width="260" alt="Mobile gameplay board"/><br/><sub><b>Mobile</b>: Beginner board with the touch hint line (real screenshot from the Android build)</sub></td>
    <td align="center"><img src="../assets/artwork/android-tv-launcher-banner.webp" width="420" alt="Android TV launcher banner artwork"/><br/><sub><b>Android TV</b>: launcher banner artwork (artwork, not a screenshot)</sub></td>
    <td align="center"><img src="../assets/artwork/apple-tv-icon-back-layer.webp" width="420" alt="Apple TV layered icon back layer"/><br/><sub><b>Apple TV</b>: back layer of the layered app icon (artwork, not a screenshot)</sub></td>
  </tr>
</table>

No TV gameplay screenshots exist in this showcase. TV behaviour is described from the source and illustrated by diagrams.

## Comparison

| Dimension | Android and iOS | Android TV | Apple TV |
|---|---|---|---|
| UI toolkit | Flutter | Flutter | SwiftUI with UIKit interop |
| Manifest or device posture | Standard launcher activity | Leanback required, touchscreen optional, TV launcher entry and banner | Apple TV target (tvOS), layered icon and top-shelf artwork |
| Orientation | Portrait only | Landscape | Landscape |
| Expert board shape | Tall (30 rows by 16 columns) | Wide (16 by 30) | Wide (16 by 30) |
| Reveal | Tap (on release) | Select short press | Select click |
| Flag | Long-press | Select hold, about 0.4 s | Select hold, about 0.5 s |
| Move | Direct touch | Arrow keys move a cursor | Swipe or directional presses move a cursor |
| Zoom and pan | Pinch and drag | No (board fits the screen) | No (board fits the screen) |
| Back | System back opens pause menu | Back or escape pauses with confirmation | Menu button pauses with confirmation |
| Focus model | Touch targets, plus a cursor highlight | Hand-managed cursor, two areas (grid, header) | Custom cursor, press tracker, bypasses focus engine on the board |
| State management | Riverpod | Riverpod | ObservableObject and Combine |
| Navigation | `go_router` with custom transitions | `go_router` | Screen-stack manager |
| Haptics | Yes, user toggle | No | No |
| Audio | Synthesised tones via native players | Same design via a native sound pool | Synthesised tones via an audio engine |
| Localization | 15 languages | English | English |
| Ads | Mediation SDK, remote kill-switches | Ads SDK, rewarded and timed interstitial | No ad SDK |
| Purchases | Play Billing and StoreKit via native bridges | Play Billing via native bridge | StoreKit 2 |
| Persistence | Hive and shared preferences | Hive and shared preferences | `UserDefaults` |
| Analytics and crashes | Firebase Analytics, Crashlytics, Performance | Firebase Analytics, Crashlytics | None |
| Push and update | Messaging topics, in-app update | None | None |
| Lifecycle pause | Yes, with resume latch | Yes | No background handling |
| Tests | Widget, gesture, lifecycle, ads, analytics, billing parsing | One analytics test file | No test target |
| Release tooling | R8, ABI filters | R8, ABI filters, baseline profile, macrobenchmark modules | Xcode project, local StoreKit configuration |

## Interaction differences

### Mobile (touch)
Each cell is a gesture target: tap-up reveals and long-press flags. Tap-down is deliberately avoided because it fires while the gesture arena is undecided, which previously let a long-press or pan reveal the cell. The board sits in an interactive viewer (pinch zoom, drag pan) so large boards remain tappable; the footer text tells the player this. Touch statistics (taps, long-presses, blocked-touch reasons, zoom and pan counts) are aggregated into a single event per game. Haptics echo each sound.

### Android TV (D-pad)
There is no pointer, so a cursor is simulated. One root focus node handles key events. Arrow keys move the highlighted cell; moving up from the top row enters a header row (hint, pause) and moving down returns. Holding select for about four tenths of a second flags, a shorter press reveals, and the release after a hold is swallowed. Back and escape always pause with confirmation. A small focus-telemetry helper throttles focus-move logging and de-duplicates select presses. Each cell still carries tap and long-press handlers, so pointer input would work, but the manifest declares touchscreen optional.

### Apple TV (Siri Remote)
Touch-surface swipes arrive as move commands, a click as a select press. A transparent, always-focusable UIKit view tracks select presses with a timer of about half a second to tell tap from hold; a completed hold suppresses the release. Play/pause and the Menu button are mapped to pause and exit confirmation. The board highlight is a custom cursor, because the SwiftUI focus engine is a poor fit for hundreds of cells. On-screen footer hints list the controls. Modal dialogs block game input through a flag on the tracker.

## UI and screen differences

- Mobile is portrait with a touch layout and an optional zoomed board. The TV apps are landscape layouts for a living-room display and scale the board to fit the screen without zoom.
- The mobile app presents onboarding (terms, language, tutorial), a settings screen with language and haptic toggles, hint-pack and premium dialogs, an in-app screen linking to other apps by the same developer, and update and notification prompts. The TV apps have a leaner set of screens: Android TV has home, difficulty selection, game, game over, how to play, settings and hint packs, with custom-unlock dialogs; the Apple TV app uses dialogs for hints and custom unlock.
- The TV apps draw their own focus states (highlighted buttons and a board cursor) rather than relying on touch feedback.

## Navigation and focus

Mobile navigation is touch-driven with `go_router`, custom transitions and an intercepted back gesture. Android TV uses per-screen key handlers that ignore key repeat and keep a focused index. Apple TV uses a hand-rolled screen-stack manager, with a shared input modifier giving every screen the same press, move and back semantics. On both TV apps the board cursor is application-managed rather than platform-managed, which gives the same feel on both but means standard focus traversal is not used for cells.

## Performance considerations

- All boards are laid out as rows of widgets (up to 480 cells for Expert) and scaled to fit. Mobile isolates each cell in its own repaint layer and clamps cell size; the Android TV cell variant does not use per-cell repaint isolation, a candidate for measurement.
- Every reveal or flag copies the grid. At these sizes it is fine, and it keeps the logic simple and immutable.
- Mobile measures board generation and startup stages with performance traces; the Android TV project carries baseline-profile and macrobenchmark modules. No measured figures are published here.
- The Apple TV app has no third-party dependencies and no network SDKs, which keeps its startup path short by construction; this is a design observation rather than a measured figure.

## Code reuse and platform-specific implementation

**Reused concepts:** the rules (generation, flood fill, flags, win and loss, scoring, hints, recovery), the visual language, the sound set and its synthesis approach, the intent vocabulary, difficulty presets and custom limits.

**Reused code:** only between the two Flutter apps, and only a small fraction (10 byte-identical files out of 59 shared paths, including all game rules).

**Platform-specific:** input handling, layouts, ad stack, consent, notifications and update (mobile), localization (mobile), billing surfaces, telemetry and persistence mechanics.

## Honest trade-offs

- Three implementations mean a rule change can need three edits. A shared rules package with unit tests, and a consolidated Flutter package for models and services with thin platform shells, would be the natural next step.
- Choosing native SwiftUI for Apple TV gave first-class remote handling and a small footprint, but it forfeited code sharing with the Flutter apps and required a hand port with no automated check that the two implementations agree.
- The bespoke cursors on both TV apps are tuned to the game but are invisible to platform accessibility tooling; accessibility semantics for the board are not implemented on any platform.
- None of the apps restore an in-progress game. The TV apps also lack localization, push notifications and in-app update prompts.
- The bomb-recovery reward flow is wired on Mobile and Android TV only. The Apple TV post-loss dialog offers restart or exit.
- Purchases are handled independently per platform; entitlements are not shared between iOS and Apple TV.
