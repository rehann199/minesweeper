# Technical Case Study

This document walks through how Minesweeper is built across four platforms. Descriptions are conceptual; identifiers and tuning values are not published.

## 1. Scope and shape

| | Mobile app | Android TV app | Apple TV app |
|---|---|---|---|
| Platforms | Android phone and tablet, iOS | Android TV | Apple TV |
| Language and UI | Dart / Flutter, Kotlin and Swift shims | Dart / Flutter, Kotlin shims | Swift / SwiftUI, UIKit interop |
| Approximate size | About 30,000 lines of Dart | About 15,700 lines of Dart | About 4,400 lines of Swift |

Sizes count `lib/` Dart files excluding generated localization, and all Swift source files. Three codebases deliver four platforms: the mobile Flutter app builds for both Android and iOS, the Android TV app is a separate Flutter project, and the Apple TV app is native.

The Flutter apps share a lineage. Of the 59 Dart source paths that exist in both, 10 are byte-identical, including the game-rules class, the cell and game-state models, and the high-score model and provider. The other 49 have diverged, almost entirely in input, layout, ads, billing surface and platform services. The Apple TV game logic is a line-by-line port to Swift. The result is identical rules everywhere at the cost of duplicated UI and services.

## 2. Game logic

The rules live in a pure, stateless logic layer operating on an immutable grid of cells. Every operation takes a grid and returns a new one (plus counts), so the state layer can compare and replace it atomically and the rules have no UI dependencies.

### Lazy generation and first-move safety
The notifier first builds an all-hidden, mine-free board with status "not started". On the first reveal the tapped cell and its eight neighbours form an exclusion set. Mines are placed by rejection sampling: draw uniform random cell indices, skip excluded or already-chosen positions, stop at the target count. Adjacent counts are then computed in one pass. Because the whole 3x3 block is mine-free, the first reveal is always a zero-cell and cascades open. Flagging before the first reveal is rejected. For custom boards the mine cap leaves room for the largest possible exclusion zone, so the sampling loop terminates for configurations that pass the setup checks. The Swift port adds an attempt cap and clamps mines to the available cells as a defensive measure; the Dart version relies on those checks.

### Iterative flood fill
A reveal on a zero-cell runs a stack-based fill: pop a position, skip it if already visited, out of bounds, revealed, flagged or a mine, reveal it, and if its count is zero push its eight neighbours (duplicates are allowed; they are filtered on pop). Numbered cells reveal but do not expand; flagged cells block expansion. The routine returns the newly revealed count, which feeds progress and score. There is no recursion. The visited set is keyed by strings in Dart and by integer indices in Swift, a small improvement made during the port.

### Flags, win and loss
Flag toggling is disallowed on revealed cells and before the first reveal. The win check counts cells that are still hidden or flagged and compares with the mine total, so correct flags are not required to win. A revealed mine marks that cell exploded, sets status to lost and stops the timer, but deliberately leaves the other mines hidden so that recovery is possible; a separate function reveals all mines for the final presentation. There is no chording.

### Hints, recovery and score
A hint selects a random hidden, unhinted, unflagged mine and marks it as hinted without revealing it. A recovery converts the exploded mine into a flag, returns the game to playing, restarts the timer, increments a per-game counter capped at two and applies a score penalty. Score combines revealed cells scaled by a difficulty multiplier, a time bonus that decays toward zero over a few minutes, and penalties for hints and recoveries, clamped to a non-negative range. For custom boards the multiplier derives from mine density within bounds. Exact constants are intentionally not published.

### Timer and persistence
A one-second periodic timer increments elapsed time only while the status is playing and stops on pause, loss and win. High scores are stored only for wins on preset difficulties, ten per preset, ordered by score then time. The Flutter apps persist them in Hive boxes with settings in shared preferences; the Apple TV app uses JSON in `UserDefaults`. An in-progress game is not persisted.

## 3. Input

The input layer is where the platforms diverge most. The shared contract is the same: reveal, flag, move, hint, pause. See the diagram in [ARCHITECTURE.md](ARCHITECTURE.md#input-pipeline).

**Touch (Android and iOS).** Each cell uses a gesture detector with tap-up for reveal and long-press for flag. Tap-down is deliberately not used: it fires while the gesture arena is still undecided, which previously caused hold-to-flag to detonate a mine first and a pan that began on a cell to reveal it. A widget test pins that behaviour. The board sits in an interactive viewer for pinch zoom and pan, with the tap position captured in global coordinates so the blast animation starts at the right place under any zoom. System back is intercepted to show the pause menu. Touch statistics are accumulated in memory and emitted as a single summary event per game.

**Android TV (D-pad).** A single root focus node receives key events. The cursor lives in two areas, the grid and a small header row, and moves with arrow keys, clamping at edges; moving up from the top row enters the header and moving down returns. Select-down starts a hold timer of roughly four tenths of a second: expiry flags, a release before expiry reveals, and a release after expiry is suppressed so it does not also reveal. Back and escape are always handled: they pause and ask for confirmation, and dialogs suppress game input. Other screens use separate handlers that ignore key-repeat events.

**Apple TV (Siri Remote).** SwiftUI cannot sensibly manage hundreds of individually focusable cells, and a plain button offers no hold gesture on select. The app instead embeds a transparent UIKit view that is always focusable and consumes select presses: a short press reveals, a hold of about half a second flags, and the following release is suppressed. SwiftUI move, play/pause and exit commands supply cursor movement and pause. The board highlight is a custom row and column cursor rather than a focus-engine target, and a blocked flag disables input under modal dialogs. The same abstraction is reused across other screens.

## 4. Feedback: audio and haptics

There are no bundled audio files. Eight short sounds (navigate, select, back, error, reveal, flag, mine, win) are synthesised as PCM at startup. On Android they play through a native sound pool, on iOS through an audio player behind a platform channel, and on Apple TV through an audio engine player node. On mobile every game sound is paired with a haptic pattern (light for reveal, medium for flag, heavy for a mine, a double pulse for a win), gated by a setting and failing silently on devices without support.

## 5. Services

**Startup.** The mobile app shows an immediate native splash, then runs staged initialisation: preferences and Firebase (bounded by a timeout), a native entitlement sync (shorter timeout, falling back to the cached value), consent, remote ad configuration, ad SDK init and an on-demand app-open ad. A hard cap on the whole gate pauses while consent prompts are on screen, premium users skip the ad path, and each stage is wrapped in performance traces. The design addresses slow-network stalls: nothing blocks the user indefinitely.

**Ads.** Mobile routes placements (startup, banners, interstitials, rewarded) through a mediation SDK; Android TV uses an ads SDK for rewarded and timed interstitial placements; the Apple TV app has no ad SDK. A cooldown spaces interstitials and premium users never see ads. Placement and global kill-switches are booleans in one remote document with a live listener; the gate fails closed, so no ad appears unless a confirmed fetch enables it.

**Billing.** The Flutter apps wrap native managers over method and event channels: Kotlin for Play Billing (consume hint packs, acknowledge unlocks, query subscriptions and one-time products on connect to restore, detect free-trial phases) and Swift on iOS with a payment-queue observer. The Apple TV app uses StoreKit 2 directly, with verified results only, a transaction-update listener and a restore sync. Entitlement is re-read from the store at launch; a negative result from a network error never locks a feature the user already owns. Hint balances are client-side.

**Telemetry.** Events are sanitised to fit analytics naming and payload limits and queued until Firebase is ready. Touch and focus streams are aggregated or throttled to avoid flooding. Crash reporting and analytics collection are disabled by default and enabled after initialisation. The Apple TV app has no analytics or crash SDK.

**Other mobile services.** Firebase Messaging topics (all users, platform, version, language), a notification permission pre-prompt with throttling, an in-app update flow and a consent flow for ads.

## 6. Performance

Boards are non-lazy column-and-row layouts of up to a few hundred widgets (an Expert board has 480 cells), scaled to fit the screen with cell size clamped to a legible range. On mobile each cell has its own repaint boundary; the Android TV cell does not. Every reveal or flag copies the grid, an O(cells) allocation that is simple and safe at these sizes but not optimal. A Firebase trace measures board generation on mobile. A lifecycle observer pauses the game on backgrounding, with a latch fixing an event sequence that used to strand the game in a paused state. The Android TV project carries baseline-profile and macrobenchmark modules; no measured numbers are published here.

## 7. Testing

The mobile app has 24 test files: widget and gesture tests (hold-to-flag, lifecycle pause and resume, blast overlay, recovery countdown, input shield, localization, tutorial coach mark), ad configuration, throttling and lifecycle logging, analytics sanitising and coverage, billing product parsing, app update and a touch-statistics tracker. The Android TV app has one analytics test file. The Apple TV app has no test target. There are no dedicated unit tests for board generation, flood fill, scoring or win detection, which is the most notable gap given that the logic is duplicated.

## 8. Future improvements

- Extract the game rules into a shared package with unit and property tests (first-move safety, flood-fill coverage, score bounds), and port-test the Swift version against the same vectors.
- Consolidate the two Flutter apps behind a shared package with thin per-platform shells.
- Persist in-progress games so a killed app can resume.
- Add semantic labels for board cells and keyboard-accessible alternatives.
- Verify entitlements server-side instead of trusting client-held balances.
- Add per-cell repaint isolation to the TV board and measure large boards.
