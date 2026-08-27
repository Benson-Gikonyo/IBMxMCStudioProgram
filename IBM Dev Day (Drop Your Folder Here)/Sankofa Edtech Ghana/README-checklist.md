# Sankofa — Edtech · Ghana — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (edtech playbook) and country parameters from the **[Country dossier: Ghana](../africa-regulatory/ghana.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What Sankofa is (working frame)

Assuming **Sankofa** is a Ghanaian edtech platform ("sankofa" = "go back and fetch it", a Ghanaian Akan concept). The Sita edtech playbook assumes blended learning (online modules + tutoring) where learner progress is tracked and reported to **funders/donors**, with instructor alerts for at-risk learners.

**Reference pipeline** (from `edtech/`): LMS activity → ingest → progress model → at-risk classifier → instructor alerts; donor-report transformer per donor schema → automated dispatch.

---

## 2. Country & sector fundamentals — Ghana

| Item | Value | Verify with |
|---|---|---|
| Ministry / regulator (education) | Ministry of Education; National Teaching Council / curriculum authorities | MoE Ghana |
| Data protection | Data Protection Commission (DPC) — register before processing consumer data | DPC |
| Revenue / tax | Ghana Revenue Authority (GRA) — CIT 25%, VAT 15% | GRA |
| Blocs | AU; ECOWAS | — |
| Official / business language | English (also Twi, Fante, Ewe, Ga, Dagbani) | — |
| Investment promotion | Ghana Investment Promotion Centre (GIPC); Ghana Free Zones Authority; one-district-one-factory | GIPC |

> **Notice:** edtech handles **minors' data**, which elevates data-protection and content-compliance obligations everywhere, Ghana included.

---

## 3. Data model (build toward the edtech playbook)

```
Learner
  - learner_id (PK)
  - name: VARCHAR
  - county / region: VARCHAR
  - school_name: VARCHAR
  - cohort_id (FK → Cohort)
  - enrollment_date: DATE
  - learning_mode: [online, blended, in_person]

Module
  - module_id (PK)
  - subject: VARCHAR
  - grade_level: INTEGER
  - total_lessons: INTEGER
  - pass_threshold: DECIMAL

Progress
  - progress_id (PK)
  - learner_id (FK → Learner)
  - module_id (FK → Module)
  - lessons_completed: INTEGER
  - last_active: DATETIME
  - score: DECIMAL
  - status: [on_track, at_risk, completed, dropped]

DonorReport
  - report_id (PK)
  - donor_id (FK → Donor)
  - period: VARCHAR
  - schema_version: VARCHAR
  - generated_at: TIMESTAMP
  - payload: JSONB
```

---

## 4. Data flow & architecture

```
[LMS Activity Logs]
    → Ingest (daily extract + normalise)
    → [Progress table] (dedupe learner across touchpoints)
    → At-risk classification model
    → [Instructor alert: score < threshold or 7-day inactivity]
    → [Donor Report engine: transform Progress → donor-specific schema]
    → Automated dispatch per donor schedule
```

**Reference implementation layers** (from `edtech/`):
- `pipelines/lms_ingest.js` — LMS → Progress ingestion
- `pipelines/at_risk_classifier.py` — ML scoring job (Watson or OSS model)
- `pipelines/donor_report_transformer.js` — Progress → donor schema (per donor config)
- `api/routes/learners.js` — `GET /learners`, `GET /learners/:id/progress`
- `api/routes/alerts.js` — `GET /alerts/at-risk`
- `api/routes/reports.js` — `POST /reports/generate/:donor_id`
- `api/services/watsonScorer.js` — model API client
- `api/services/donorSchemaMap.js` — donor schema registry
- `config/donor_schemas/*.json` — per-donor schema
- `tests/at_risk_classifier.test.py`, `tests/donor_transformer.test.js`

---

## 5. Technology build reference (recommendations — pick the fit)

| Layer | Recommended (open, low-friction) | IBM option (sugg.) | Notes |
|---|---|---|---|
| Ingestion | LMS API/webhooks; Airbyte/dbt | IBM DataStage | Daily extract + normalise |
| Analytics / risk | Scikit-learn, XGBoost, FastAPI | IBM Watson Studio | At-risk model on completion velocity |
| Mastering | Postgres dedup; Segleton | IBM MDM | Dedupe learners across touchpoints |
| Reporting | Templating + scheduler | Cognos / custom | Donor schema transformer |
| Alerts / comms | SendGrid, Twilio, in-app | Watson Orchestrate / Assistant | Instructor alerts |
| Sandbox / deploy | Docker + CI (GitHub Actions) | IBM TechZone / CP4D | Prototyping + deployment |

> Use mock-first development and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & compliance checklist (Ghana)

- [ ] Liaise with the **Ministry of Education** and curriculum authority for **content/curriculum accreditation** before rollout in schools.
- [ ] Register with the **Data Protection Commission (DPC)** as a data controller.
- [ ] Implement **enhanced data protection for minors' data** — parental consent, purpose limitation, minimal collection.
- [ ] Secure **intellectual-property and content licensing** for all learning materials.
- [ ] Publish clear **terms, privacy notice, and consent** in English and major Ghanaian languages.
- [ ] Follow **GIPA / school data-sharing** rules where integrating with government-run schools.
- [ ] Prepare **donor/funder impact reporting** per agreed schema (this is the crux of the edtech play) — don't hand-roll in Excel.

## 7. Product, data & integration checklist

- [ ] Build the **learner / module / progress / report** model (§3 schema).
- [ ] Map the **data flow**: LMS ingest → progress → at-risk → alerts; donor transform → dispatch.
- [ ] Decide **mock-first** for integrations with LMS and donor systems; promote with real credentials later.
- [ ] Wire **scheduled LMS ingestion** (daily) and **at-risk scoring** with a threshold (e.g. score < threshold or 7-day inactivity).
- [ ] Maintain a **donor schema registry** so each funder's format is handled declaratively.
- [ ] Set up **instructor alerts** and an **audit log** of every donor report generated.
- [ ] Track **cohorts** for reporting by county/region as donors require.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for instructors vs admins vs donors.
- [ ] Encrypt **minors' PII and learner data at rest and in transit**.
- [ ] Log every **learner-data access and donor-report generation** for audit.
- [ ] Keep records of **who/what/when** accessed learner data (DPC compliance).

## 9. Tax & finance checklist

- [ ] Register for **CIT** (25%) and **VAT** (15%).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties.
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **GIPC / Free Zones / one-district-one-factory** incentives where applicable.

## 10. Expansion notes (when scaling)

- Ghana sits in **ECOWAS** — factor regional data-flow (children's data) and licensing harmonisation into any cross-border play.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when Sankofa expands; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting. Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
