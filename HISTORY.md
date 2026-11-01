# History – Docs

Work log of the documentation. Newest entries first.
Detailed implementation history: `desktop/HISTORY.md`.

## 2026-11-01
- Transfer formats `.leasepack` and the new `.leasesigned` are specified in `shared/libs/leasepack`.
  Desktop imports mobile returns; the Android app opens packages, signs and returns them
  (details in `desktop/HISTORY.md`). Authenticity across devices is still open.
- Android app skeleton (`mobile-app/android`, application ID `kienzle.leasemanager.android`) created;
  package import, PDF display and signing are the next steps.
- Desktop `.leasepack` import/export is implemented, but verification is local to one data directory.
  Cross-installation trust and the shared package schema remain prerequisites for mobile transfer
  (details in `desktop/HISTORY.md`).
- Desktop full data backup and isolated restore are implemented; backups include the local package key
  and are not encrypted.

## 2026-10-31
- `concept/README.md`: added a section on the QR payload (`lease:v1:<contractId>:<contentHash>`,
  PDF metadata `/Keywords`, `LeaseContractId`, `LeaseContentHash`; implemented in `desktop/.../pdf/`).
- Concept (roadmap) assessed: step 1 (data model/persistence/status) and step 2
  (QR/ID + hash in the PDF) are implemented in the desktop app; step 6 (i18n) already existed.
- Step 3 (scan import) is now implemented in the desktop app (see `desktop/HISTORY.md`); the QR code is now
  at the top right instead of in the footer (`concept/README.md` adjusted).
- Scan import additionally checks the hash in the QR code against the original (warning for an older
  version); unsigned contracts can be deleted permanently, signed ones only archived (see `desktop/HISTORY.md`).
- Not implemented yet: the shared package schema in `shared/` and cross-installation package trust,
  5 (mobile), 7 (convenience transfer/backend relay).
