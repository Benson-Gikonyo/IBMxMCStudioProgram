# Ajo — Fintech · Nigeria — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (fintech regulatory-reporting playbook) and country parameters from the **[Country dossier: Nigeria](../africa-regulatory/nigeria.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What Ajo is (working frame)

Assuming **Ajo** is a Nigerian digital lending / savings-and-loan ("ajo" = rotating savings) fintech. The Sita fintech playbook assumes a licensed digital lender that submits monthly returns to the central bank and quarterly summaries to a markets regulator, from a loan-ledger pipeline (extract → master → transform → artifact).

**Reference pipeline** (from `fintech/pipelines/cbk_report_pipeline.js`): loan ledger → extract → dedupe → transformation rules → report artifact → audit log.

---

## 2. Country & sector fundamentals — Nigeria

| Item | Value | Verify with |
|---|---|---|
| Central bank | Central Bank of Nigeria (CBN) | CBN licensing desk |
| Financial regulator (lending/payments) | CBN payment & lending licenses; NDPC for data protection | NDPC registration portal |
| Data protection | Nigeria Data Protection Commission (NDPC) — register before consumer launch | NDPC |
| Blocs | AU; ECOWAS | — |
| Corporate income tax | 30% (25% for qualifying SMEs) | Federal Inland Revenue Service (FIRS) |
| VAT / digital-service tax | 7.5% | FIRS |
| Official / business language | English (also Hausa, Yoruba, Igbo, Pidgin) | — |
| Investment promotion | Nigerian Investment Promotion Commission (NIPC); pioneer status; NEPZA zones | NIPC / NEPZA |

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
  - channel: [mpesa, bank, ussd]

RegulatoryReport
  - report_id (PK)
  - regulator: [CBN, ...]            ← sub local regulator names
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
    → Transformation (apply CBN regulator schema rules)
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
| Master data / dedupe | Postgres + fuzzy-match (pg_trgm), Salsa/Okta, your own rules | IBM MDM | Dedupe customers across products/channels |
| Rules / compliance | Open-source rules engine (Drools), code, OpenPages alt | IBM OpenPages | Rule library per regulator |
| Analytics / scoring | Scikit-learn, XGBoost, FastAPI | IBM Watson Studio | Risk scoring, anomaly, next-best-action |
| Orchestration | Airflow, Prefect, n8n | IBM Watson Orchestrate | Alerts, dispatch, workflows |
| Reports / artifacts | ReportLab, PDFKit, templating | Watson Studio / Cognos | Generate + auto-submit |
| Sandbox / deploy | Docker + CI (GitHub Actions) | IBM TechZone / CP4D | Prototyping + deployment |

> Use mock-first development (`SITA_MOCK_MODE=1`) and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & licensing checklist

- [ ] Confirm whether Ajo is a **lender**, **payment/savings** platform, or **both** — each attracts a different CBN licence and capital threshold.
- [ ] Obtain **CBN licensing** for the applicable activity (digital lender / MMO / payment) before Go-Live.
- [ ] Enrol with **NDPC** as a data controller — register before processing consumer data at scale.
- [ ] Implement **AML/CFT** controls and register under any relevant reporting regime (KYC tiers, transaction monitoring, suspicious-transaction reporting).
- [ ] Put in place **consumer-credit rules** — transparent pricing, caps, fair-debt-collection practices (Nigeria enforces strongly on digital lenders).
- [ ] Confirm **fee / rate disclosure** obligations and any **loan-cap** rules in force at the CBN.
- [ ] Document a **complaints and redress** channel consumers can actually use.
- [ ] Plan the **regulatory-reporting cadence**: monthly credit returns + periodic summaries — automate, don't hand-roll in Excel (that is exactly the Sita fintech pain point).

## 7. Product, data & integration checklist

- [ ] Build the **loan ledger** model: account, customer, repayment, report tables (§3 schema).
- [ ] Map the **data flow**: extract → master (dedupe on customer ID / ID number) → regulator-style transform → artifact.
- [ ] Decide **mock-first** strategy for all third-party integrations; only promote with real credentials later.
- [ ] Wire **scheduled, repeatable ingestion** (ETL) — not one-off scripts.
- [ ] Use **dedupe / master record** to avoid duplicate or mis-matched lending records.
- [ ] Define KYC tiers (basic vs enhanced) and downstream risk scoring on the customer record.
- [ ] Set up **scheduled jobs** (monthly/quarterly) with an **audit log** of every generated report.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for anyone touching the reporting pipeline.
- [ ] Encrypt **PII at rest and in transit**; pseudonymise where the schema allows.
- [ ] Log every **report generation and submission** (regulator + period + status + file path) for audit.
- [ ] Keep records of **who/what/when** accessed customer data (data-protection readiness).

## 9. Tax & finance checklist

- [ ] Register for **CIT** (30%, or 25% SME rate if eligible) and **VAT** (7.5%).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties (commonly 5–15%, per treaty).
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **pioneer-status / NEPZA zone** incentives through NIPC before structuring.
- [ ] Confirm whether **digital-service tax** applies to any cross-border revenue stream.

## 10. Expansion notes (when scaling)

- Nigeria sits in **ECOWAS** — factor regional customs, data-flow and licensing harmonisation into any cross-border play.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when Ajo expands beyond Nigeria; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting. Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
