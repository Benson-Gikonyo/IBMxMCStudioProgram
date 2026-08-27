# Relfast — Retail & E-commerce · Nigeria — Launch Checklist & Guidelines

> Prepared for the IBM Dev Day whitehacking session. Pulls sector guidance from the **Sita Sector Live files** (retail & e-commerce playbook) and country parameters from the **[Country dossier: Nigeria](../africa-regulatory/nigeria.md)**. Treat every figure as a planning baseline — verify with the named authority before acting. Technology suggestions are recommendations, not mandates.

## 1. What Relfast is (working frame)

Assuming **Relfast** is a Nigerian omnichannel retailer / e-commerce brand ("relfast" = reliable + fast delivery). The Sita retail playbook assumes a retailer operating physical stores and an online shop with customer data siloed across POS, e-commerce, and a loyalty program — unified into a **golden customer profile** so marketing and returns work consistently.

**Reference pipeline** (from `retail/`): POS + ecommerce + loyalty → ingest → IBM MDM-style unification (match on email+phone) → CX events → next-best-action → personalised comms; unified returns handler across channels.

---

## 2. Country & sector fundamentals — Nigeria

| Item | Value | Verify with |
|---|---|---|
| Consumer-protection authority | Federal Competition & Consumer Protection Commission (FCCPC) | FCCPC |
| Data protection | Nigeria Data Protection Commission (NDPC) — register before consumer launch | NDPC |
| Revenue / tax | FIRS — VAT (7.5%) + digital-service tax where applicable | FIRS |
| Standards bureau | Standards Organisation of Nigeria (SON) — product compliance | SON |
| Blocs | AU; ECOWAS | — |
| Official / business language | English (also Hausa, Yoruba, Igbo, Pidgin) | — |
| Investment promotion | NIPC; NEPZA free-zones | NIPC / NEPZA |

---

## 3. Data model (build toward the retail playbook)

```
CustomerProfile  (golden / unified record)
  - profile_id (PK, UUID)
  - email: VARCHAR (deduplicated master key)
  - phone: VARCHAR
  - loyalty_tier: [bronze, silver, gold]
  - total_lifetime_value: DECIMAL
  - last_purchase_date: DATE
  - preferred_channel: [in_store, online, both]
  - mdm_confidence: DECIMAL
  - golden_resolved_at: TIMESTAMP

Order
  - order_id (PK)
  - profile_id (FK → CustomerProfile)
  - channel: [pos, woocommerce, social]
  - order_date: DATETIME
  - total_amount: DECIMAL
  - status: [pending, fulfilled, returned, cancelled]

OrderItem
  - item_id (PK)
  - order_id (FK → Order)
  - sku: VARCHAR
  - quantity: INTEGER
  - unit_price: DECIMAL
  - category: VARCHAR

CXEvent
  - event_id (PK)
  - profile_id (FK → CustomerProfile)
  - event_type: [browse, wishlist, cart_abandon, purchase, return, complaint]
  - occurred_at: DATETIME
  - channel: VARCHAR
  - metadata: JSONB
```

---

## 4. Data flow & architecture

```
[POS System] ─┐
[Ecommerce]  ─┼→ Ingest (real-time + batch connectors)
[Loyalty App] ─┘
    → Unify → golden CustomerProfile (match on email + phone)
    → [CXEvent stream]
    → Next-best-action model (recommend, retain, win-back)
    → [Personalised email/SMS trigger]
    → [Unified returns handler → update both POS and ecommerce]
```

**Reference implementation layers** (from `retail/`):
- `pipelines/pos_connector.js` — POS → Order + CXEvent
- `pipelines/woocommerce_connector.js` — ecommerce webhook handler (swap vendor)
- `pipelines/loyalty_sync.js` — loyalty tier sync
- `pipelines/mdm_unification.js` — golden profile builder (dedupe on email/phone)
- `api/routes/customers.js` — `GET /customers/:id`, history
- `api/routes/orders.js` — `GET /orders`, `POST /orders/return`
- `api/routes/cx_events.js` — `POST /cx-events` (ingest from channels)
- `api/services/watsonNBA.js` — next-best-action client
- `api/services/emailTrigger.js` — personalised comms dispatcher
- `tests/mdm_unification.test.js`, `tests/woocommerce_connector.test.js`

---

## 5. Technology build reference (recommendations — pick the fit)

| Layer | Recommended (open, low-friction) | IBM option (sugg.) | Notes |
|---|---|---|---|
| Connectors | WooCommerce/Shopify webhooks, custom POS sync, Zapier/n8n | IBM DataStage | POS, ecommerce, loyalty ingestion |
| Unification / dedupe | Postgres + pg_trgm fuzzy match; Segment / RudderStack alt | IBM MDM | Golden profile on email + phone |
| Personalisation | Recommendation libs, scikit-learn; store models | IBM Watson Studio | Purchase propensity + churn risk |
| CX automation | n8n, Zapier, custom flows | IBM Watson Orchestrate | Cart-abandon → reminder → offer |
| Comms | SendGrid, Twilio, Postmark | Watson (assistant) | Email/SMS triggers |
| Sandbox / deploy | Docker + CI (GitHub Actions) | IBM TechZone / CP4D | Prototyping + deployment |

> Use mock-first development and only add real credentials (IBM or otherwise) in a protected secret store.

---

## 6. Regulatory & consumer-protection checklist (Nigeria)

- [ ] Register with **NDPC** as a data controller before processing consumer data at scale.
- [ ] Comply with **FCCPC** consumer-protection rules — refunds, returns, and complaint handling for retail/e-commerce. **Notice:** Nigeria has strict consumer-protection enforcement on return/refund and on "software update" retro-upselling; factor this in.
- [ ] Register with **FIRS** for **VAT** (7.5%) and any **digital-service tax** on cross-border revenue.
- [ ] Ensure **SON** product/standards compliance for physical goods you sell.
- [ ] Publish clear **terms, return policy, and privacy notice** in plain language (English + major languages).
- [ ] Provide a working **complaints and redress** channel.
- [ ] Handle **minors' data** carefully if any product lines target children.

## 7. Product, data & integration checklist

- [ ] Build the **customer profile** model and unify across POS + ecommerce + loyalty (§3 schema).
- [ ] Map the **data flow**: ingest → unify (dedupe on email+phone) → CX events → next-best-action → comms; unified returns.
- [ ] Decide **mock-first** for integrations; promote with real credentials later.
- [ ] Wire **POS and ecommerce connectors** with real-time + batch ingestion.
- [ ] Implement the **unified returns** handler so a store return updates online and vice versa.
- [ ] Set up **CX event** capture across channels (browse, cart-abandon, purchase, return, complaint).
- [ ] Build **loyalty tier** sync and lifetime-value tracking on the golden profile.

## 8. Security & data-protection checklist

- [ ] Keep **API keys and credentials out of source control** — use `.env` + a `.gitignore` entry (the repo's `.gitignore` already excludes `.env*`, `credentials.txt`, `api-keys.txt`, `personal-info.md`). Never commit these.
- [ ] Rotate keys in **mock mode first**; only introduce real credentials in a protected secret store.
- [ ] Apply **least-privilege roles** for anyone touching customer profiles.
- [ ] Encrypt **PII at rest and in transit**; pseudonymise where the schema allows.
- [ ] Log every **customer-profile read and comms trigger** for audit.
- [ ] Keep records of **who/what/when** accessed customer data (NDPC compliance).

## 9. Tax & finance checklist

- [ ] Register for **CIT** (30%, or 25% SME rate if eligible) and **VAT** (7.5%).
- [ ] Plan **withholding tax** treatment on dividends/interest/royalties.
- [ ] Keep **transfer-pricing documentation** if part of a wider group.
- [ ] Check **NEPZA / free-zone and pioneer-status** incentives through NIPC where applicable.

## 10. Expansion notes (when scaling)

- Nigeria sits in **ECOWAS** — factor regional customs, data-flow and licensing harmonisation into any cross-border play.
- Reuse other [Africa Regulatory Dossiers](../africa-regulatory/README.md) when Relfast expands; each dossier carries the same sector map for its country.

---

**Status:** planning reference, verify before acting. Nothing here is legal, tax or financial advice — engage local counsel (Aikya) before commitment.
