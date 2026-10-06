# Architecture

Diagram labels are conceptual and do not correspond to production identifiers.

## Platform and codebase map

Four platforms are built from three codebases that implement one game design.

<p align="center">
  <img src="../assets/diagrams/platform-overview.svg" alt="Platform overview: four platforms built from three codebases with one shared game design" width="900" />
</p>

## Flutter apps: layers

The mobile app and the Android TV app follow the same layering. The mobile app has additional services (consent, notifications, update, localization, ad mediation); the TV app has a smaller set plus focus telemetry.

```mermaid
flowchart TD
    subgraph Presentation
        SC["Screens: splash, home, difficulty, game, game over, how to play, settings"]
        DL["Dialogs, popups, blast and revive overlays"]
        MK["Shared motion tokens and pressable widgets"]
    end
    subgraph State
        GS["Game state notifier"]
        US["User profile: hint wallet, premium, custom unlock"]
        HS["High-score board"]
        ST["Settings and locale"]
    end
    subgraph Domain
        GL["Game logic: pure static functions"]
        MD["Cell, game state, difficulty config, high score: immutable models"]
    end
    subgraph Services
        AU["Audio synthesis + haptics"]
        BL["Billing facade"]
        AD["Ad services and gates"]
        AN["Analytics queue, sanitiser, crash reporting, performance traces"]
        OT["Consent, notifications, in-app update (mobile)"]
    end
    subgraph Data
        HV["Hive boxes"]
        SP["Shared preferences: settings, premium flag"]
    end
    subgraph Native
        KT["Kotlin: Play Billing, sound pool, splash"]
        SW["Swift (iOS): payment queue, sound, splash"]
    end
    SC --> GS
    DL --> GS
    GS --> GL
    GL --> MD
    GS --> AU
    GS --> AN
    SC --> US
    US --> BL
    BL <--> KT
    BL <--> SW
    SC --> AD
    HS --> HV
    ST --> SP
    US --> SP
    US --> HV
```

State is held in Riverpod providers: a game-state notifier owns the grid, status, timer and counters; separate providers own the user profile, high scores and settings. The notifier calls the pure logic layer and replaces its state with the result, so rebuilds are driven by immutable value changes.

## Apple TV app: layers

The native app has no third-party dependencies and no analytics, crash or ad SDKs.

```mermaid
flowchart TD
    APP["App entry: environment objects"] --> CV["Content view: splash, then screen switch"]
    NAV["Navigation manager: enum screen stack"] --> CV
    CV --> GVW["Game view"]
    GVW --> PT["UIKit press tracker<br/>select tap, select hold"]
    GVW --> MC["SwiftUI move, play-pause and exit commands"]
    GVW --> VM["Game view model: published state"]
    VM --> GL["Game logic: pure struct functions"]
    VM --> AUD["Audio engine: synthesised tones"]
    UVM["User view model: hints, custom unlock"] --> SK["StoreKit 2 service<br/>updates listener, entitlements, restore"]
    HVM["High-score and settings view models"] --> UD["UserDefaults"]
    UVM --> UD
```

The SwiftUI side uses `ObservableObject` view models and environment injection. The press tracker and the move commands are the only input path to the board.

## Input pipeline

<p align="center">
  <img src="../assets/diagrams/input-pipeline.svg" alt="Input pipeline: touch, D-pad and Siri Remote inputs become intents, then game logic, state and feedback" width="900" />
</p>

Each platform converts hardware input into the same few intents, each guarded by state: not while paused, not after the game is over, not while a dialog is open, and not a flag before the first reveal.

## One reveal, end to end

```mermaid
sequenceDiagram
    participant I as Input
    participant N as State notifier
    participant L as Game logic
    participant U as UI and feedback
    I->>N: reveal (row, col)
    alt first reveal
        N->>L: generate mines outside 3x3 zone
        L-->>N: grid with counts
        N->>N: start one-second timer
    end
    N->>L: reveal cell (iterative flood fill)
    L-->>N: new grid, newly revealed count, mine hit?
    N->>N: recompute score
    N->>L: win check
    N-->>U: new state: rebuild, sound, haptic
    opt mine hit
        N->>N: status lost, stop timer
        N-->>U: blast, then recovery prompt
    end
```

## Bomb recovery state

```mermaid
stateDiagram-v2
    [*] --> NotStarted
    NotStarted --> Playing: first reveal
    Playing --> Won: all safe cells revealed
    Playing --> Lost: mine revealed
    Lost --> Playing: recovery granted, at most twice
    Lost --> GameOver: declined or exhausted
    Playing --> Paused: back, menu or backgrounding
    Paused --> Playing: resume
    Won --> [*]
    GameOver --> [*]
```

On recovery the exploded mine is converted to a flag, so the hidden-plus-flagged count still equals the mine total and the win condition stays valid. The reward gate resolves once per request and the recovery is re-checked against the per-game limit. On Apple TV the logic transition exists but the post-loss dialog offers restart or exit.

## Startup (mobile)

```mermaid
flowchart LR
    S["Native splash"] --> F["Preferences and Firebase<br/>timeout-bounded"]
    F --> E["Native entitlement sync<br/>shorter timeout, cached fallback"]
    E --> P{"Premium?"}
    P -- yes --> H["Home"]
    P -- no --> C["Consent"]
    C --> R["Remote ad config<br/>fail-closed"]
    R --> A["Ad SDK init and app-open ad<br/>hard cap, paused during prompts"]
    A --> H
    H --> D["Deferred init: analytics, crash reporting"]
```

## Billing path

```mermaid
flowchart LR
    UI["Paywall or hint packs"] --> FA["Dart billing facade"]
    FA <-->|"method and event channels"| NM["Native manager<br/>Kotlin Play Billing or Swift StoreKit"]
    NM --> ST["Store"]
    NM -->|"purchase, restore, error events"| FA
    FA --> PR["Premium flag and hint wallet"]
    PR --> UI
```

The Apple TV app skips the bridge and uses StoreKit 2 from Swift directly. The Flutter apps and the Apple TV app handle purchases independently; entitlements are not shared between them.

## Shared versus platform-specific

| Concern | Shared across platforms | Platform-specific |
|---|---|---|
| Rules | Generation, flood fill, flags, win and loss, score, hints, recovery (identical Dart in two apps, hand-ported Swift) | None |
| Models | Cell, game state, high-score and profile models (byte-identical in the two Flutter apps) | Swift equivalents |
| Input | Intent vocabulary | Gesture detectors, key handling, UIKit press tracker |
| Layout | Visual language and colours | Portrait mobile layout, landscape TV layouts, cell sizing |
| Services | Concepts: audio synthesis, analytics sanitising | Ads vendor, consent, notifications, update, billing surface, telemetry |
| Persistence | Data shape of high scores | Hive and shared preferences, or `UserDefaults` |
