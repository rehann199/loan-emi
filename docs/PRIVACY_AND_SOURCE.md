# Privacy and Source

## What this repository is

A portfolio showcase for a production application. It describes the engineering of Loan EMI; it is not a source distribution.

## What is published

- Written descriptions of architecture, design decisions and trade-offs
- Original diagrams created for this showcase
- Real screenshots from the Android app
- The app icon, resized

## What is intentionally not published

- Production source code, in whole or in part
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration, API keys and tokens
- Ad network identifiers and placement IDs
- Billing product and subscription identifiers
- Security-rule contents and internal endpoints
- Version control history of the production repository
- Any user or financial data

## How the descriptions were produced

The production project was inspected read-only. Statements describe behaviour observed in code, manifests and project documents. Where behaviour could not be confirmed it is worded conservatively or left out. Diagram labels are conceptual and do not match production identifiers.

## What the app does with data

Based on the app's own code and in-app privacy text: loans, preferences and history are stored on the device. Sign-in is optional; when a user signs in, that data is backed up to a user-scoped cloud store. The app also uses analytics, crash reporting and an ad network.

## Images

Screenshots are real captures from the Android app, taken on a fresh install, so they contain no saved loans or personal entries. Android system status and gesture bars were removed. Captures with transitional frames or content that did not match this write-up were not used. Images were re-encoded with metadata removed.

## Publication checks

Before publishing, every file was checked for source code, credentials, signing material, personal data, email addresses, local file-system paths and embedded image metadata. None remain.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
