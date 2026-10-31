# Concept: Lease Contract Workflow

## Goal
Create and manage lease contracts in the desktop app and prove the signature either
**digitally via smartphone/tablet (transfer)** or **via printout + scan**.
Both paths end in the same link: *contract ↔ signed PDF*.

## Components
| Folder | Role |
|---|---|
| `desktop/` | JavaFX app: create contracts, generate PDFs, manage, import signed PDFs |
| `mobile-app/android`, `ios` | Receive, view and sign the PDF, send it back |
| `shared/` | Platform-neutral specification (JSON schema of the contract package, test data) |
| `backend/` | Optional, not needed for v1 (see below) |
| `docs/` | Documentation |

## Status model
`DRAFT → PDF_CREATED → AWAITING_SIGNATURE → SIGNED`
Additional fields: `signatureSource = MOBILE | SCAN`, `signedAt`, `signedPdfSha256`.

## Key idea: contract ID inside the PDF
Every generated PDF contains the `contractId` (UUID) and a hash as
- a QR code in the footer (readable on the printout as well),
- PDF metadata (`/Keywords`, custom property).

This allows both return paths to be matched automatically.

## Path A – Transfer (smartphone/tablet)
1. The desktop app exports a **contract package** (`.leasepack` = ZIP):
   `manifest.json` (contractId, version, sha256, parties, title) + `contract.pdf`.
2. Transfer (v1, no server): share the file (AirDrop/Nearby Share/USB/mail).
   Later: QR pairing + local Wi-Fi (HTTP, one-time token, TLS/pinning).
3. The mobile app shows the PDF; the signature is drawn by finger/pen (page + position),
   written into the PDF, and `signed.json` is added (timestamp, device, hash of the original).
4. Return transfer by the same means; the desktop app **imports** the file, matches it by
   `contractId` and verifies that the original hash matches the generated PDF.

## Path B – Printout + scan
1. The desktop app prints the PDF (with QR code). Status `AWAITING_SIGNATURE`.
2. Scan the signed document → import the PDF/image into the desktop app.
3. The QR code is detected → automatic matching; otherwise pick manually from the
   list of open contracts. The scan is stored unchanged and its hash is recorded.

## Overview (desktop)
Table with status, signature source, date, link to original/signed PDF,
filter "open/signed".

## Data model (proposal)
```
Contract   { id: UUID, ...contract data, status, createdAt, updatedAt }
Document   { id, contractId, kind: ORIGINAL|SIGNED_MOBILE|SIGNED_SCAN, path, sha256, createdAt }
Signature  { contractId, source: MOBILE|SCAN, signedAt, signerName?, deviceInfo? }
```
Storage v1: local data directory, contracts as JSON + PDFs on the file system
(or SQLite). The package format is versioned in `shared/` as a JSON schema
(`schemaVersion`) so that desktop/Android/iOS read it identically.

## Backend / Shared
- **shared/**: JSON schema + sample packages + test vectors (hash, QR payload). Do not share code
  (Java/Kotlin/Swift) – only specification and fixtures.
- **backend/**: skip for v1 (privacy: contract data stays local). Optionally later as a
  relay (end-to-end encrypted, temporary package storage) for transfer without a shared Wi-Fi.

## Legal note
A signature drawn by finger is not a qualified electronic signature.
Lease contracts longer than one year require written form (§ 585a BGB) – check whether the
handwritten signature on the printout (Path B) must remain the standard case.

## Analysis of the current desktop state
- [`LeaseContract`](../../desktop/src/main/java/kienzle/desktop/LeaseContract.java): record with ~38
  `String` fields (dates, amounts, areas as text), no status/document fields.
  → Introduce proper types (`LocalDate`, `BigDecimal`), add status + documents, use a UUID as ID.
- No persistence visible → introduce a `ContractRepository` (JSON/SQLite).
- [`MainLayoutController`](../../desktop/src/main/java/kienzle/desktop/MainLayoutController.java)
  (679 lines) mixes UI and logic → extract services (`PdfService`, `ContractService`,
  `PackageExporter/Importer`).
- [`AmtsgerichtRepository`](../../desktop/src/main/java/kienzle/desktop/AmtsgerichtRepository.java):
  clean and tested, no action needed.
- `SyncDialog.fxml` already exists → starting point for the transfer UI.
- Dependencies: PDFBox is present (QR via ZXing, signature image embedding possible with PDFBox).

## Localization (i18n)
The UI supports German and English, switchable at runtime via a language button.
- Texts live in resource bundles, not in code: `messages_de.properties`, `messages_en.properties`
  (loaded with `ResourceBundle`, UTF-8; FXML uses `%key` references).
- The selected language is stored in a config file (e.g. `config.properties`, key `language=de|en`)
  and loaded at startup; the button updates it and reloads the views.
- This includes validation/error messages (e.g. in `AmtsgerichtRepository`) – they use message keys.
- Generated contract PDFs stay German (legal text); only the app UI is switched.
  Make this configurable if English contract templates are needed later.
- Mobile apps use their platform mechanisms (Android `strings.xml`, iOS `Localizable.strings`),
  with keys kept in sync with desktop where useful.

## Roadmap
1. Data model + persistence + status in desktop
2. QR/ID + hash in the PDF
3. Scan import with matching (Path B, lowest risk)
4. `.leasepack` format + schema in `shared/`
5. Mobile: import, sign, export
6. Localization (DE/EN bundles, config, language button) – best done early
7. Convenience transfer (QR pairing/Wi-Fi), optional backend relay
