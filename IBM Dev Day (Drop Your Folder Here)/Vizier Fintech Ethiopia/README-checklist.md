# Vizier — Fintech · Ethiopia — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (fintech regulatory-reporting playbook) and country parameters from the **[Country dossier: Ethiopia](../africa-regulatory/ethiopia.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What Vizier is (working frame)

Assuming **Vizier** is an Ethiopian digital financial platform (lending, payments, or analytics — "vizier" = adviser). The Sita fintech playbook assumes a licensed player that submits periodic returns to the central bank from a transaction/loan-ledger pipeline (extract → master → transform → artifact).

**Reference pipeline** (from `fintech/pipelines/cbk_report_pipeline.js`): ledger → extract → dedupe → transformation rules → report artifact → audit log.

---

## 2. Country & sector fundamentals — Ethiopia

| Item | Value | Verify with |
|---|---|---|
| Central bank | National Bank of Ethiopia (NBE) | NBE licensing |
| Financial regulation | NBE licensing for payments/lending; fintech sandbox | NBE |
| Data protection / cyber | INSA (Information Network Security Administration — verify data mandate) | INSA |
| Blocs | AU; COMESA; IGAD | — |
| Corporate income tax | 30% | Ministry of Revenues |
| VAT / sales tax | 15% | Ministry of Revenues |
| Official / business language | Amharic (federal working language); English in business | — |
| Investment promotion | Ethiopian Investment Commission (EIC); industrial parks; export incentives | EIC |

> **Notice:** Ethiopia's fintech market is among the most restricted in Africa (foreign-ownership limits on financial services, strong NBE control). Legal counsel is essential before any structure is assumed.

---

## 3. Data model (build toward the fintech playbook)

```
Account / Ledger
  - account_id (PK)
  - customer_id (FK → Customer)
  - product_type: [personal, business, merchant]
  - balance / principal: DECIMAL
  - interest_rate: DECIMAL (where lending)
  - open_date: DATE
  - status: [active, defaulted, closed]

Customer
  - customer_id (PK)
  - id_number: VARCHAR (national/kebele ID)
  - kyc_tier: [basic, enhanced]
  - risk_score: INTEGER
  - created_at: TIMESTAMP

Transaction
  - transaction_id (PK)
  - account_id (FK → Account/Ledger)
  - amount: DECIMAL
  - payment_date: DATE
  - channel: [telebirr, bank, ussd]
  - direction: [debit, credit]

RegulatoryReport
  - report_id (PK)
  - regulator: [NBE, ...]           ← sub local regulator names
  - period: VARCHAR
  - generated_at: TIMESTAMP
  - status: [draft, submitted, accepted]
  - file_path: VARCHAR
```

---

## 4. Data flow & architecture

```
[Core / Payments DB]
    → Extract + validate
    → Master customer registry (dedupe on national ID / phone)
    → Transformation (apply NBE regulator schema rules)
    → [RegulatoryReport table]
    → Auto report generation + dispatch
    → Audit log entry
```

**Reference implementation layers** (from `fintech/`):
- `pipelines/*_report_pipeline.js` — pipeline config + transform
- `pipelines/transform.js` — schema mapping internal → regulator format
- `pipelines/report_scheduler.js` — cron trigger for monthly/quarterly runs
- `api/routes/reports.js` — `GET /reports`, `POST /reports/generate`
- `api/routes/compliance.js` — `POST /compliance/submit`
- `api/services/regulatoryEngine.js` — core transformation logic
- `api/services/mdmClient.js` — MDM API wrapper (identity resolution)
- `tests/*_pipeline.test.js`

---

## 5. Technology build reference (recommendations — pick the fit)

| Layer | Recommended (open, low-friction) | IBM option (sugg.) | Notes |
|---|---|---|---|
| Ingestion / ETL | Airbyte, dbt, Apache NiFi, cron + SQL | IBM DataStage | Repeatable, scheduled; not one-off scripts |
| Master data / dedupe | Postgres + fuzzy-match (pg_trgm), your own rules | IBM MDM | Dedupe customers across products/channels |
| Rules / compliance | Open-source rules engine (Drools), code | IBM OpenPages | Rule library per regulator |
| Analytics / scoring | Scikit-learn, XGBoost, FastAPI | IBM Watson Studio | Risk scoring, anomaly, next-best-action |
| Orchestration | Airflow, Prefect, n8n | IBM Watson Orchestrate | Alerts, dispatch, workflows |
| Reports / artifacts | ReportLab, PDFKit, templating | Watson Studio / Cognos | Generate + auto-submit |
| Sandbox / deploy | Docker + CI (GitHub Actions) | IBM TechZone / CP4D | Prototyping + deployment |
| Mobile-money | Ethio Telecom (telebirr) APIs | IBM API Connect (gateway) | telebirr is the dominant rail |

> Use mock-first development (`SITA_MOCK_MODE=1`) and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & licensing checklist

- [ ] Confirm whether Vizier is a **payments**, **lender**, or **both** — each attracts a different **NBE** licence and capital threshold.
- [ ] Engage **local counsel first**: Ethiopia restricts foreign ownership in financial services; confirm the correct corporate structure (local JV, license type, sandbox).
- [ ] Obtain **National Bank of Ethiopia** licensing / sandbox approval for the applicable activity.
- [ ] Verify **INSA** cyber-security and data-registration obligations (mandate to confirm).
- [ ] Implement **AML/CFT** controls — KYC tiers, transaction monitoring, suspicious-transaction reporting.
- [ ] Put in place **consumer-protection rules** — transparent pricing, caps, fair-debt-collection.
- [ ] Confirm **fee / interest-rate disclosure** obligations in force.
- [ ] Document a **complaints and redress** channel consumers can actually use.
- [ ] Plan the **regulatory-reporting cadence**: periodic returns — automate, don't hand-roll in Excel.

## 7. Product, data & integration checklist

- [ ] Build the **ledger** model: account, customer, transaction, report tables (§3 schema).
- [ ] Map the **data flow**: extract → master (dedupe on national ID / phone) → regulator-style transform → artifact.
- [ ] Decide **mock-first** strategy for all third-party integrations (incl. telebirr); promote with real credentials later.
- [ ] Wire **scheduled, repeatable ingestion** (ETL) — not one-off scripts.
- [ ] Use **dedupe / master record** to avoid duplicate or mis-matched customer records.
- [ ] Define KYC tiers and downstream risk scoring on the customer record.
- [ ] Set up **scheduled jobs** with an **audit log** of every generated report.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for anyone touching the reporting pipeline.
- [ ] Encrypt **PII at rest and in transit**; pseudonymise where the schema allows.
- [ ] Log every **report generation and submission** for audit.
- [ ] Keep records of **who/what/when** accessed customer data (INSA/DPA obligations).

## 9. Tax & finance checklist

- [ ] Register for **CIT** (30%) and **VAT** (15%).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties (commonly 5–15%, per treaty).
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **EIC / industrial-park / export** incentives before structuring.
- [ ] Confirm whether **digital-service / transaction levy** applies to payment flows.

## 10. Expansion notes (when scaling)

- Ethiopia sits in **COMESA** and **IGAD** — factor regional data-flow and licensing harmonisation into any cross-border play.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when Vizier expands; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting. Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
