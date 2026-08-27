# AILearning Box — Edtech · South Sudan — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (edtech playbook) and country parameters from the **[Country dossier: South Sudan](../africa-regulatory/south-sudan.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What AILearning Box is (working frame)

Assuming **AILearning Box** is an edtech learning platform ("AI learning" delivery, potentially offline/box-based given connectivity constraints) for South Sudan. The Sita edtech playbook assumes blended learning with learner-progress tracking plus **funder/donor impact reporting** — highly relevant in a donor-funded reconstruction market like South Sudan.

**Reference pipeline** (from `edtech/`): LMS activity → ingest → progress model → at-risk classifier → instructor alerts; donor-report transformer per donor schema → automated dispatch.

---

## 2. Country & sector fundamentals — South Sudan

| Item | Value | Verify with |
|---|---|---|
| Ministry / regulator (education) | Ministry of General Education & Instruction (moGEI) | MoGEI |
| Data protection | Verify national authority (no established DPA yet) | — |
| Revenue / tax | National Revenue Authority — CIT 25% (30% petroleum; verify), VAT 18% (verify) | NRA |
| Blocs | AU; EAC; IGAD | — |
| Official / business language | English (also Juba Arabic, Dinka, Nuer, Bari) | — |
| Investment promotion | South Sudan Investment Promotion Agency (verify) | SIPA |

> **Notice:** South Sudan is a **reconstruction / frontier market** — infrastructure (power, connectivity, banking) and legal/regulatory certainty are weaker. Build for **offline/low-bandwidth delivery** and donor-funded models; verify every regulatory and tax figure — the dossier marks several as uncertain.

---

## 3. Data model (build toward the edtech playbook)

```
Learner
  - learner_id (PK)
  - name: VARCHAR
  - county / state: VARCHAR
  - school_name: VARCHAR
  - cohort_id (FK → Cohort)
  - enrollment_date: DATE
  - learning_mode: [online, offline_box, blended]

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
[LMS / offline box activity]
    → Ingest (daily/batch extract + normalise; sync when connectivity is available)
    → [Progress table] (dedupe learner across touchpoints)
    → At-risk classification model
    → [Instructor alert: score < threshold or long inactivity]
    → [Donor Report engine: transform Progress → donor-specific schema]
    → Automated dispatch per donor schedule
```

**Reference implementation layers** (from `edtech/`):
- `pipelines/lms_ingest.js` — LMS / offline sync → Progress ingestion
- `pipelines/at_risk_classifier.py` — ML scoring job
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
| Ingestion | LMS API/webhooks; **offline-first sync (PWA, local-first)** | IBM DataStage | Daily extract + normalise; handle intermittent connectivity |
| Analytics / risk | Scikit-learn, XGBoost, FastAPI | IBM Watson Studio | At-risk model on completion velocity |
| Mastering | Postgres dedup | IBM MDM | Dedupe learners across touchpoints |
| Reporting | Templating + scheduler | Cognos / custom | Donor schema transformer |
| Alerts / comms | SMS (physical + mobile), in-app; **USSD/offline** | Watson Orchestrate / Assistant | Instructor alerts — plan for low-connectivity |
| Sandbox / deploy | Docker + CI (GitHub Actions); edge/offline boxes | IBM TechZone / CP4D | Prototyping + deployment |

> South Sudan connectivity reality: design the platform **offline-first** with sync-on-connect. Use mock-first development and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & compliance checklist (South Sudan)

- [ ] Liaise with the **Ministry of General Education & Instruction (moGEI)** for **content/curriculum accreditation** and teacher-program approval.
- [ ] **Verify the data-protection authority** — South Sudan has no well-established DPA; confirm the current regulator and registration threshold before consumer scale.
- [ ] Implement **enhanced data protection for minors' data** — parental consent, purpose limitation, minimal collection (best practice even where no DPA exists).
- [ ] Secure **intellectual-property and content licensing** for all learning materials.
- [ ] Publish clear **terms, privacy notice, and consent** in English and major South Sudanese languages (Juba Arabic, Dinka, Nuer, Bari).
- [ ] Prepare **donor/funder impact reporting** per agreed schema — critical to sustaining donor-funded programs.
- [ ] Confirm **NRA** registration and current **CIT/VAT** rates (figures marked "verify").

## 7. Product, data & integration checklist

- [ ] Build the **learner / module / progress / report** model (§3 schema).
- [ ] Map the **data flow**: LMS/offline ingest → progress → at-risk → alerts; donor transform → dispatch.
- [ ] Design **offline-first**: learners and instructors use local/box modes; data syncs when connectivity is available.
- [ ] Decide **mock-first** for integrations; promote with real credentials later.
- [ ] Wire **scheduled LMS/offline ingestion** and **at-risk scoring** with a threshold.
- [ ] Maintain a **donor schema registry** so each funder's format is handled declaratively.
- [ ] Set up **instructor alerts** (SMS/USSD/offline-synced) and an **audit log** of every donor report generated.
- [ ] Track **cohorts** for reporting by county/state as donors require.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for instructors vs admins vs donors.
- [ ] Encrypt **minors' PII and learner data at rest and in transit** — including data on offline boxes/edge devices.
- [ ] Log every **learner-data access and donor-report generation** for audit.
- [ ] Keep records of **who/what/when** accessed learner data.

## 9. Tax & finance checklist

- [ ] Register with the **National Revenue Authority**; confirm **CIT** (25%; 30% petroleum — verify) and **VAT** (18% — verify).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties.
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **SIPA / reconstruction and donor-program** incentives where applicable.

## 10. Expansion notes (when scaling)

- South Sudan sits in **EAC** and **IGAD** — factor regional data-flow and licensing harmonisation into any cross-border play; EAC influence on education standards may apply.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when AILearning Box expands; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting (especially the uncertain South Sudan figures). Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
