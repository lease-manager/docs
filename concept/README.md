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
- a QR code at the top right of every page (readable on the printout as well),
- PDF metadata (`/Keywords`, custom property).

This allows both return paths to be matched automatically.

QR payload (v1): `lease:v1:<contractId>:<contentHash>` where `contentHash` is the SHA-256 of the PDF
*before* stamping. PDF metadata: `/Keywords` = payload, custom properties `LeaseContractId`,
`LeaseContentHash`. The SHA-256 of the final file is stored in `Document.sha256`.
Implemented in `desktop/.../pdf/` (`PdfStamper`, `QrPayload`, `QrCodes`).

## Path A – Transfer (smartphone/tablet)
1. The desktop app exports a **contract package** (`.leasepack` = ZIP):
   `manifest.json` (contractId, version, sha256, parties, title) + `contract.pdf`.
2. Transfer (v1, no server): share the file (AirDrop/Nearby Share/USB/mail).
   Later: QR pairing + local Wi-Fi (HTTP, one-time token, TLS/pinning).
3. The mobile app shows the PDF; the signature is drawn by finger/pen (page + position),
   written into the PDF, and `signed.json` is added (timestamp, device, hash of the original).
4. Return transfer by the same means; the desktop app **imports** the file, matches it by
   `contractId` and verifies that the original hash matches the generated PDF.

**Current limitation:** desktop `.leasepack` export/import is implemented, but package signatures
are verified with a key local to one data directory. Packages therefore cannot yet be exchanged
between installations or used by the mobile apps. Cross-device transfer needs a shared package
schema and an explicit trust/key-distribution model.

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
(or SQLite), with `.leasebackup` full backups and restore into a separate data directory. Backups
include the local package-signing key and are not encrypted. The `.leasepack` package format still
needs a platform-neutral JSON schema in `shared/` before desktop/Android/iOS can read it identically.

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
- [`LeaseContract`](../../desktop/src/main/java/kienzle/desktop/LeaseContract.java) uses typed date and
  amount fields; persisted workflow state, documents and signature metadata live in `ContractEntry`.
- Contracts and documents are stored locally through `ContractRepository` and `JsonContractRepository`.
- The desktop app implements PDF generation with QR/hash metadata, scan import, archive/restore and
  `.leasepack` export/import.
- Full local backups include active/archived records, documents, configuration and the package key;
  restoration uses a separate data directory and verifies file hashes.
- Language changes preserve unsaved form input and the current list selection.
- The package schema is not yet shared with Android/iOS, and current local package signatures prevent
  exchange between installations.
- Real-world print/scan testing remains open (see `desktop/HISTORY.md`).

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

## Offline signing, pairing and Wi-Fi sync (design)

Workflow: the contract goes to the phone, is signed there without any connection, stays stored in the app
and is transferred to the desktop later (file, or Wi-Fi sync when both are on the same network).

1. **Pairing (once):** the desktop shows a QR code with its public key (Ed25519) and a one-time token; the
   phone scans it and shows its own public key as QR code, which the desktop scans. Both sides store the peer key.
2. **Packages:** the desktop signs `.leasepack` with its private key; the phone accepts only packages from the paired desktop.
3. **Returns:** the phone signs `.leasesigned` (manifest incl. hashes) with its private key; the desktop accepts
   only returns from the paired phone, in addition to the existing hash and status checks.
4. **Wi-Fi sync:** the desktop opens a short-lived local HTTP endpoint on request (address and port in a QR code,
   no cloud); the phone uploads stored `.leasesigned` files, authenticated by the pairing keys. File transfer stays possible.
5. **Migration:** the local HMAC (`leasepack:v1`) remains readable for old packages; new packages use a new schema version.

Order of implementation: key pairs and pairing (desktop, then Android), signature in the formats, Wi-Fi sync.

## Roadmap
1. Data model + persistence + status in desktop
2. QR/ID + hash in the PDF
3. Scan import with matching (Path B, lowest risk)
4. Shared `.leasepack` schema and cross-installation trust/key-distribution model
5. Mobile: import, sign, export
6. Localization (DE/EN bundles, config, language button) – best done early
7. Convenience transfer (QR pairing/Wi-Fi), optional backend relay
