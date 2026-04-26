# Tax Filing Tool - Spec v1 (Final)

## Problem

German businesses file monthly UStVA + quarterly ZM with tax authorities via ELSTER. Currently:
- Manual data entry or export → manual ELSTER upload
- No unified interface, no audit trail, no analytics

**MVP solves:** Central UI to manage UStVA + ZM, draft storage, ELSTER submission, analytics.

---

## Scope (v1)

### Documents
- **USt 1 A** (Umsatzsteuervoranmeldung): 55 Meldezeilen + all 68 Kennziffern
- **USt 1 H** (Sondervorauszahlung): 13 Zeilen
- **ZM** (Zusammenfassende Meldung): Quarterly + monthly option

### Input Methods
1. **Manual form** (web UI with validation)
2. **CSV upload** (BMF 2026 format)
3. **REST API** (external systems)

### Storage & Submission
- Save drafts (unlimited versions)
- Submit to ELSTER via erica (REST wrapper)
- Download receipts + XML
- Audit trail (all versions + submissions)

### Dashboard (Post-Submission)
- **Single doc:** Highlights (6 KPIs) + metadata
- **Multi-doc:** Comparison + Abweichungsanalyse + Entwicklungsanalyse

### Exports
- **PDF:** Formatted form (USt 1 A / USt 1 H / ZM) — ready to print
- **CSV:** BMF 2026 format — for re-import or accounting software
- **XML:** ELSTER submission format

---

## Data Model

```
User
├── id (UUID)
├── email (unique)
├── password_hash
├── certificate (encrypted PEM/P12)
└── created_at

Document
├── id (UUID)
├── user_id (FK)
├── type (ENUM: USt1A, USt1H, ZM)
├── period (YYYY-MM or YYYY-Q#)
├── status (draft | submitted | error)
├── data (JSON: Kz → value pairs)
├── elster_receipt (JSON: response from erica)
├── created_at, updated_at, submitted_at
└── versions (array of prior states)

Certificate
├── id (UUID)
├── user_id (FK)
├── content (encrypted)
├── uploaded_at
└── expires_at (nullable)
```

---

## API Endpoints

### Auth
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/certificate (upload + encrypt)
```

### Documents
```
GET    /api/documents (filter: type, period, status)
GET    /api/documents/:id
POST   /api/documents (create draft)
PATCH  /api/documents/:id (update draft)
DELETE /api/documents/:id (delete draft)
POST   /api/documents/:id/submit (→ ELSTER)
GET    /api/documents/:id/download (receipt/XML)
GET    /api/documents/:id/export/pdf (formatted form)
GET    /api/documents/:id/export/csv (BMF 2026 format)
POST   /api/documents/import/csv
```

### Dashboard
```
GET /api/dashboard/highlights/:id
GET /api/dashboard/comparison (multi-doc analysis)
GET /api/dashboard/trends (12-month trend)
```

---

## USt 1 A Field Structure (55 Lines)

### Input Fields (Betrag/Text/Datum)
- KZ 10: Berichtigte Anmeldung (checkbox)
- KZ 22: Belege beigelegt (checkbox)
- KZ 70: Datum Wechsel Kleinunternehmer (datum)
- **KZ 81: Umsätze 19%** (betrag)
- **KZ 86: Umsätze 7%** (betrag)
- **KZ 87: Umsätze 0% PV** (betrag, NEW 2026)
- KZ 35/36: Andere Steuersätze (betrag)
- ... (25+ more input fields)

### Tax Calculation Fields (Berechnet)
- **KZ 19/20: SUMME Umsatzsteuer** = SUM(Z13-18, Z25-32)
- **KZ 66-67: Vorsteuer** = individual inputs
- **KZ 62: Einfuhrumsatzsteuer** (betrag)
- **KZ 65: Verbleibender Betrag** = KZ19/20 - (KZ66+61+62+67+63+59) + KZ64

### Final Calculation (Zahllast)
- **KZ 83: Vorauszahlung** = KZ65 - KZ69 - KZ39
- KZ 39: Unrichtig ausgewiesene Steuer
- KZ 69: Nachsteuer/Wechsel Besteuerungsform
- **KZ 29, 26, 500:** Checkboxes (Verrechnung, SEPA, Ergänzungen)

### Dashboard Highlights (for submitted doc)
```
1. Steuerpflichtig Inland (KZ 81)
2. Steuerfrei lief. EU (KZ 41)
3. Steuerfrei lief. Drittland (KZ 43/48)
4. Vorsteuer Inland (KZ 66)
5. Einfuhrumsatzsteuer (KZ 62)
6. ZAHLLAST (KZ 83) ← primary
```

---

## USt 1 H Field Structure (13 Lines)

- Steuernummer (text, required)
- Wirtschafts-ID (text)
- **Summe Vorauszahlungen 2025 + SV** (betrag)
- **KZ 38: Sondervorauszahlung 2026** = (Summe : 11) [berechnet]
- KZ 10: Berichtigte Anmeldung (checkbox)
- KZ 29: Verrechnung erwünscht (checkbox)
- KZ 26: SEPA-Lastschrift soll nicht verwendet (checkbox)
- KZ 500: Ergänzende Angaben (checkbox)

---

## ZM (Zusammenfassende Meldung)

**Scope:** Quarterly + monthly option (user choice per period)

**Fields:**
- Steuernummer (text, required)
- Partner country table (EU supplies)
  - Land (DE, AT, etc.)
  - Umsatz netto
  - MwSt-Satz

**Status:** Draft + submitted (like UStVA)

---

## Security

1. **Certificate Management:**
   - User uploads `.pfx` or `.pem`
   - Encrypted at rest (AES-256)
   - Decrypted only for ELSTER submission
   - Never logged

2. **Authentication:** JWT (1h expiry)

3. **Validation:**
   - Input sanitization
   - XML schema validation (before ELSTER)
   - Calculation verification (all formulas)

4. **Audit:**
   - All submissions logged
   - Version history per document
   - User action timestamps

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Invalid field | UI validation + API 400 + field highlight |
| Cert expired | Block submission, prompt upload |
| ELSTER API down | Queue + retry (exponential backoff) + notify user |
| ELSTER validation error | Show details, user corrects, resubmits |
| Network timeout | Preserve session, offer retry |

---

## Success Criteria (MVP v1 Complete)

- [ ] User registration + certificate upload (encrypted)
- [ ] Create/edit/delete USt 1 A draft
- [ ] Create/edit/delete USt 1 H draft
- [ ] Create/edit/delete ZM draft
- [ ] All 68 KZ calculations verified
- [ ] Submit USt 1 A to ELSTER (erica)
- [ ] Submit ZM to ELSTER
- [ ] Download receipts + XML
- [ ] Dashboard: single doc highlights (6 KPIs)
- [ ] Dashboard: multi-doc comparison + trends
- [ ] CSV import (BMF 2026 format)
- [ ] PDF export (formatted forms)
- [ ] CSV export (BMF 2026 format)
- [ ] XML export (ELSTER format)
- [ ] Audit log (all changes)
- [ ] Deployed on cloud (HTTPS, secure cert storage)

---

## Out of Scope (v2+)

- Sondervorauszahlung (Antrag)
- Multi-workspace / client management
- Webhook notifications
- Accounting software integrations
- Mobile app
- 2FA

---

## Tech Stack (Final)

**Backend:** Python 3.12 + FastAPI + PostgreSQL
**Frontend:** React 19 + TypeScript + TailwindCSS
**ELSTER:** erica REST wrapper (dockerized)
**Deployment:** Docker Compose + Cloud (AWS/Hetzner)
**Security:** TLS 1.3, AES-256, bcrypt hashing
