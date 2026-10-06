# Product Overview

Minesweeper is a classic logic puzzle: a grid of hidden cells, some of which hold mines. Revealing a safe cell shows how many of its eight neighbours are mines; the player uses those numbers and flags to clear every safe cell without detonating a mine. The product is free to play with optional in-app purchases and ads, and is published for Android phones and tablets, Android TV, iOS (iPhone and iPad) and Apple TV.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/mobile/mobile-home-menu.webp" width="260" alt="Home menu"/><br/><sub><b>Home</b>: highest score, difficulty selector, entry points</sub></td>
    <td align="center"><img src="../assets/screenshots/mobile/mobile-gameplay-beginner.webp" width="260" alt="Beginner board before the first reveal"/><br/><sub><b>Board</b>: Beginner, before the first reveal</sub></td>
    <td align="center"><img src="../assets/screenshots/mobile/mobile-how-to-play.webp" width="260" alt="How to Play, page one of six"/><br/><sub><b>How to Play</b>: page 1 of 6</sub></td>
  </tr>
</table>
<sub>Real screenshots from the Android build, cropped to the app content, captured with sample data, metadata removed.</sub>

## Core play

- **Reveal** a cell. A number shows adjacent mines; a zero opens its neighbours automatically (a cascade).
- **Flag** a cell you believe holds a mine. A flagged cell is protected: tapping it does nothing until the flag is removed.
- **Win** by revealing every non-mine cell. **Lose** by revealing a mine.
- The flag counter shows flags placed against the mine total (for example 0/10) and can exceed it if a player over-flags.
- The first reveal is always safe. The board is generated at that moment, away from the tapped cell, so the opening move cannot lose and always opens an area.

## Difficulty and custom boards

| Mode | Description |
|---|---|
| Beginner | A small square board with a low mine count |
| Intermediate | A larger square board with a moderate mine count |
| Expert | The largest preset; shaped to fit the device (tall on portrait phones, wide on TV screens) |
| Custom | The player chooses width, height and mine count within bounded limits; unlocked by purchase, or for a single game through a rewarded ad on the Flutter apps |

Custom boards are constrained so they stay playable and generatable: dimensions have minimum and maximum limits, the mine count has a minimum and a density cap, and the model keeps enough free cells to honour the first-move exclusion zone. A density-based multiplier feeds the score, so board difficulty is reflected in the result.

## Scoring

Score is computed live and updates after each reveal, hint or recovery. Conceptually it combines revealed cells weighted by a difficulty multiplier, a time bonus that shrinks as the game goes on, and penalties for hints and bomb recoveries, clamped so it never goes negative. High scores keep the top ten per preset difficulty, ordered by score and then by faster time. Custom boards are excluded from the high-score table.

## Hints

A hint marks one still-hidden mine as a warning without revealing it. Hints come from a consumable balance that starts small and can be topped up with hint packs. Each hint carries a score penalty, and the balance is stored on the device.

## Bomb recovery

When a mine is revealed the game does not end immediately. The timer stops, mines stay hidden, and a recovery prompt appears. On Mobile and Android TV the player can continue through a reward flow (up to two recoveries per game): the exploded mine becomes a flag, the timer restarts and the score takes a penalty. The mobile app animates the board shattering and then reassembling on a revive. The Apple TV app contains the same recovery state transition in its logic layer, but its post-loss dialog offers restart or exit.

## How each platform plays

| Platform | How you play |
|---|---|
| **Android phone and tablet** | Portrait. Tap to reveal, long-press to flag, pinch to zoom and drag to pan on large boards. Haptic feedback is optional. System back opens the pause menu. |
| **iOS** | The same Flutter app as Android, with touch input and native purchase and sound bridges. |
| **Android TV** | Landscape, D-pad driven. Arrow keys move a highlighted cell cursor; a short press on select reveals, a hold flags; the back key pauses with an exit confirmation. |
| **Apple TV** | Landscape, Siri Remote driven. Swipes move the cursor, a click reveals, a hold flags, and the Menu button pauses. |

On every platform reveal is the primary action, flag is the secondary action (long-press or hold), and back or menu pauses.

## Supporting screens and systems

- **Onboarding (mobile):** terms acceptance, language selection and a guided tutorial, plus a six-page How to Play.
- **Settings:** sound, language, replay tutorial, policy links, restore purchases, and an in-app screen linking to other apps by the same developer. Haptics are a toggle on mobile only.
- **Premium:** on mobile a premium option unlocks Custom mode and removes ads, and the mobile app also offers a trial-style subscription path on Google Play. On Android TV, custom unlock and ad removal are separate purchases. On Apple TV a one-time purchase unlocks Custom mode (the app has no ad SDK). Hint packs are separate consumables on every platform.
- **Notifications and updates (mobile):** push topics and an in-app update prompt.
- **Startup:** a splash screen shown while staged initialisation, consent and remote configuration complete.

<p align="center">
  <img src="../assets/screenshots/mobile/mobile-premium-paywall.webp" width="260" alt="Premium dialog (heading and description)"/>
  <img src="../assets/screenshots/mobile/mobile-splash-screen.webp" width="260" alt="Splash screen"/>
</p>

## Localization

The mobile app ships 15 languages with a user-selectable language setting. The two TV apps are English only.

## What the product does not have

To keep the case study honest: there is no account system or cloud sync, no achievements or leaderboards, no chording (revealing the neighbours of a satisfied number), no restore of an in-progress game after the app is closed, and no URL deep links (notification taps open in-app screens). Statistics beyond high scores are collected as analytics events rather than shown in a statistics screen.

## TV imagery

The Android TV launcher banner and the layered Apple TV icon are artwork, not screenshots. No TV gameplay screenshots are published here.

<table>
  <tr>
    <td align="center"><img src="../assets/artwork/android-tv-launcher-banner.webp" width="380" alt="Android TV launcher banner artwork"/><br/><sub>Android TV launcher banner (artwork)</sub></td>
    <td align="center"><img src="../assets/artwork/apple-tv-icon-back-layer.webp" width="380" alt="Back layer of the layered Apple TV app icon"/><br/><sub>Apple TV icon, back layer (artwork)</sub></td>
  </tr>
</table>

See [MOBILE_VS_TV.md](MOBILE_VS_TV.md) for the platform comparison and [TECHNICAL_CASE_STUDY.md](TECHNICAL_CASE_STUDY.md) for how it is built.
