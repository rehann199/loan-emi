# Engineering Challenges

Six problems that stand out in the production code. Outcomes describe observable behaviour in the code; no metrics are claimed.

---

## 1. Money maths that survives edge cases

**Problem**
The standard EMI formula divides by zero at a 0% rate, and accumulated rounding leaves a stray sub-cent balance on the last schedule row.

**Context**
Zero-interest offers are real. A visible leftover balance on the final month makes the schedule look wrong even though every figure is nearly right.

**Approach**
- A zero rate takes a separate branch (principal divided by months).
- Input is validated at the function boundary and invalid values throw.
- The final row absorbs the residue into its principal portion and sets the balance to zero.
- Every other calculator reuses this function instead of re-deriving the formula.
- Maths was cross-checked against an independent computation (closed form, present-value sum and a search for the payment that clears the balance).

**Trade-offs**
Totals are computed as EMI times months, so the headline total and the schedule sum can differ by a sub-cent amount. Amounts use floating point; rounding happens only when formatting.

**Outcome**
No NaN or infinite output from valid input, a final balance of exactly zero, and a single engine shared by every tool.

---

## 2. Progress that cannot be corrupted by the clock

**Problem**
A tracker that derives progress from the calendar breaks when the device clock moves, and a "mark paid" action can double-count or become impossible.

**Context**
The paid state has to survive restarts, sync across devices and stay consistent between the progress bar, the balance, the payments tally and the pay button.

**Approach**
- Store the set of paid calendar months, never a computed count.
- Count only months that belong to the loan's schedule; recording is idempotent and always fills the earliest unpaid installment.
- Derive progress, remaining balance, remaining interest and the pay action from that one count. The clock decides only whether the next installment has arrived yet.
- A loan is complete on the Nth recorded payment, not when the term runs out.

**Trade-offs**
Progress follows what the user records, not what time implies. A user who forgets to mark payments sees an older balance.

**Outcome**
A wrong clock cannot create a duplicate, a negative or an out-of-range count, and cannot make remaining installments unpayable.

---

## 3. Currency and number input across locales

**Problem**
Grouping separators differ by locale ("50,000" vs "50.000"), so re-reading a formatted field can turn a thousands separator into a decimal point and collapse the value.

**Context**
Currency is independent of language, so a user can be reading German with a rupee amount.

**Approach**
- Amount fields are whole numbers: the formatter drops every non-digit and re-groups, so no separator is ever treated as a decimal.
- Digit and decimal limits are enforced during typing rather than reported afterwards.
- Display formatting follows the currency's home-region conventions, and the written-out amount is produced only for English and Turkish, the two languages with a unit-tested converter, with a localised compact number for the rest.

**Trade-offs**
No fractional loan amounts can be entered. Written-out words exist for two languages only, by design.

**Outcome**
Typing and editing mid-string behave predictably in any locale, and no unverified spelling of a number reaches a user.

---

## 4. Local-first data with a cloud copy

**Problem**
The app must work offline for everyone, yet back up and sync for signed-in users without bringing back deleted data.

**Context**
Edits and deletions can happen offline; a late cloud snapshot may predate them.

**Approach**
- The device is the immediate write target; writes report whether they landed.
- Pending upserts and deletions are recorded locally and flushed when connectivity returns.
- The first sync merges cloud and local loans by id; later snapshots are applied with pending edits and deletes protected.
- Cloud writes carry a timeout, and a corrupt record is skipped without losing the rest.
- Reset App backs up everything first and aborts the local wipe unless the complete cloud copy is confirmed.

**Trade-offs**
On an id collision the local copy wins, with no per-field timestamps, which is simple but not a field-level merge.

**Outcome**
Guests and signed-in users run the same local code, and destructive actions do not run on an unconfirmed backup.

---

## 5. Ads that can be switched off, and an entitlement that sticks

**Problem**
An ad layer needs a kill switch that does not require a release, and a paying user must not lose their entitlement to a flaky store connection.

**Context**
The mediation plugin binds banner creation to a platform view, so ads cannot be preloaded from Dart, and repeated requests for the same zone are answered as no-fill.

**Approach**
- Placements read per-placement and per-version-and-platform switches from Firestore and fail open on any error.
- A vendored fork of the plugin adds a preload cache.
- A per-zone throttle sits in front of every request path.
- The premium entitlement is persisted locally, read before the first frame and updated together with one central ad gate; a transient store error never revokes it.

**Trade-offs**
Failing open favours revenue over caution if the config is unreadable. The vendored fork must be maintained as third-party code.

**Outcome**
Placements can be turned off remotely, and a premium user sees no ads even offline.

---

## 6. A native splash that hands off cleanly

**Problem**
A cold start shows a native window before Flutter draws, and a mismatch between the two reads as a flash.

**Approach**
A Kotlin splash activity statically mirrors the resting frame of the Flutter splash (background, centred icon, title, a static ring), forwards any incoming intent, and starts the main activity on the next frame with no transition animation. Android 12 and later splash overrides are made transparent so only this layout shows.

**Trade-offs**
The two splashes must be kept visually in step by hand.

**Outcome**
The hand-off to Flutter's first frame is designed to be invisible.
