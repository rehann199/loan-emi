# Product Overview

Loan EMI is published on Google Play as "Loan & EMI Calculator". Its store description reads: "Calculate loan EMI, view schedule & save your loans. Simple, fast, offline."

## Who it is for

People comparing or tracking loans: a car or home loan, a personal loan, an offer from a second lender. The app answers "what will I pay each month, what will the loan cost in total, and can I afford it?"

## Navigation

A bottom navigation bar with four tabs: **Calculator**, **My Loans**, **Tools** and **Settings**.

First run goes through a language picker, a mandatory income and healthy-EMI-threshold step, and a short onboarding, before the calculator.

## Calculator and result

Inputs: loan amount, annual interest rate, tenure (toggle between years and months), a loan type, a start date and a currency chip. The amount field spells the number out in words under the input for English and Turkish, the two languages with a unit-tested converter, and falls back to a localised compact number (for example lakh and crore for Hindi and Urdu) for the rest.

The result screen shows the monthly EMI, total interest, total repayment, a principal-versus-interest chart and a balance-over-time chart. From there a user can open the full schedule, save the loan, compare it, or share a text summary.

## Schedule

One row per month with calendar month labels. It can be exported as a PDF, or as a CSV that holds plain numbers and sortable "YYYY-MM" months so a spreadsheet can calculate on it.

## My Loans

Saved loans with a name, loan type, favourite flag and progress. Each card shows EMI, remaining balance and a progress bar based on installments marked paid. A summary block shows combined monthly EMI of active loans, outstanding balance and remaining interest, grouped by currency. An empty state points back to the calculator. A "payment due" local reminder can be scheduled per loan.

## Tools

| Tool | What it does |
|---|---|
| Prepayment | Recurring extra payment and/or one-time lump sum; shows months and interest saved |
| Affordability | Combined EMI against monthly income and a threshold (default 40%) |
| Compare loans | Two to four offers, optional flat-rate and upfront costs |
| Refinance | Existing loan against a refinance offer, with break-even months on fees |
| Rate stress test | EMI at the current rate plus several rate increases |
| Borrowing power | Loan size supported by an affordable payment, or debt-to-income ratio |
| Interest | Simple and compound interest |
| APR | Effective borrowing cost once upfront fees are included |
| Savings and SIP | Future value of a lump sum plus monthly contributions |
| Inflation | Future cost and purchasing power |

Recent calculations (the last 10) are kept automatically so a user can return to one.

## Free and premium

Free users see ads and have two limits: three saved loans and five calculations per local day. A subscription (monthly or yearly, each with a 7-day free trial offer) removes ads and both limits. Schedule export is not behind the paywall in the code that was reviewed.

## Settings

Language, currency, theme (light, dark, system), sound effects and haptics, loan-type management, income and affordability, optional Google or Apple sign-in for backup, app tour replay, rate us, privacy, other apps and reset.

## Languages

English, Spanish, French, German, Portuguese, Italian, Arabic, Turkish, Russian, Hindi, Urdu, Chinese (Simplified), Japanese, Korean and Vietnamese. Arabic and Urdu are right-to-left; several widget tests build screens under a right-to-left text direction.
