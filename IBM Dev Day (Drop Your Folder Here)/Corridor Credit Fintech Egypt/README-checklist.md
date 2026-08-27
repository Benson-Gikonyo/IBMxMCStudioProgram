# Corridor Credit — Fintech · Egypt — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (fintech regulatory-reporting playbook) and country parameters from the **[Country dossier: Egypt](../africa-regulatory/egypt.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What Corridor Credit is (working frame)

Assuming **Corridor Credit** is an Egyptian digital lender / credit product ("corridor" = credit line / BNPL-style financing). The Sita fintech playbook assumes a licensed lender that submits periodic returns to the central bank and summary reports to a markets regulator, from a loan-ledger pipeline (extract → master → transform → artifact).

**Reference pipeline** (from `fintech/pipelines/cbk_report_pipeline.js`): loan ledger → extract → dedupe → transformation rules → report artifact → audit log.

---

## 2. Country & sector fundamentals — Egypt

| Item | Value | Verify with |
|---|---|---|
| Central bank | Central Bank of Egypt (CBE) | CBE licensing |
| Financial regulation | CBE licensing for payments/lending | CBE |
| Data protection | Data Protection Authority (Law 151/2020 — verify operational status) | DPA |
| Blocs | AU; COMESA; Arab League | — |
| Corporate income tax | 22.5% | Egyptian Tax Authority (ETA) |
| VAT / sales tax | 14% | ETA |
| Official / business language | Arabic; English & French in business | — |
| Investment promotion | GAFI; Suez Canal Economic Zone; free zones | GAFI |

---

## 3. Data model (build toward the fintech playbook)

```
LoanAccount
  - account_id (PK)
  - customer_id (FK → Customer)
  - product_type: [personal, business, asset]
  - principal_amount: DECIMAL
  - interest_rate: DECIMAL
  - disbursement_date: DATE
  - maturity_date: DATE
  - status: [active, defaulted, closed]

Customer
  - customer_id (PK)
  - id_number: VARCHAR
  - kyc_tier: [basic, enhanced]
  - risk_score: INTEGER
  - created_at: TIMESTAMP

RepaymentTransaction
  - transaction_id (PK)
  - account_id (FK → LoanAccount)
  - amount_paid: DECIMAL
  - payment_date: DATE
  - channel: [bank, mobile_wallet, card]

RegulatoryReport
  - report_id (PK)
  - regulator: [CBE, ...]            ← sub local regulator names
  - period: VARCHAR
  - generated_at: TIMESTAMP
  - status: [draft, submitted, accepted]
  - file_path: VARCHAR
```

---

## 4. Data flow & architecture

```
[Loan / Core Banking DB]
    → Extract + validate
    → Master customer/loan registry (dedupe on ID)
    → Transformation (apply CBE regulator schema rules)
    → [RegulatoryReport table]
    → Auto PDF/report generation + dispatch
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

> Use mock-first development (`SITA_MOCK_MODE=1`) and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & licensing checklist

- [ ] Confirm whether Corridor Credit is a **lender**, **payment/savings** platform, or **both** — each attracts a different CBE licence and capital threshold.
- [ ] Obtain **Central Bank of Egypt** licensing for the applicable activity before Go-Live.
- [ ] Enrol with the **Data Protection Authority (Law 151/2020)** — verify whether the authority is operational and register before processing consumer data at scale.
- [ ] Implement **AML/CFT** controls — KYC tiers, transaction monitoring, suspicious-transaction reporting.
- [ ] Put in place **consumer-credit rules** — transparent pricing, caps, fair-debt-collection practices.
- [ ] Confirm **fee / interest-rate disclosure** (pricing transparency is heavily scrutinised for Egyptian digital lenders).
- [ ] Document a **complaints and redress** channel consumers can actually use.
- [ ] Plan the **regulatory-reporting cadence**: periodic credit returns — automate, don't hand-roll in Excel.

## 7. Product, data & integration checklist

- [ ] Build the **loan ledger** model: account, customer, repayment, report tables (§3 schema).
- [ ] Map the **data flow**: extract → master (dedupe on customer ID / national ID) → regulator-style transform → artifact.
- [ ] Decide **mock-first** strategy for all third-party integrations; only promote with real credentials later.
- [ ] Wire **scheduled, repeatable ingestion** (ETL) — not one-off scripts.
- [ ] Use **dedupe / master record** to avoid duplicate or mis-matched lending records.
- [ ] Define KYC tiers and downstream risk scoring on the customer record.
- [ ] Set up **scheduled jobs** (monthly/quarterly) with an **audit log** of every generated report.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for anyone touching the reporting pipeline.
- [ ] Encrypt **PII at rest and in transit**; pseudonymise where the schema allows.
- [ ] Log every **report generation and submission** for audit.
- [ ] Keep records of **who/what/when** accessed customer data (data-protection readiness under Law 151/2020).

## 9. Tax & finance checklist

- [ ] Register for **CIT** (22.5%) and **VAT** (14%).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties (commonly 5–15%, per treaty).
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **GAFI / Suez Canal Economic Zone / free-zone** incentives before structuring.
- [ ] Confirm whether **digital-service tax** applies to any cross-border revenue stream.

## 10. Expansion notes (when scaling)

- Egypt sits in **COMESA** and the **Arab League** — factor regional customs, data-flow and licensing harmonisation into any cross-border play.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when Corridor Credit expands; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting. Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
