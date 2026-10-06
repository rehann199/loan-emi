# Architecture

A single Flutter app with a deliberately thin boundary: pure calculation code at the centre, on-device storage around it, and optional services (Firebase, Play billing, ads) layered on the edges. Component names here are conceptual.

## Layers

| Layer | Responsibility |
|---|---|
| Screens | Calculator, Result, Schedule, My Loans, Tools hub and each tool, Settings, onboarding, paywall |
| App state | One change-notifier model exposed through `provider`: language, currency, theme, income, saved loans, free-tier counters, current tab |
| Calculation layer | A set of small files of pure Dart with no UI, storage or network: EMI engine, prepayment, comparison, flat rate, refinance, stress test, APR, borrowing power, affordability, interest, savings, inflation, loan progress and totals |
| Persistence | `shared_preferences`; saved loans stored one JSON record each, with a fail-safe wrapper that falls back to defaults |
| Sync | Optional Firestore mirror for signed-in users |
| Services | Billing, ads, analytics, notifications, in-app update, connectivity |
| Native | Kotlin splash activity and main activity |

State is held by one `AppState` change notifier plus a purchase service, supplied through `MultiProvider` with Firebase access as a plain provider.

## System overview

<p align="center">
  <img src="../assets/diagrams/system-overview.svg" alt="System overview diagram" width="900" />
</p>

## Calculation flow

```mermaid
flowchart TD
    IN[Inputs: amount, rate, tenure, start date] --> V{Valid?}
    V -- no --> ERR[Field error while typing / ArgumentError at the engine]
    V -- yes --> R{Rate is zero?}
    R -- yes --> E0[EMI = principal / months]
    R -- no --> E1[EMI = closed-form annuity payment]
    E0 --> S[Build schedule: interest = balance x rate,<br/>principal = EMI - interest]
    E1 --> S
    S --> F[Final row absorbs rounding residue,<br/>balance set to zero]
    F --> T[Total payment = EMI x months<br/>Total interest = total - principal]
    T --> OUT[Result screen: EMI, totals, charts]
    S --> SCH[Schedule screen: CSV / PDF]
    S --> PRE[Prepayment, stress test,<br/>comparison reuse the engine]
```

## Saved loan lifecycle

```mermaid
flowchart LR
    RES[Result screen] -->|Save loan, free-tier check| SL[Saved loan<br/>inputs + paid months]
    SL --> ST[(On-device store)]
    ST --> SUM[Recompute through engine]
    SUM --> PR[Progress from paid count]
    SUM --> TOT[Totals per currency]
    PR --> DASH[My Loans dashboard]
    TOT --> DASH
    DASH -->|Mark paid| SL
    SL --> REM[Local payment-due reminder]
```

## Sync flow

```mermaid
sequenceDiagram
    participant U as User
    participant L as Local store
    participant S as Sync service
    participant F as Firestore (user scope)
    U->>L: Save / edit / delete loan
    L->>S: Record pending change
    S->>F: Write when signed in and online (timeout bounded)
    F-->>S: Live snapshot
    S->>L: Apply snapshot, keeping pending edits and deletes
    Note over S,F: First sign-in merges by loan id, local wins on collision
```

## Ads and entitlement

```mermaid
flowchart LR
    CFG[(Firestore ad config<br/>placement + version switches)] --> GATE{Ads allowed?}
    PREM[Premium entitlement<br/>persisted locally] --> GATE
    GATE -- yes --> TH[Per-zone request throttle]
    TH --> SDK[Mediation SDK]
    GATE -- no --> NONE[No ads]
```

Remote config fails open: if it cannot be read, every placement stays enabled. Premium removes ads everywhere through one central gate.

## Startup

`main` loads preferences before the first frame so routing is correct on a cold start, records the first-launch time, loads saved loans and loan types, re-syncs payment reminders (to recover any the OS evicted), and brings up Firebase and the ad SDK on a best-effort basis. Firebase and ad failures degrade the feature and never block launch. A native Kotlin splash activity mirrors the first Flutter splash frame and hands off to the main activity without an animation.

## Design choices

- **Pure core.** Maths lives outside widgets so it can be tested without a widget tree.
- **Derived over stored.** Saved loans keep inputs; everything displayed is recomputed.
- **Hand-drawn charts.** The pie and balance-over-time charts are custom painters, so there is no charting dependency.
- **Single source of limits.** Free-tier numbers are defined in one class.
