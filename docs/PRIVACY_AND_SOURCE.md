# Privacy and Source

## What this repository is

A portfolio showcase for a production application. It describes the engineering of Minesweeper across Android phone and tablet, iOS, Android TV and Apple TV; it is not a source distribution.

## What is published

- Written descriptions of architecture, design decisions and trade-offs
- Original diagrams created for this showcase
- Real screenshots from the Android build, cropped to the app content (system status bar and ad-placeholder strips removed), captured with sample data
- The app icon, the Android TV launcher banner artwork and the back layer of the layered Apple TV icon, resized

## What is intentionally not published

- Production source code, in whole or in part
- Tuning values, scoring constants and economy data
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration, API keys and tokens
- Ad network identifiers, placement IDs and account details
- Billing product identifiers and store-internal configuration
- Internal endpoints, database paths and security-rule contents
- Version control history of the production repositories
- User data of any kind

## Notes on figures

Line counts cover Dart under `lib/` (excluding generated localization) and all Swift source files. File counts cover test files under `test/`. Diagram labels are conceptual and are not production identifiers.

## Images

- **Screenshots** are real captures from the Android build. They show a fresh install with sample data; no account details, notifications or personal information appear. They were cropped to the app content and re-encoded to WebP with metadata removed.
- **Android TV and Apple TV** are represented by artwork and by diagrams, and each piece of artwork is labelled as artwork. No TV gameplay screenshots are published, and none are implied.
- **Diagrams** are original vector illustrations.

## Scope limits

The showcase does not make claims about store ratings, download counts, revenue or performance measurements, and it does not claim features that the code does not implement (game-state restore, accounts, cloud sync, achievements, chording, URL deep links). Test suites are described by what exists, not by whether they pass.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
