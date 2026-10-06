# Technical Case Study

Loan EMI is a Flutter app published on Google Play. This document explains how its calculation layer works and how the rest of the app is organised. Statements describe behaviour observed in the production code, which was inspected read-only. No metrics are claimed beyond counts taken from source.

## 1. Goals

- Correct loan arithmetic, including the cases a textbook formula forgets (zero rate, final-row rounding).
- Fully usable offline, with no account.
- Global: 15 languages, 47 currencies, right-to-left layouts.
- A tracker, not a one-shot calculator: saved loans with payment progress.

## 2. The EMI maths

The calculator uses the standard reducing-balance model. With principal P, annual rate R percent and n monthly installments:

- Monthly rate: r = R / 12 / 100.
- Level payment that clears the loan in n months: EMI = P * r * (1 + r)^n / ((1 + r)^n - 1).
- Each month, interest is the opening balance times r; the rest of the EMI repays principal; the balance falls by that principal.
- Total repayment is EMI times n, and total interest is total repayment minus P.

Why it works: the EMI is the payment for which the present value of n equal monthly payments, discounted at r, equals the amount borrowed. The closed form is that annuity relationship solved for the payment.

Edge cases handled in the code:

- **Zero rate.** The closed form divides by zero, so the code uses P / n directly.
- **Final row.** Rounding accumulates across the schedule and leaves a sub-cent remainder. The last row absorbs it into its principal portion and sets the balance to exactly zero. Totals use EMI times n, so the schedule sum and the headline total can differ by that sub-cent amount.
- **Validation.** Principal must be finite and positive, the rate finite and not negative, and tenure at least one month; otherwise the function throws instead of returning a misleading number.

### Verification

For 100,000 at 12% for 12 months the formula implies: monthly rate 1%, EMI 8,884.88 (8,884.878868 unrounded), total repayment 106,618.55 and total interest 6,618.55.

This was reproduced independently in three ways outside the app: the closed form, the present value of the payment stream (100,000 divided by the sum of 1/(1.01)^t for t = 1 to 12), and a bisection search for the payment that drives a month-by-month balance to zero. All three agree to about 1e-9, and a simulated schedule ends at a balance of effectively zero with total interest matching. A zero-rate case gives 100,000 / 12 = 8,333.33, matching the zero-interest branch.

## 3. Other calculators

Each is a pure function that reuses or mirrors the same discipline.

- **Prepayment.** The regular EMI is held fixed. Recurring extra and an optional lump sum (paid after a chosen installment, or before the schedule starts) go straight to principal. Amounts larger than the balance are capped, a lump sum for a month the loan never reaches is reported as not applied, and the loop is bounded by the original tenure so it cannot run forever.
- **Comparison.** Computes EMI, interest and total payment per offer (reducing balance or flat rate) and ranks lowest EMI, lowest interest, lowest effective cost (payments plus upfront costs) and shortest tenure separately. There is deliberately no single winner.
- **Flat rate.** Interest on the full original principal for the whole term, a different model from reducing balance and labelled separately.
- **Refinance.** Monthly saving, net saving after fees, and break-even months only when the monthly saving is positive.
- **Rate stress test.** Re-runs the EMI at the current rate plus each increase.
- **APR.** Solves for the monthly rate at which the amount actually received (principal minus upfront fees) equals the present value of the payments, using bisection; reports a nominal and a compounded effective figure.
- **Borrowing power.** The EMI formula inverted: the principal a given payment supports.
- **Affordability.** Total EMI as a percentage of income, against a user-adjustable threshold with comfortable, manageable and high-risk bands.
- **Interest, savings and SIP, inflation.** Closed-form simple and compound interest, an annuity projection (SIP is the same formula with payments at period start), and future cost and purchasing power under inflation.

## 4. Numeric precision and input rules

- Calculations use double-precision floats; there is no rounding inside the engine, only at display time.
- Amount fields accept whole numbers only (up to 8 digits). The rate field allows up to 100 with two decimals, and tenure up to 100 years or 1,200 months. Limits are enforced while typing.
- The grouping input formatter drops every non-digit and re-groups, so a locale's separator typed or inserted by the formatter is never mistaken for a decimal point.
- Display formatting uses the currency's home-region conventions (for example `1,00,000` style grouping for INR and `100.000,00` style for euro-zone locales) with two decimals. Currency is a separate setting from language.

## 5. Saved loans and progress

A saved loan stores inputs only: name, principal, rate, months, type, start date, currency, the set of paid calendar months, and a favourite flag. Everything else is recomputed. The count of paid installments is the single source of truth for progress, remaining balance, remaining interest and whether the "mark paid" action is still offered. Payments fill the earliest unpaid installment, recording is idempotent, and only months belonging to the schedule count, so a changed device clock cannot produce duplicates or an out-of-range tally. A future start date reads as not started; the Nth recorded payment reads as complete.

Dashboard totals aggregate per currency and never across currencies. Completed loans drop out of the monthly EMI total.

## 6. Storage and sync

Preferences and saved loans live in `shared_preferences`, each loan as its own JSON string so a corrupt entry can be dropped without losing the list. Writes to the loan list return success or failure so the UI can tell the user a save did not land.

When signed in (Google or Apple through Firebase Auth), a sync service mirrors loans, custom loan types, recent calculations and preferences to Firestore through live listeners. The first sync merges cloud and local loans by id (a union, with the local copy winning an id collision); after that, snapshots are applied with pending local edits and deletions protected, so a stale snapshot does not resurrect deleted data. The security rules limit user data to the owning authenticated user.

## 7. Export

CSV is built from plain numbers with sortable month keys. PDF is built with a header of totals and a table row per month. Both are written to a temporary file and handed to the system share sheet. The result screen also shares a three-line text summary.

## 8. Monetisation

Free users see ads from a mediation SDK, through a vendored plugin fork that adds a preload cache. Placements (language native ad, onboarding and home banners, interstitials on calculate and on two tab switches, an exit-confirm interstitial, and a startup app-open ad) can be switched off through Firestore documents, per placement or per app version and platform, and fail open. A request throttle puts a floor under repeated requests for the same zone. Premium is a subscription with monthly and yearly plans and a free-trial offer; the entitlement is persisted and not revoked by a transient store error. Free-tier limits (three saved loans, five calculations per day) are defined in one place.

## 9. Localisation

ARB files for 15 languages produce `AppLocalizations`; user-facing strings come from them. Noto fonts for Arabic, Urdu and Devanagari are bundled. Arabic and Urdu lay out right to left.

## 10. Testing

61 test files with 450 `test` and `testWidgets` declarations (261 `test`, 189 `testWidgets`; counted from source, not an executed-run result). Coverage spans the engine and every calculator, saved loans, progress, payment cycle, totals, free-tier counters, currency and language, and widget tests for most screens, plus one golden-image test.

## 11. Release

Android release builds use R8 minification and resource shrinking, with keep rules for the ad SDK. Signing credentials are read from a git-ignored properties file; when it is missing the build falls back to debug keys, which Play would reject. Crashlytics and Performance plugins are applied.

## 12. Known limits

- A schedule is not editable per row.
- Loan sync merges by loan id and, on an id collision, the local copy wins; there are no per-field timestamps.
- The project also contains an iOS target, but only the Google Play listing was verified for this write-up.
