# History – Docs

Work log of the documentation. Newest entries first.
Detailed implementation history: `desktop/HISTORY.md`.

## 2026-10-31
- `concept/README.md`: added a section on the QR payload (`lease:v1:<contractId>:<contentHash>`,
  PDF metadata `/Keywords`, `LeaseContractId`, `LeaseContentHash`; implemented in `desktop/.../pdf/`).
- Concept (roadmap) assessed: step 1 (data model/persistence/status) and step 2
  (QR/ID + hash in the PDF) are implemented in the desktop app; step 6 (i18n) already existed.
- Step 3 (scan import) is now implemented in the desktop app (see `desktop/HISTORY.md`); the QR code is now
  at the top right instead of in the footer (`concept/README.md` adjusted).
- Scan import additionally checks the hash in the QR code against the original (warning for an older
  version); unsigned contracts can be deleted permanently, signed ones only archived (see `desktop/HISTORY.md`).
- Not implemented yet: step 4 (`.leasepack` + schema in `shared/`),
  5 (mobile), 7 (convenience transfer/backend relay).
