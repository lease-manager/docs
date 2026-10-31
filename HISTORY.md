# History – Docs

Arbeitsprotokoll der Dokumentation. Neueste Einträge zuerst.
Detailverlauf der Umsetzung: `desktop/HISTORY.md`.

## 2026-10-31
- `concept/README.md`: Abschnitt zum QR-Payload ergänzt (`lease:v1:<contractId>:<contentHash>`,
  PDF-Metadaten `/Keywords`, `LeaseContractId`, `LeaseContentHash`; Umsetzung in `desktop/.../pdf/`).
- Konzept (Roadmap) bewertet: Schritt 1 (Datenmodell/Persistenz/Status) und Schritt 2
  (QR/ID + Hash im PDF) sind im Desktop umgesetzt; Schritt 6 (i18n) war bereits vorhanden.
- Schritt 3 (Scan-Import) ist inzwischen im Desktop umgesetzt (siehe `desktop/HISTORY.md`); QR-Code sitzt
  jetzt oben rechts statt im Footer (`concept/README.md` angepasst).
- Noch nicht umgesetzt: Schritt 4 (`.leasepack` + Schema in `shared/`),
  5 (Mobile), 7 (Komfort-Transfer/Backend-Relay).
