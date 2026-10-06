<div align="center">

<img src="assets/branding/app-icon.webp" alt="Minesweeper app icon" width="110" />

# Minesweeper

**A classic Minesweeper puzzle engineered for four platforms: one Flutter app for Android and iOS, a separate Flutter app for Android TV driven by a D-pad, and a native SwiftUI app for Apple TV driven by the Siri Remote.**

**Android phone and tablet · iOS · Android TV · Apple TV**<br/>
Flutter · Dart · Kotlin · Swift · SwiftUI · Firebase · Play Billing · StoreKit

![Flutter](https://img.shields.io/badge/Flutter-Dart-02569B?logo=flutter&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-SwiftUI-F05138?logo=swift&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-native%20shims-7F52FF?logo=kotlin&logoColor=white)
![Android](https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-000000?logo=apple&logoColor=white)
![Android TV](https://img.shields.io/badge/Android%20TV-D9822B?logo=androidtv&logoColor=white)
![Apple TV](https://img.shields.io/badge/Apple%20TV-1D1D1F?logo=apple&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?logo=firebase&logoColor=black)

<table>
  <tr>
    <td><img src="assets/screenshots/mobile/mobile-home-menu.webp" width="300" alt="Home menu with Play, difficulty selector, Settings, How to Play, Hint Packs and the custom game unlock" /></td>
    <td><img src="assets/screenshots/mobile/mobile-gameplay-beginner.webp" width="300" alt="Empty Beginner board with timer, hints, score, flag counter and control hints" /></td>
  </tr>
</table>
<sub>Real screenshots from the Android build, cropped to the app content (system status bar and ad-placeholder strips removed), captured with sample data, metadata removed. No TV screenshots are published in this repository.</sub>

</div>

---

## Get the App

<p>
  <a href="https://play.google.com/store/apps/details?id=com.futurewatch.minesweeper&hl=en"><img alt="Get it on Google Play" src="https://img.shields.io/badge/Google%20Play-Phone%2C%20tablet%20and%20Android%20TV-414141?style=for-the-badge&logo=googleplay&logoColor=white" /></a>
  <a href="https://apps.apple.com/pk/app/minesweeper-puzzle-game/id6766427737"><img alt="Download on the App Store" src="https://img.shields.io/badge/App%20Store-iPhone%2C%20iPad%20and%20Apple%20TV-0D96F6?style=for-the-badge&logo=apple&logoColor=white" /></a>
</p>

## Product at a Glance

| | |
|---|---|
| **What it is** | Classic Minesweeper: reveal every safe cell using adjacent-mine counts and flags |
| **Boards** | Three presets (Beginner, Intermediate, Expert) plus custom boards, unlocked by purchase or, on the ad-supported apps, for one game through a rewarded ad |
| **Game systems** | Flagging, hints, a bomb-recovery continue mechanic, live scoring, top-10 high scores per preset |
| **Platforms** | Android phone and tablet, iOS, Android TV, Apple TV |
| **Codebases** | Three: a Flutter app (Android and iOS), a Flutter fork for Android TV, a native Swift/SwiftUI app for Apple TV |
| **Input** | Touch with pinch/pan zoom, D-pad key events, Siri Remote press tracking |
| **Platform services** | Native Play Billing and StoreKit bridges, remote ad kill-switches and a consent flow (mobile), analytics and crash reporting (both Flutter apps) |
| **Localization** | 15 languages on the mobile app; the TV apps are English only |
| **Code size** | About 30,000 lines of Dart in the mobile app, 15,700 in the Android TV app (`lib/`, excluding generated localization) and 4,400 lines of Swift in the Apple TV app |

## Showcase

<table>
  <tr>
    <td align="center"><img src="assets/screenshots/mobile/mobile-how-to-play.webp" width="260" alt="How to Play screen, page one of six, with Previous and Next buttons"/><br/><sub><b>How to Play</b><br/>Six-page in-app tutorial</sub></td>
    <td align="center"><img src="assets/screenshots/mobile/mobile-premium-paywall.webp" width="260" alt="Premium dialog heading and description"/><br/><sub><b>Premium dialog</b><br/>Heading and description</sub></td>
    <td align="center"><img src="assets/screenshots/mobile/mobile-splash-screen.webp" width="260" alt="Splash screen with the game title and a loading indicator"/><br/><sub><b>Splash</b><br/>Shown while startup stages complete</sub></td>
  </tr>
</table>

## Four Platforms, Three Codebases

The headline engineering decision is how one game design is delivered to four very different devices without pretending they share an input model. A phone is touched, an Android TV is driven by a D-pad, and an Apple TV by a Siri Remote that is part trackpad, part button. Game rules are identical everywhere; input, layout and services are native to each.

<p align="center">
  <img src="assets/diagrams/platform-overview.svg" alt="Platform overview: four platforms built from three codebases with one shared game design" width="900" />
</p>

<table>
  <tr>
    <td width="50%" align="center"><img src="assets/artwork/android-tv-launcher-banner.webp" width="400" alt="Android TV launcher banner artwork"/><br/><sub><b>Android TV</b>: launcher banner artwork shipped with the TV app (artwork, not a screenshot)</sub></td>
    <td width="50%" align="center"><img src="assets/artwork/apple-tv-icon-back-layer.webp" width="400" alt="Back layer of the layered Apple TV app icon"/><br/><sub><b>Apple TV</b>: back layer of the layered app icon (artwork, not a screenshot)</sub></td>
  </tr>
</table>

Android TV and Apple TV are represented here by artwork and original diagrams only. There are no TV gameplay screenshots in this repository.

| | Android and iOS | Android TV | Apple TV |
|---|---|---|---|
| **Codebase** | One Flutter app | Separate Flutter fork | Native Swift / SwiftUI |
| **Primary input** | Touch: tap-up reveal, long-press flag, pinch/pan zoom | D-pad keys: select reveals, hold-select flags | Siri Remote: select press tracker, move commands |
| **Focus model** | Direct touch targets | Hand-managed cursor over a keyboard-style key handler | Custom cursor that bypasses the platform focus engine |
| **Back / exit** | Android system back opens the pause menu | Back key pauses with exit confirmation | Menu button pauses with exit confirmation |
| **Orientation** | Portrait | Landscape | Landscape |
| **Ads** | Mediation SDK, remote kill-switches | Ads SDK, rewarded and interstitial | No ad SDK in the app |
| **Purchases** | Play Billing / StoreKit via native bridges | Play Billing via native bridge | StoreKit 2 |
| **Haptics** | Yes, user toggle | No | No |
| **Localization** | 15 languages | English | English |
| **Telemetry** | Analytics, crash reporting, performance traces | Analytics, crash reporting | None |

Full comparison: [docs/MOBILE_VS_TV.md](docs/MOBILE_VS_TV.md).

## Engineering Highlights

### A fair, lazily generated board
- **Problem:** the first tap in Minesweeper must never lose, and a board generated at game start can be inspected or solved before the first move.
- **Approach:** the board stays empty and mine-free until the first reveal. Mines are then placed by rejection sampling over every cell outside a 3x3 exclusion zone around the tapped cell. Custom boards cap the mine count so the exclusion zone always leaves room.
- **Outcome:** the first reveal is always a zero-cell that cascades open. The same algorithm runs in Dart (two apps) and Swift, where the port adds an attempt cap as a safety net.

### Flood fill that cannot overflow the stack
- **Problem:** a cascading reveal on a board of a few hundred cells must be safe, predictable and easy to reason about.
- **Approach:** an iterative, stack-based fill with a visited set, living in a pure logic class that returns a fresh copy of the grid (cells are immutable value objects) plus the count of newly revealed cells. Flags block expansion.
- **Outcome:** identical behaviour across three apps, with no recursion and no hidden mutation between the rules and the UI.

### Three input models, one intent vocabulary
- **Problem:** the same actions (reveal, flag, move, pause) must feel native on touch, a D-pad and a Siri Remote.
- **Approach:** touch reveals on tap-up, not tap-down, so a long-press or pan cannot detonate a mine; Android TV runs a hand-managed focus cursor with a hold timer; Apple TV uses a UIKit press tracker feeding SwiftUI, so the board does not depend on the focus engine.
- **Outcome:** reveal, flag and pause map the same way on every platform. See the [input pipeline](docs/ARCHITECTURE.md#input-pipeline).

<p align="center">
  <img src="assets/diagrams/input-pipeline.svg" alt="Input pipeline: touch, D-pad and Siri Remote inputs become intents, then game logic, state and feedback" width="900" />
</p>

### Bomb recovery without corrupting game state
- **Problem:** letting a player continue after a mine (through a rewarded flow) must not break scoring, flag counts or win detection.
- **Approach:** a loss stops the timer but keeps mines hidden. Recovery converts the exploded cell into a flag, restarts the timer, applies a score penalty, is capped per game and guarded against being granted twice.
- **Outcome:** the win condition stays valid because flagged cells count as unrevealed. Recovery is wired to the reward flow on Mobile and Android TV.

### Ads that fail closed
- **Problem:** ad placements need a remote off switch without an app release, and a stalled network must never look like a launch failure.
- **Approach:** per-placement and global kill-switches read from a single remote configuration document, with no ad until a confirmed fetch says yes. Startup is staged (storage and Firebase init, native entitlement sync, consent, remote config, ad load), each stage bounded by a timeout.
- **Outcome:** ads default to off, premium users skip the ad path entirely, and every stage is measured with performance traces.

### More
| Area | What was built |
|---|---|
| **Billing** | Native Kotlin (Play Billing) and Swift (StoreKit) managers behind platform channels on the Flutter apps; StoreKit 2 directly in the Apple TV app; restore and consumable handling |
| **Audio** | No audio asset files: tones are synthesised at startup and played through SoundPool, AVAudioPlayer or an AVAudioEngine node |
| **Haptics** | Every game sound maps to a haptic pattern, behind a user toggle (mobile only) |
| **Lifecycle** | Pause on backgrounding with a latch that fixes a stranded-pause event sequence |
| **Telemetry** | In-memory aggregation of touch events into one summary per game; throttled TV focus events; payload sanitiser and a pre-init queue |
| **Localization** | 15 languages on mobile via generated localizations |
| **Tests** | Mobile: widget and gesture tests, lifecycle, ads, analytics, billing parsing. Android TV: analytics. Apple TV: none. No dedicated unit tests for core game logic |

Details and trade-offs: [docs/ENGINEERING_CHALLENGES.md](docs/ENGINEERING_CHALLENGES.md).

## Architecture

```mermaid
flowchart TD
    subgraph Inputs
        TO["Touch: tap-up, long-press, pinch and pan"]
        DP["Android TV D-pad key events"]
        SR["Apple TV Siri Remote presses"]
    end
    TO --> IN["Intents: reveal, flag, move, hint, pause"]
    DP --> IN
    SR --> IN
    IN --> GL["Pure game logic<br/>lazy generation, flood fill, flags, win and loss, score"]
    GL --> ST["State: Riverpod notifier or ObservableObject"]
    ST --> UI["UI: Flutter widgets or SwiftUI views"]
    ST --> FX["Audio synthesis, haptics on mobile"]
    ST --> PR[("Local persistence<br/>high scores, settings, hint balance")]
    SV["Services: ads, consent, billing bridges,<br/>analytics, remote config"] <--> ST
    NB["Native bridges: Kotlin Play Billing, Swift StoreKit, sound playback"] <--> SV
```

More diagrams (per-platform layers, startup, billing, shared versus platform-specific code): [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Honest Trade-offs

- **Duplication.** The two Flutter apps share a lineage but not a package: 59 source paths exist in both, 10 are byte-identical (including the game-rules class and the cell and game-state models) and 49 have diverged. The Apple TV logic is a hand port. A fix to shared rules must be applied up to three times.
- **Test depth.** There are widget tests for gestures and lifecycle, plus ads, analytics and billing-parsing tests on mobile, but no dedicated unit tests for board generation, flood fill or scoring, and no test target in the Apple TV app.
- **Client-side balances.** Hint balances and entitlement flags are stored on the device. Server-side entitlement verification would be a natural next step.
- **Not implemented:** game-state restore after the app is killed, accounts or cloud sync, achievements, chording, and accessibility semantics for the board.

## Technology Stack

| Layer | Technology |
|---|---|
| Mobile and Android TV UI | Flutter / Dart 3 |
| Apple TV UI | Swift, SwiftUI, UIKit interop for the remote |
| UI state | Riverpod (Flutter); `ObservableObject` and Combine (Apple TV) |
| Navigation | `go_router` (Flutter); a small screen-stack manager (Apple TV) |
| Local storage | Hive and shared preferences (Flutter); `UserDefaults` (Apple TV) |
| Backend services | Firebase Analytics, Crashlytics, Performance, Messaging and remote configuration on mobile; Analytics and Crashlytics on Android TV |
| Billing | Kotlin Play Billing; Swift StoreKit (iOS bridge) and StoreKit 2 (Apple TV) |
| Ads | Mediation SDK (mobile), ads SDK (Android TV), none in the Apple TV app |
| Audio | Procedurally synthesised PCM/WAV, platform audio players |
| Tooling | R8, baseline profile and macrobenchmark modules (Android TV) |

## Documentation

| Read | For |
|---|---|
| [Product overview](docs/PRODUCT_OVERVIEW.md) | What the game does and how each platform plays |
| [Technical case study](docs/TECHNICAL_CASE_STUDY.md) | Game logic, input, services and platform engineering end to end |
| [Architecture](docs/ARCHITECTURE.md) | Layers, data flow and component diagrams |
| [Mobile vs TV](docs/MOBILE_VS_TV.md) | Four-platform comparison: input, focus, performance, reuse |
| [Engineering challenges](docs/ENGINEERING_CHALLENGES.md) | Problem, approach, outcome and trade-offs |
| [Privacy and source](docs/PRIVACY_AND_SOURCE.md) | What is and is not published |

## Source Code

This repository is a portfolio showcase for a production application. Production source code is maintained privately. See [RIGHTS.md](RIGHTS.md) for usage terms.
