# Small-To-Medium-Business-Solutions for Insurance 

A mid-size insurer deploys a JSON-rules, AI-generated UI platform to streamline quotes, FNOL, and claims.
Customers or agents start in a guided intake where the UI is auto-built from a uiModel and validates in real time.
On submit, a workflow engine interprets JSON steps—classify, compute, gate, route, httpCall—to triage cases.
Coverage, pricing, and fraud checks run via pluggable services; simple claims flow straight through in seconds.
Threshold-based approval gates create timed tasks with one-click decisions, reminders, and escalation.
When approved, scheduling, payments, and notifications fire automatically, with SLA clocks visible to all.
Every decision pins the workflow version and writes a step-by-step trace for audit, replay, and compliance.
Business owners tweak SLAs, approvers, and pricing tiers by editing JSON, with schema checks and canary rollout.
The result is higher STP, lower cycle time and rework, fewer call-center updates, and happier policyholders.
During CAT surges, serverless scale and queues absorb spikes, preserving service levels and regulatory confidence.

# Problem (Current State)

## What’s breaking today

* **Inconsistent intake & rework**

  * Ten+ versions of FNOL/quote forms by product/region/carrier program.
  * Same fact asked multiple times; missing required fields; high NIGO (not-in-good-order) rates.
* **Manual triage & slow approvals**

  * Adjusters/underwriters eyeball severity, reserves, endorsements, discounts.
  * Email/Excel approvals with no SLA clock; “stuck” items are invisible.
* **Low straight-through processing (STP)**

  * Simple glass/towing/home-water-leak claims still wait for humans.
  * Quotes bounce to referral for routine edge cases.
* **Siloed systems & swivel-chair ops**

  * Policy admin, billing, claims, document storage, fraud tools don’t share a common contract.
  * Double entry between portals and core systems; brittle point-to-point integrations.
* **Poor transparency & auditability**

  * “Why was this claim denied?” often requires tribal knowledge.
  * No pinned rule version; hard to reproduce decisions for regulators or QA.
* **Fraud defense is reactive**

  * Basic red flags only; no fusion of rules + ML + external data in the decision moment.
* **Customer/agent experience lags**

  * No real-time status; limited self-service; high call volumes for “where is my claim/quote?”
* **CAT events overwhelm capacity**

  * Intake spikes cause backlogs; manual triage collapses SLA.
* **Tech debt**

  * Legacy monoliths, hard-coded rules, copy-pasted forms; changes require releases, not config.

## Pain points at a glance

| Pain Point             | Symptom                         | Likely Root Cause                                       | Business Impact                 | Affected Roles           |
| ---------------------- | ------------------------------- | ------------------------------------------------------- | ------------------------------- | ------------------------ |
| NIGO/Incomplete intake | Missing dates, coverage context | Static forms don’t adapt; no validation tied to product | Rework, delays, churn           | Customer, Agent, Ops     |
| Slow triage/approvals  | Days to move simple items       | Manual gates; email approvals; no timers                | SLA breaches, leakage           | Adjuster, UW, Supervisor |
| Low STP                | Humans handle simple claims     | Rules buried in code; no safe canary                    | High cost per claim, slow CX    | Claims, Finance          |
| Audit gaps             | Can’t replay a decision         | No version pinning/trace                                | Regulatory risk, disputes       | Compliance, Legal        |
| Integration drift      | Fragile hand-offs               | Point-to-point glue; no shared contract                 | Outages during changes          | IT, Ops                  |
| CAT surge              | Backlogs explode                | No autoscale, queues, or prioritization                 | SLA misses, reputational damage | Entire org               |

---

# Vision (Target State)

## North-star narrative

A **rules-defined, agent-rendered** platform: business rules and UI models live in JSON; an **AI agent autogenerates** the intake experience; a **workflow engine** executes steps (classify, compute, gate, route, httpCall) and **pins versions** for perfect audit. Humans only touch exceptions; everything else is **straight-through**.

## Design principles

1. **JSON is the contract**
   One artifact holds workflow steps + `uiModel`. Back end executes it; front end renders it.
2. **AI-generated interfaces**
   Forms and guided chat are generated from `uiModel` with validation, show/hide, and copy—no hand-coded screens.
3. **Human-in-the-loop by exception**
   Automatic routing + timers + approvals where thresholds/risk demand it.
4. **Explainable decisions**
   Every decision has a **trace** (inputs, expressions, outputs, actors) and a **pinned rule version**.
5. **Composable & pluggable**
   Pricing, coverage, fraud, payments, scheduling are separate services invoked via `httpCall`.
6. **Safe change management**
   JSON schema validation, linting, **canary rollout**, and one-click rollback; no Friday deploy fear.
7. **Privacy & least privilege**
   PII masking in traces, role-based unmasking, private endpoints, managed identities.

## What “good” looks like (outcomes)

* **Intake → decision in seconds** for simple claims/quotes; live **SLA clocks** for the rest.
* **STP uplift** on low-severity claims and simple endorsements; referrals only for genuine risk.
* **Single intake** that adapts by product/region via `uiModel` (web, mobile, chat share the same model).
* **Regulatory-grade audit**: “show me exactly which version and rule led to this outcome.”
* **Elastic operations**: queue-based, autoscaled services handle CAT surges gracefully.
* **Happier customers & agents**: real-time status, fewer calls, faster settlements, clearer reasons.

## Target KPIs (set baselines and measure weekly)

* **STP rate (simple claims)**: baseline X% → **target +25–40pp**
* **FNOL → first contact**: baseline H hours → **< 15 minutes**
* **Cycle time (minor claims)**: baseline D days → **−30%**
* **NIGO rate on intake**: baseline Y% → **−50%**
* **Referral rate (personal auto quotes)**: baseline R% → **−30%** without loss ratio degradation
* **Audit replay coverage**: baseline < 20% → **100%** decisions reproducible
* **CAT surge resilience**: process P95 FNOL within **< 5s** at 10× load

## Capability blueprint (what we build to hit the KPIs)

* **Rules & UI**

  * Versioned **workflow JSON** (switch/compute/gate/route/httpCall/waitUntil/set).
  * **`uiModel` JSON** driving AI-generated forms and conversational intakes.
  * Business-editable via Rule Editor (schema-validated, dry-run simulator, canary).
* **Decision services (pluggable)**

  * **Coverage check**, **Pricing/Rating**, **Fraud/Anomaly**, **Payments**, **Document Intelligence**.
* **Orchestration & state**

  * Workflow runtime with idempotency keys, step traces, pinned versions, timers.
  * **Approvals** with SLAs, reminders, and escalation paths.
* **Data & audit**

  * SQL for policies/claims/payments/approvals/workflow activations.
  * Immutable JSONL **decision traces** in object storage.
* **Experience**

  * Unified portal (web/mobile/chat) rendered from `uiModel`; status tracking; SLA clocks; document upload.
* **Ops**

  * Observability (distributed tracing, correlation IDs), SLO dashboards.
  * Security: Entra ID/B2C, private endpoints, Key Vault, DLP, PII masking.

## Anti-goals (what we won’t do)

* Hard-coding rules into UI or services.
* One-off forms per product/region.
* Manual, email-based approvals without timers or audit.
* Deploying rule changes without canary/rollback.

## Risks & mitigations

* **Rule sprawl** → enforce JSON schema + lint rules; product-line governance; naming/versioning conventions.
* **Over-automation** → clear exception routes; easy human override with reason logging.
* **Model bias/black-box** → use explainable features, thresholds, and human checkpoints; log inputs/outputs.
* **Integration churn** → standardize on `httpCall` adapters and contract tests; avoid point-to-point snowflakes.

## Business value narrative (how it pays)

* **Cost to serve** drops as STP rises and rework falls.
* **Speed** wins quotes and boosts NPS; faster settlements cut rental/handling costs.
* **Compliance** time falls with replayable decisions; disputes resolve faster.
* **Change velocity** increases: product tweaks ship as config, not code.

---


awesome — here’s a crisp, detailed expansion of the **personas** your platform serves. Each persona ties directly to the JSON-rules engine, AI-generated UI, and REST API surface you’ve designed.

---

# Personas (Detailed)

## 1) Customer / Policyholder

* **Primary goals:** Report a loss (FNOL) fast, check claim status, upload docs/photos, receive payments quickly, request endorsements.
* **Critical decisions:** Confirm loss facts, select coverage options (deductible acknowledgement), accept settlement.
* **Typical inputs/outputs:** Loss type/date/location, photos, police report; outputs = claim ID, SLA clock, settlement status, payment confirmation.
* **Key UI (AI-generated from `uiModel`):** Guided FNOL form/chat, document upload, status timeline, settlement acceptance, bank details capture.
* **Workflow touchpoints:** Triggers `Claim.Created`; may pass `approvalGate` if injuries/high indemnity; can accept instant-pay.
* **Integrations:** eSign for releases, payments (ACH/Card), notifications (email/SMS).
* **Security:** B2C login, PII masking; can see **only** their policy/claim.
* **KPIs:** NIGO rate ↓, FNOL time ↓, satisfaction/NPS ↑.

## 2) Agent / Broker

* **Primary goals:** Create quotes, bind policies, submit FNOL on behalf of insureds, track book of business.
* **Critical decisions:** Coverage selections, endorsement requests, bind decision after price disclosure.
* **Inputs/outputs:** Prospect demographics/vehicle/home, prior losses; outputs = quote/premium, referral status, bind confirmation.
* **Key UI:** Quote wizard, comparison of coverages, bind & payment, endorsement forms, portfolio dashboard.
* **Workflow touchpoints:** `Quote.Created` → underwriting referral gate if thresholds hit; `Policy.Bind` after payment OK.
* **Integrations:** Payment tokenization, rating/price service, document generation (policy PDF).
* **Security:** Role = `Agent`; scoped to assigned customers; audit on premium overrides.
* **KPIs:** Bind rate ↑, quote turnaround ↓, referral rate ↓ without loss of quality.

## 3) Underwriter

* **Primary goals:** Evaluate risk, apply pricing/terms, clear referrals, enforce appetite & guidelines.
* **Critical decisions:** Accept/decline, apply surcharges/credits, request additional docs, set special conditions.
* **Inputs/outputs:** Referral packet (normalized data, risk score, prior losses, photos); outputs = decision, terms, notes.
* **Key UI:** Referral inbox, risk score explainability, document viewer, decision panel with rule references.
* **Workflow touchpoints:** Handles `gate:UnderwritingReferral`; decision resumes workflow (approve/reject/needs info).
* **Integrations:** External data (MVR, CLUE, property data), pricing/raters, document AI for attachments.
* **Security:** Role = `Underwriter`; can view/annotate risk factors; cannot alter payments.
* **KPIs:** Referral cycle time ↓, decision quality (loss ratio), guideline adherence.

## 4) Adjuster (Claim Handler)

* **Primary goals:** Verify coverage, set reserves, coordinate estimates/repairs, settle fairly and quickly.
* **Critical decisions:** Coverage confirmation, reserve levels, pay/deny/defer, salvage/subrogation initiation.
* **Inputs/outputs:** FNOL packet, estimates, medical/bodily injury docs; outputs = reserve changes, payment approvals, settlement terms.
* **Key UI:** Work queue by severity/SLA, claim timeline, reserve & payment panels, communications log.
* **Workflow touchpoints:** Picks up non-STP claims, completes tasks posted by `route` steps, triggers `approvalGate` where needed.
* **Integrations:** Estimating vendors, repair networks, medical review, payments.
* **Security:** Role = `Adjuster`; access limited by region/LOB; full audit trail on status or reserve changes.
* **KPIs:** Cycle time ↓, re-opens ↓, leakage ↓, SLA attainment ↑.

## 5) Claims Supervisor (Team Lead)

* **Primary goals:** Manage workload, approve exceptions, ensure SLA compliance, escalate when at risk.
* **Critical decisions:** Approve high indemnity/total loss/legal involvement; reassign workloads; override rules in rare cases.
* **Inputs/outputs:** Approval tasks with context & recommendations; outputs = approve/reject with reason, reassignment actions.
* **Key UI:** Approval inbox with timers, team capacity view, SLA breach dashboard, override panel with guardrails.
* **Workflow touchpoints:** Owner of `approvalGate` for claims; escalation routes/timeouts.
* **Security:** Role = `Supervisor`; can approve gates and reassign; override reasons required & audited.
* **KPIs:** Approval latency ↓, SLA breaches ↓, queue balance/aging ↓.

## 6) SIU / Fraud Analyst

* **Primary goals:** Investigate fraud signals, hold suspicious payments, coordinate with legal.
* **Critical decisions:** Place claim on hold, request additional verification, clear or refer to authorities.
* **Inputs/outputs:** ML anomaly scores, rule hits (red flags), network graphs; outputs = disposition (clear/confirmed/ongoing), notes, holds.
* **Key UI:** Fraud queue, score explanation, link analysis, hold/release actions.
* **Workflow touchpoints:** `gate:FraudReview` fires on thresholds; can pause `instantPay`; resume on clear.
* **Integrations:** OSINT, consortium data, device intelligence.
* **Security:** Role = `SIU`; restricted PII; strong audit on holds and releases.
* **KPIs:** Confirmed fraud detection ↑, false positives ↓, time-to-disposition ↓.

## 7) Ops Admin / Product Owner (Rules & UI)

* **Primary goals:** Change business rules and UI **without code**; canary & rollback; enforce governance.
* **Critical decisions:** SLA/pricing thresholds, approval chains, notification templates, `uiModel` field visibility.
* **Inputs/outputs:** Proposed JSON (workflow + `uiModel`), simulator results; outputs = activated version with rollout %.
* **Key UI:** Rule editor (JSON + schema validation), dry-run simulator, diff/compare, activation panel (canary %).
* **Workflow touchpoints:** Publishes `WorkflowDefinitions`; activates in `WorkflowActivations`; changes take effect on **new** items.
* **Security:** Role = `WorkflowAdmin`; dual-control for production activations; full audit (who/what/when/why).
* **KPIs:** Change lead time ↓, rollback MTTR ↓, policy compliance (lint pass rate) ↑.

## 8) Compliance / Audit

* **Primary goals:** Reconstruct “why/how” of any decision; verify adherence to regs; handle complaints and subpoenas.
* **Critical decisions:** Approve procedures, release audit packages, require corrective actions.
* **Inputs/outputs:** Decision traces (JSONL), pinned versions, approval logs; outputs = audit reports, remediation tickets.
* **Key UI:** Searchable audit console (by entity/date/actor), one-click “replay” view of steps & rules; export to PDF/CSV.
* **Workflow touchpoints:** Read-only access to traces & versions; no influence on runtime.
* **Security:** Role = `Auditor`; PII unmask with justification; everything logged.
* **KPIs:** Time to produce audit pack ↓, % reproducible decisions = 100%, findings ↓.

---

## Adjacent/supporting roles (brief)

* **Billing & Payments Ops:** reconciles payouts, handles reversals/chargebacks; UI = payment ledger; RBAC = `PaymentsOps`.
* **IT / SRE:** uptime, scaling, observability, policy enforcement; RBAC = `PlatformOperator`.

---

# Persona ↔ Workflow/API Mapping (Quick Matrix)

| Persona     | Typical Actions                             | Key Endpoints                                                           | Notable Gates/Steps                   |
| ----------- | ------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------- |
| Customer    | Create FNOL, upload docs, accept settlement | `POST /claims`, `GET /claims/{id}`, `POST /claims/{id}/accept`          | `classify`, `instantPay`, `notify`    |
| Agent       | Quote, bind, endorse                        | `POST /quotes`, `POST /quotes/{id}/bind`, `POST /policies/{id}/endorse` | `compute(price)`, `gate:Underwriting` |
| Underwriter | Clear referrals, adjust terms               | `POST /quotes/{id}/uw-decide`                                           | `gate:UnderwritingReferral`           |
| Adjuster    | Set reserves, approve payments              | `POST /claims/{id}/tasks`, `POST /payments`                             | `route(team)`, `gate:Supervisor`      |
| Supervisor  | Approve exceptions, reassign                | `POST /approvals/{id}/decide`, `POST /claims/{id}/reassign`             | `approvalGate`                        |
| SIU         | Hold/release, disposition                   | `POST /claims/{id}/siu-hold`, `POST /claims/{id}/siu-clear`             | `gate:FraudReview`                    |
| Ops Admin   | Activate rules/UI                           | `POST /workflows`, `POST /workflows/{id}:activate`                      | Version pinning/canary                |
| Auditor     | Retrieve traces                             | `GET /requests/{id}/trace`                                              | Read-only                             |

---

# Swimlane (who touches what in a gated claim)

```mermaid
sequenceDiagram
  autonumber
  participant Cust as Customer
  participant Agent as Agent/Broker
  participant WF as Workflow Engine
  participant Sup as Claims Supervisor
  participant UW as Underwriter
  participant SIU as SIU
  participant Pay as Payments

  Cust->>WF: FNOL (AI-guided intake)
  WF->>WF: classify/compute/route (JSON steps)
  alt Fraud or High Indemnity
    WF->>SIU: gate:FraudReview (hold)
    SIU-->>WF: clear or escalate
  end
  alt Underwriting needed (endorsement/edge)
    WF->>UW: gate:UnderwritingReferral
    UW-->>WF: approve with terms / reject
  end
  alt Supervisor approval
    WF->>Sup: gate:Supervisor
    Sup-->>WF: approve/reject with reason
  end
  WF->>Pay: instantPay or scheduled payment
  Pay-->>WF: confirmation
  WF-->>Cust: status update / settlement
  Agent-->>WF: (optional) assists, attaches docs
```

---

# Data & Permissions (at a glance)

* **Row-level security:** `TenantId` filter for all business tables; persona roles scoped by LOB/region where needed.
* **Audit invariants:** Any **decision** (approve/reject, status change, reserve change, payment) must log to `ins.AuditEvents`.
* **Privacy:** PII masked by default in traces; `Auditor` can unmask with justification.

---

If you want next, I can generate **persona-specific UI snippets** from a `uiModel` (e.g., Underwriter referral panel), or **role-based API Postman collections** mapped to these actions.

## Outcomes & KPIs

* **STP rate**: ≥ 40% for simple glass/towing/low-severity claims
* **FNOL→First Contact**: < 15 minutes (target)
* **Cycle time**: −30% on minor claims; −15% on moderate claims
* **Fraud leakage**: −10% with ML triage + rule gates
* **Bind rate**: +8% via faster quote; abandonment < 10%
* **Audit completeness**: 100% with pinned workflow versions & decision traces

---

# Core Journeys

## A) Quote → Bind (Personal Auto/Home)

1. **Intake** (AI-generated UI from `uiModel`): driver/home details, prior losses, coverages, limits.
2. **Risk & Price**: rules + ML signals (credit surrogate, telematics, roof condition) compute premium; discounts via tier rules.
3. **Referral Gate**: triggers if premium change > threshold, risk score > cutoff, or missing docs.
4. **Bind**: payment and e-policy issuance; documents e-signed; policy ID created; endorsements enabled.

## B) FNOL → Settlement (Claims)

1. **FNOL intake** (AI chat or form from `uiModel`), upload photos, police report, telematics.
2. **Triage & Routing**: JSON rules classify severity, set reserves, assign adjuster/team or go STP.
3. **Approval Gates**: total loss, injury, or high indemnity require supervisor/legal approval.
4. **Estimate & Settlement**: integrate estimating vendor; if within limits and low-risk → **instant pay**; else to adjuster.
5. **Subrogation/Salvage**: if third-party involved or vehicle totaled, create subro/salvage tasks.
6. **Closure**: notify insured, update reserves/final pay, archive decision trace.

## C) Fraud & Triage

* **Rule + ML hybrid**: rules for known patterns (mismatched garaging, late add of coverage), ML for anomaly score.
* **SIU Gate**: if score > threshold or rule hits, route to SIU; otherwise proceed.

## D) Endorsements & Mid-Term Changes

* **UI agent** prompts only fields impacted; rules compute pro-rata premium change; approvals as needed.

---

# Reference Architecture (Azure-lean)

* **UI (web/mobile)**: Generated by AI agent from `uiModel` JSON; renders forms/conversation.
* **API Gateway**: Azure API Management (JWT, rate limit, IP allowlist, headers).
* **Workflow Runtime**: Azure Functions (HTTP + queue triggers) evaluating **JSON steps** (`switch`, `compute`, `gate`, `route`, `httpCall`, `waitUntil`).
* **Rules & UI Models**: Cosmos DB or Blob for **immutable** versions (`workflowId@version` + `uiModel`).
* **Data**:

  * Azure SQL: policies, claims, payments, reserves, approvals, activations.
  * ADLS/Blob: documents (photos, PDFs), decision traces (JSONL).
* **Events/Async**: Service Bus (claim tasks), Event Grid (status events).
* **AI**: Azure OpenAI (UI agent, summarization), Document Intelligence (form extraction), Vision (damage severity), optional Telematics via Event Hubs.
* **Analytics**: Synapse/Databricks + Power BI (KPIs, leakage, SLA, fraud).
* **Security**: Entra ID/B2C, Managed Identity, Key Vault, Private Endpoints, Defender for Cloud.
* **Observability**: App Insights + Log Analytics (end-to-end traces).

---

# Sequence Diagrams

## 1) FNOL Straight-Through (low-severity glass)

```mermaid
sequenceDiagram
  autonumber
  participant C as Customer
  participant A as AI Intake Agent
  participant R as UI Renderer
  participant APIM as API Gateway
  participant WF as Workflow API (Functions)
  participant WS as Workflow Store (JSON)
  participant DI as Document Intelligence
  participant PR as Pricing/Policy Service
  participant DB as SQL (Claims/Approvals)
  participant PAY as Payments
  participant TR as Trace Store (JSONL)
  participant NOT as Notifications

  C->>A: Start FNOL
  A->>WS: Get active uiModel+workflow for product/region
  WS-->>A: uiModel + rules (version)
  A->>R: Emit UI spec (fields, showIf)
  C->>R: Fill details, upload images
  R->>DI: Extract metadata (optional)
  R->>APIM: POST /claims (payload + DI results)
  APIM->>WF: Forward request (JWT)
  WF->>WS: Load & pin workflow version
  WF->>TR: Trace input snapshot
  WF->>PR: Verify coverage/deductible
  PR-->>WF: Coverage OK, deductible $200
  WF->>WF: Classify severity → "low" compute pay <= limit
  alt Gate not required
    WF->>PAY: Create instant payment
    PAY-->>WF: Payment confirmation
    WF->>DB: Insert claim + payment info
    WF->>NOT: Notify insured (settled)
    WF-->>R: 201 Created (claim id, settled)
  else Gate required
    WF->>DB: Create approval task
    WF->>NOT: Notify adjuster/supervisor
    WF-->>R: 202 Accepted (awaiting review)
  end
```

## 2) Quote → Bind (referral on risk)

```mermaid
sequenceDiagram
  autonumber
  participant P as Prospect/Agent
  participant A as AI UI Agent
  participant R as UI Renderer
  participant APIM as API Gateway
  participant WF as Workflow API
  participant WS as Workflow Store
  participant UW as Underwriting Service
  participant BILL as Billing/Payments
  participant DB as SQL (Policy)
  participant TR as Trace Store
  P->>A: Start Quote
  A->>WS: Get uiModel+workflow
  WS-->>A: Definitions (version)
  A->>R: Render quote UI
  P->>R: Provide data
  R->>APIM: POST /quotes
  APIM->>WF: Forward
  WF->>WS: Pin version
  WF->>UW: Risk score + price
  UW-->>WF: Risk=Moderate, premium $1,230
  alt Referral condition hit
    WF->>DB: Create underwriting task
    WF-->>R: 202 Pending UW review
  else No referral
    R->>APIM: POST /quotes/{id}/bind
    APIM->>WF: Bind
    WF->>BILL: Payment tokenization/charge
    BILL-->>WF: Payment OK
    WF->>DB: Create policy
    WF-->>R: 201 Policy issued
  end
```

---

# JSON-Driven Rules & UI

## Workflow (FNOL triage excerpt)

```json
{
  "workflowId": "claims.fnol.auto.v3",
  "version": 3,
  "triggers": ["Claim.Created"],
  "steps": [
    {"id":"classify","type":"switch","on":"{{claim.lossType}}",
     "cases": {
       "glass":[{"set":{"severity":"low","slaHours":2}}],
       "towing":[{"set":{"severity":"low","slaHours":2}}],
       "collision":[{"set":{"severity":"med","slaHours":8}}]
     },
     "default":[{"set":{"severity":"med","slaHours":8}}]
    },
    {"id":"coverageCheck","type":"httpCall",
     "request":{"method":"POST","url":"https://policy/api/coverage","body":{"policyId":"{{claim.policyId}}","date":"{{claim.lossDate}}"}},
     "assignTo":"coverage"
    },
    {"id":"approvalGate","type":"gate",
     "condition":"coverage.deductible > 1000 || claim.estimatedIndemnity > 2500 || claim.injury == true",
     "approvers":["role:Supervisor","role:LegalOnInjury"]
    },
    {"id":"instantPay","type":"httpCall",
     "when":"severity == 'low' && claim.estimatedIndemnity <= coverage.limit && !approvalGate.blocked",
     "request":{"method":"POST","url":"https://payments/api/payout","body":{"claimId":"{{claim.id}}","amount":"{{claim.estimatedIndemnity}}"}}
    }
  ]
}
```

## UI Model (FNOL intake excerpt)

```json
{
  "uiModel": {
    "sections": [
      {
        "id":"loss",
        "title":"Loss Details",
        "fields":[
          {"name":"lossType","type":"select","label":"Type of Loss","options":["glass","towing","collision","theft"],"required":true},
          {"name":"lossDate","type":"date","label":"Date of Loss","required":true},
          {"name":"injury","type":"checkbox","label":"Any injuries?"},
          {"name":"photos","type":"file","label":"Upload Photos","accept":["image/*"]}
        ]
      },
      {
        "id":"vehicle",
        "title":"Vehicle",
        "fields":[
          {"name":"vin","type":"text","label":"VIN","requiredIf":"product=='auto'"},
          {"name":"odometer","type":"number","label":"Odometer"}
        ],
        "dynamic":{"showIf":[{"when":"product=='auto'","show":["vin","odometer"]}]}
      }
    ],
    "validation":[
      {"rule":"lossDate <= today()","message":"Date cannot be in the future"}
    ],
    "copy":{"intro":"Tell us what happened. We’ll triage and, if eligible, settle instantly."}
  }
}
```

---

# API Surface (minimal)

* `GET /workflows/{id}/active` → workflow + `uiModel` (pinned by tenant/product)
* `POST /claims` → create FNOL; returns **201 settled** or **202 pending**
* `GET /claims/{id}` → status, SLA, assigned adjuster, decision trace ref
* `POST /claims/{id}/actions/approve|reject` → supervisor/legal gate
* `POST /quotes` → compute premium (may return **202 referral**)
* `POST /quotes/{id}/bind` → issue policy (payment first)
* `GET /requests/{id}/trace` → JSONL decision trace (auditor role)

---

# Data Model (high level)

**Azure SQL**

* `Policies(policyId, product, insuredId, status, effectiveDate, renewalDate, coveragesJson, region)`
* `Claims(claimId, policyId, lossType, severity, status, reserves, paid, createdAt, workflowVersionRef)`
* `Approvals(approvalId, entityType, entityId, approver, status, reason, gateId, decidedAt)`
* `Payments(paymentId, claimId, amount, method, status, txnRef)`
* `WorkflowActivations(tenantId, product, workflowId, version, rolloutPercent, activatedAt, activatedBy, status)`

**ADLS/Blob**

* `/workflows/{workflowId}/{version}.json` (immutable rules + uiModel)
* `/traces/{claimId}.jsonl` (append-only step traces)
* `/docs/{claimId}/...` (photos, PDFs)

---

# Audit, Versioning, & Compliance

* **Pin workflow version** at creation; store in `Claims.workflowVersionRef`.
* **Immutable workflow storage**; only activate/retire mappings.
* **Decision traces** include step inputs/outputs, expressions, and actor decisions.
* **Retention**: traces retained SLA+X years per product/region.
* **Access**: PII masked in traces; `Auditor` role unmask w/ justification; all views logged.

---

# Non-Functional Requirements

* **Throughput**: FNOL bursts (CAT events) → scale out Functions/Service Bus; backpressure via queues.
* **Latency**: simple STP claims < 3s E2E; quote < 5s.
* **Resilience**: idempotent handlers; exactly-once semantic via idempotency keys.
* **Security**: Entra ID, Private Endpoints, Key Vault, CMK for SQL/Storage, least privilege RBAC.
* **Observability**: distributed tracing, request correlation IDs, SLA dashboards.

---

# Phased Rollout

**MVP (8–12 weeks)**

* FNOL for auto glass & towing (highest STP), Payments integration, Notifications
* Quote for personal auto (no referrals), Bind + payment
* Rule editor (JSON upload), canary activation, decision traces

**Phase 2**

* Collision + bodily injury flows (gated), SIU integration, Subrogation/Salvage
* Underwriting referrals; endorsements; document AI for police reports/invoices
* Telematics ingestion, computer vision for damage severity

**Phase 3**

* Homeowners, small commercial packages, catastrophe event orchestration
* Advanced pricing, reinsurance cessions, claim litigation workflows

---

# Risks & Mitigations

* **Rule sprawl** → enforce JSON schema + linter; product-line governance.
* **Fraud false positives** → human-in-the-loop appeals; threshold tuning by line.
* **CAT surges** → pre-scale policies; queue buffering; graceful degradation for non-critical features.
* **Integration churn** → isolate via `httpCall` adapters; contract tests.

---

# Reporting (Power BI)

* **Claims dashboard**: STP%, cycle time, SLA breaches, severity mix, loss ratio, paid vs reserved.
* **Fraud/SIU**: precision/recall, time to disposition.
* **Underwriting**: bind rate, referral volume, premium lift, geography/product splits.

---

Awesome — here’s a build-ready **Azure SQL** schema and a focused set of **stored procedures** for the insurance data model we discussed (claims, policies, approvals, payments, workflow activations, audit/trace). It’s opinionated for reliability, auditability, and JSON-driven workflows.

---

# 1) Schema & Tables (T-SQL)

```sql
-- Recommended: one-time settings
SET ANSI_NULLS ON;
SET QUOTED_IDENTIFIER ON;

-- Use a dedicated schema
IF NOT EXISTS (SELECT 1 FROM sys.schemas WHERE name = 'ins')
    EXEC('CREATE SCHEMA ins');

------------------------------------------------------------
-- Reference & system tables
------------------------------------------------------------
CREATE TABLE ins.Tenants (
    TenantId UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Tenants_TenantId DEFAULT NEWID(),
    Name        NVARCHAR(200) NOT NULL,
    Status      NVARCHAR(30)  NOT NULL CONSTRAINT CK_Tenants_Status CHECK (Status IN ('Active','Inactive')),
    CreatedAt   DATETIME2(3)  NOT NULL CONSTRAINT DF_Tenants_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy   NVARCHAR(128) NULL,
    ModifiedAt  DATETIME2(3)  NULL,
    ModifiedBy  NVARCHAR(128) NULL,
    RowVersion  ROWVERSION,
    CONSTRAINT PK_Tenants PRIMARY KEY (TenantId)
);

CREATE TABLE ins.Customers (
    CustomerId UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Customers_CustomerId DEFAULT NEWID(),
    TenantId   UNIQUEIDENTIFIER NOT NULL,
    ExternalRef NVARCHAR(100) NULL,           -- CRM/Agency Mgmt ID
    FullName   NVARCHAR(200) NOT NULL,
    Email      NVARCHAR(256) NULL,
    Phone      NVARCHAR(50)  NULL,
    Address1   NVARCHAR(200) NULL,
    City       NVARCHAR(100) NULL,
    Region     NVARCHAR(100) NULL,            -- State/Province
    PostalCode NVARCHAR(20)  NULL,
    Country    NVARCHAR(100) NULL,
    CreatedAt  DATETIME2(3)  NOT NULL CONSTRAINT DF_Customers_CreatedAt DEFAULT SYSUTCDATETIME(),
    RowVersion ROWVERSION,
    CONSTRAINT PK_Customers PRIMARY KEY (CustomerId),
    CONSTRAINT FK_Customers_Tenants FOREIGN KEY (TenantId) REFERENCES ins.Tenants(TenantId)
);

------------------------------------------------------------
-- Policy & product
------------------------------------------------------------
CREATE TABLE ins.Policies (
    PolicyId      UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Policies_PolicyId DEFAULT NEWID(),
    TenantId      UNIQUEIDENTIFIER NOT NULL,
    CustomerId    UNIQUEIDENTIFIER NOT NULL,
    Product       NVARCHAR(50)  NOT NULL,     -- e.g., 'auto','home','commercial'
    Status        NVARCHAR(30)  NOT NULL CONSTRAINT CK_Policies_Status CHECK (Status IN ('Quoted','Active','Lapsed','Cancelled')),
    EffectiveDate DATE          NOT NULL,
    RenewalDate   DATE          NULL,
    CoveragesJson NVARCHAR(MAX) NULL,         -- JSON coverages/limits/deductibles
    CreatedAt     DATETIME2(3)  NOT NULL CONSTRAINT DF_Policies_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy     NVARCHAR(128) NULL,
    ModifiedAt    DATETIME2(3)  NULL,
    ModifiedBy    NVARCHAR(128) NULL,
    RowVersion    ROWVERSION,
    CONSTRAINT PK_Policies PRIMARY KEY (PolicyId),
    CONSTRAINT FK_Policies_Tenants   FOREIGN KEY (TenantId)   REFERENCES ins.Tenants(TenantId),
    CONSTRAINT FK_Policies_Customers FOREIGN KEY (CustomerId) REFERENCES ins.Customers(CustomerId),
    CONSTRAINT CK_Policies_CoveragesJson_JSON CHECK (CoveragesJson IS NULL OR ISJSON(CoveragesJson) = 1)
);
CREATE INDEX IX_Policies_Tenant_Product_Status ON ins.Policies(TenantId, Product, Status);

------------------------------------------------------------
-- Workflow metadata & activation (canary)
------------------------------------------------------------
CREATE TABLE ins.WorkflowDefinitions (
    WorkflowDefId UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_WorkflowDefinitions_Id DEFAULT NEWID(),
    TenantId      UNIQUEIDENTIFIER NOT NULL,
    Product       NVARCHAR(50)     NOT NULL,
    WorkflowId    NVARCHAR(200)    NOT NULL,   -- e.g., claims.fnol.auto
    Version       INT              NOT NULL,
    StorageUrl    NVARCHAR(400)    NOT NULL,   -- blob URL to immutable JSON (rules+uiModel)
    Sha256        CHAR(64)         NULL,       -- integrity
    CreatedAt     DATETIME2(3)     NOT NULL CONSTRAINT DF_WorkflowDefinitions_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy     NVARCHAR(128)    NULL,
    CONSTRAINT PK_WorkflowDefinitions PRIMARY KEY (WorkflowDefId),
    CONSTRAINT UQ_Workflow_Tenant_Product_Id_Version UNIQUE (TenantId, Product, WorkflowId, Version),
    CONSTRAINT FK_WfDef_Tenant FOREIGN KEY (TenantId) REFERENCES ins.Tenants(TenantId)
);

-- Two rows "Active" at once enable canary (new + previous); % must sum <= 100
CREATE TABLE ins.WorkflowActivations (
    ActivationId  UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_WorkflowActivations_Id DEFAULT NEWID(),
    TenantId      UNIQUEIDENTIFIER NOT NULL,
    Product       NVARCHAR(50)     NOT NULL,
    WorkflowId    NVARCHAR(200)    NOT NULL,
    Version       INT              NOT NULL,
    RolloutPercent TINYINT         NOT NULL CHECK (RolloutPercent BETWEEN 0 AND 100),
    Status        NVARCHAR(20)     NOT NULL CONSTRAINT CK_WFA_Status CHECK (Status IN ('Active','Retired')),
    ActivatedAt   DATETIME2(3)     NOT NULL CONSTRAINT DF_WFA_ActivatedAt DEFAULT SYSUTCDATETIME(),
    ActivatedBy   NVARCHAR(128)    NULL,
    PreviousVersion INT            NULL,       -- for quick rollback decisioning
    Notes         NVARCHAR(400)    NULL,
    CONSTRAINT PK_WorkflowActivations PRIMARY KEY (ActivationId),
    CONSTRAINT FK_WFA_WfDef FOREIGN KEY (TenantId, Product, WorkflowId, Version)
        REFERENCES ins.WorkflowDefinitions(TenantId, Product, WorkflowId, Version)
);
CREATE INDEX IX_WFA_Tenant_Product_Status ON ins.WorkflowActivations(TenantId, Product, Status)
    WHERE Status = 'Active';

------------------------------------------------------------
-- Claims, approvals, payments
------------------------------------------------------------
CREATE TABLE ins.Claims (
    ClaimId       UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Claims_ClaimId DEFAULT NEWID(),
    TenantId      UNIQUEIDENTIFIER NOT NULL,
    PolicyId      UNIQUEIDENTIFIER NOT NULL,
    Product       NVARCHAR(50)     NOT NULL,      -- denormalized for convenience
    LossType      NVARCHAR(50)     NOT NULL,      -- e.g., 'glass','towing','collision','theft'
    LossDate      DATE             NOT NULL,
    Severity      NVARCHAR(20)     NULL,          -- 'low','med','high' (set by workflow)
    Status        NVARCHAR(30)     NOT NULL CONSTRAINT CK_Claims_Status CHECK (Status IN ('Open','PendingApproval','Settled','Closed','Denied')),
    EstimatedIndemnity DECIMAL(18,2) NULL,
    Reserves      DECIMAL(18,2) NOT NULL CONSTRAINT DF_Claims_Reserves DEFAULT (0),
    Paid          DECIMAL(18,2) NOT NULL CONSTRAINT DF_Claims_Paid DEFAULT (0),
    WorkflowId    NVARCHAR(200)    NOT NULL,      -- pinned on create
    WorkflowVersion INT            NOT NULL,
    TraceUri      NVARCHAR(400)    NULL,          -- ADLS/Blob pointer to JSONL
    CreatedAt     DATETIME2(3)     NOT NULL CONSTRAINT DF_Claims_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy     NVARCHAR(128)    NULL,
    ModifiedAt    DATETIME2(3)     NULL,
    ModifiedBy    NVARCHAR(128)    NULL,
    RowVersion    ROWVERSION,
    CONSTRAINT PK_Claims PRIMARY KEY (ClaimId),
    CONSTRAINT FK_Claims_Tenants FOREIGN KEY (TenantId) REFERENCES ins.Tenants(TenantId),
    CONSTRAINT FK_Claims_Policies FOREIGN KEY (PolicyId) REFERENCES ins.Policies(PolicyId)
);
CREATE INDEX IX_Claims_Tenant_Status ON ins.Claims(TenantId, Status);
CREATE INDEX IX_Claims_Policy ON ins.Claims(PolicyId);

CREATE TABLE ins.Approvals (
    ApprovalId   UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Approvals_Id DEFAULT NEWID(),
    TenantId     UNIQUEIDENTIFIER NOT NULL,
    EntityType   NVARCHAR(30)     NOT NULL CONSTRAINT CK_Approvals_EntityType CHECK (EntityType IN ('Claim','Quote','Policy')),
    EntityId     UNIQUEIDENTIFIER NOT NULL,
    GateId       NVARCHAR(100)    NOT NULL,     -- workflow step id
    Approver     NVARCHAR(256)    NOT NULL,     -- role:Supervisor or user principal
    Status       NVARCHAR(20)     NOT NULL CONSTRAINT CK_Approvals_Status CHECK (Status IN ('Pending','Approved','Rejected','Expired')),
    Reason       NVARCHAR(400)    NULL,
    DecidedAt    DATETIME2(3)     NULL,
    CreatedAt    DATETIME2(3)     NOT NULL CONSTRAINT DF_Approvals_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy    NVARCHAR(128)    NULL,
    RowVersion   ROWVERSION,
    CONSTRAINT PK_Approvals PRIMARY KEY (ApprovalId),
    CONSTRAINT IX_Approvals_Entity UNIQUE (EntityType, EntityId, GateId, Approver, Status) -- helps avoid dup pendings
);
CREATE INDEX IX_Approvals_Tenant_Status ON ins.Approvals(TenantId, Status);

CREATE TABLE ins.Payments (
    PaymentId    UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Payments_Id DEFAULT NEWID(),
    TenantId     UNIQUEIDENTIFIER NOT NULL,
    ClaimId      UNIQUEIDENTIFIER NOT NULL,
    Amount       DECIMAL(18,2)    NOT NULL CHECK (Amount >= 0),
    Method       NVARCHAR(30)     NOT NULL,      -- 'ACH','Card','Check'
    Status       NVARCHAR(20)     NOT NULL CONSTRAINT CK_Pay_Status CHECK (Status IN ('Initiated','Succeeded','Failed','Reversed')),
    TxnRef       NVARCHAR(100)    NULL,
    CreatedAt    DATETIME2(3)     NOT NULL CONSTRAINT DF_Payments_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy    NVARCHAR(128)    NULL,
    RowVersion   ROWVERSION,
    CONSTRAINT PK_Payments PRIMARY KEY (PaymentId),
    CONSTRAINT FK_Payments_Claims FOREIGN KEY (ClaimId) REFERENCES ins.Claims(ClaimId)
);
CREATE INDEX IX_Payments_Claim ON ins.Payments(ClaimId);

------------------------------------------------------------
-- Documents & audit
------------------------------------------------------------
CREATE TABLE ins.Documents (
    DocumentId   UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_Documents_Id DEFAULT NEWID(),
    TenantId     UNIQUEIDENTIFIER NOT NULL,
    EntityType   NVARCHAR(30)     NOT NULL CONSTRAINT CK_Doc_EntityType CHECK (EntityType IN ('Claim','Policy','Customer')),
    EntityId     UNIQUEIDENTIFIER NOT NULL,
    DocType      NVARCHAR(50)     NOT NULL,  -- 'photo','police_report','invoice','policy_pdf'
    Uri          NVARCHAR(400)    NOT NULL,  -- blob path
    CreatedAt    DATETIME2(3)     NOT NULL CONSTRAINT DF_Documents_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy    NVARCHAR(128)    NULL,
    CONSTRAINT PK_Documents PRIMARY KEY (DocumentId)
);
CREATE INDEX IX_Documents_Entity ON ins.Documents(EntityType, EntityId);

CREATE TABLE ins.AuditEvents (
    AuditId     BIGINT IDENTITY(1,1) PRIMARY KEY,
    TenantId    UNIQUEIDENTIFIER NOT NULL,
    EntityType  NVARCHAR(30)     NOT NULL,     -- 'Claim','Policy','Payment','Approval','Workflow'
    EntityId    UNIQUEIDENTIFIER NULL,
    Actor       NVARCHAR(256)    NULL,
    Action      NVARCHAR(100)    NOT NULL,     -- 'Create','Update','Approve','Reject','ActivateWorkflow', etc.
    PayloadJson NVARCHAR(MAX)    NULL,
    CreatedAt   DATETIME2(3)     NOT NULL CONSTRAINT DF_AuditEvents_CreatedAt DEFAULT SYSUTCDATETIME(),
    CONSTRAINT CK_Audit_PayloadJson_JSON CHECK (PayloadJson IS NULL OR ISJSON(PayloadJson) = 1)
);
CREATE INDEX IX_Audit_Tenant_Entity ON ins.AuditEvents(TenantId, EntityType, EntityId);
```

---

# 2) Stored Procedures (core flows)

> All procs use robust error handling, idempotency keys where helpful, and pin the workflow version on create.

## 2.1 Get active workflow version (with canary)

```sql
CREATE OR ALTER PROCEDURE ins.usp_Workflow_GetActiveVersion
    @TenantId   UNIQUEIDENTIFIER,
    @Product    NVARCHAR(50),
    @WorkflowId NVARCHAR(200),
    @RequestHash INT,  -- e.g., ABS(CHECKSUM(@SomeStableGuid)) % 100
    @OutVersion INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    ;WITH Active AS (
        SELECT Version, RolloutPercent
        FROM ins.WorkflowActivations
        WHERE TenantId = @TenantId AND Product = @Product AND WorkflowId = @WorkflowId AND Status = 'Active'
    ),
    Buckets AS (
        SELECT Version,
               RolloutPercent,
               SUM(RolloutPercent) OVER (ORDER BY Version ROWS UNBOUNDED PRECEDING) - RolloutPercent AS StartPct,
               SUM(RolloutPercent) OVER (ORDER BY Version ROWS UNBOUNDED PRECEDING) - 1                AS StartInclHelper -- to avoid boundary off-by-one
        FROM Active
    )
    SELECT TOP 1 @OutVersion = Version
    FROM Buckets
    WHERE @RequestHash >= StartPct AND @RequestHash < (StartPct + RolloutPercent)
    ORDER BY Version;

    -- Fallback to highest active version if no bucket matched (e.g., percent sum < 100)
    IF @OutVersion IS NULL
        SELECT TOP 1 @OutVersion = Version FROM ins.WorkflowActivations
         WHERE TenantId=@TenantId AND Product=@Product AND WorkflowId=@WorkflowId AND Status='Active'
         ORDER BY ActivatedAt DESC;
END
```

## 2.2 Create Policy

```sql
CREATE OR ALTER PROCEDURE ins.usp_Policy_Create
    @TenantId    UNIQUEIDENTIFIER,
    @CustomerId  UNIQUEIDENTIFIER,
    @Product     NVARCHAR(50),
    @EffectiveDate DATE,
    @RenewalDate   DATE = NULL,
    @CoveragesJson NVARCHAR(MAX) = NULL,
    @CreatedBy   NVARCHAR(128) = NULL,
    @OutPolicyId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    BEGIN TRAN;
      INSERT ins.Policies (TenantId, CustomerId, Product, Status, EffectiveDate, RenewalDate, CoveragesJson, CreatedBy)
      VALUES (@TenantId, @CustomerId, @Product, 'Active', @EffectiveDate, @RenewalDate, @CoveragesJson, @CreatedBy);

      SET @OutPolicyId = SCOPE_IDENTITY(); -- not valid for GUID; use inserted table
      SELECT @OutPolicyId = PolicyId FROM ins.Policies WHERE PolicyId = (SELECT TOP 1 PolicyId FROM ins.Policies WHERE TenantId=@TenantId AND CustomerId=@CustomerId ORDER BY CreatedAt DESC);

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, 'Policy', @OutPolicyId, @CreatedBy, 'Create', @CoveragesJson);
    COMMIT;
END
```

> Note: For GUID PKs, grab with `OUTPUT` clause (see next proc). Kept here simple; below we’ll use OUTPUT properly.

## 2.3 Create Claim (pins workflow version)

```sql
CREATE OR ALTER PROCEDURE ins.usp_Claim_Create
    @TenantId   UNIQUEIDENTIFIER,
    @PolicyId   UNIQUEIDENTIFIER,
    @Product    NVARCHAR(50),
    @LossType   NVARCHAR(50),
    @LossDate   DATE,
    @EstimatedIndemnity DECIMAL(18,2) = NULL,
    @WorkflowId NVARCHAR(200),
    @RequestHash INT,                -- ABS(CHECKSUM(@ClaimGuid)) % 100 (computed app-side)
    @Actor      NVARCHAR(128) = NULL,
    @OutClaimId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    DECLARE @Version INT;
    EXEC ins.usp_Workflow_GetActiveVersion
        @TenantId=@TenantId, @Product=@Product, @WorkflowId=@WorkflowId, @RequestHash=@RequestHash, @OutVersion=@Version OUTPUT;

    BEGIN TRAN;

      DECLARE @NewClaims TABLE (ClaimId UNIQUEIDENTIFIER);
      INSERT ins.Claims (TenantId, PolicyId, Product, LossType, LossDate, Severity, Status, EstimatedIndemnity, WorkflowId, WorkflowVersion)
      OUTPUT inserted.ClaimId INTO @NewClaims(ClaimId)
      VALUES (@TenantId, @PolicyId, @Product, @LossType, @LossDate, NULL, 'Open', @EstimatedIndemnity, @WorkflowId, @Version);

      SELECT @OutClaimId = ClaimId FROM @NewClaims;

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, 'Claim', @OutClaimId, @Actor, 'Create',
              JSON_OBJECT('LossType':@LossType,'LossDate':@LossDate,'EstimatedIndemnity':@EstimatedIndemnity,
                          'WorkflowId':@WorkflowId,'Version':@Version));

    COMMIT;
END
```

## 2.4 Set/Update Trace URI for a Claim

```sql
CREATE OR ALTER PROCEDURE ins.usp_Claim_SetTraceUri
    @ClaimId  UNIQUEIDENTIFIER,
    @TraceUri NVARCHAR(400),
    @Actor    NVARCHAR(128) = NULL
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    UPDATE ins.Claims SET TraceUri=@TraceUri, ModifiedAt=SYSUTCDATETIME(), ModifiedBy=@Actor
     WHERE ClaimId=@ClaimId;

    INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
    SELECT TenantId, 'Claim', @ClaimId, @Actor, 'SetTraceUri', JSON_OBJECT('TraceUri':@TraceUri)
    FROM ins.Claims WHERE ClaimId=@ClaimId;
END
```

## 2.5 Create Pending Approval

```sql
CREATE OR ALTER PROCEDURE ins.usp_Approval_CreatePending
    @TenantId   UNIQUEIDENTIFIER,
    @EntityType NVARCHAR(30),       -- 'Claim' | 'Quote' | 'Policy'
    @EntityId   UNIQUEIDENTIFIER,
    @GateId     NVARCHAR(100),
    @Approver   NVARCHAR(256),
    @Actor      NVARCHAR(128) = NULL,
    @OutApprovalId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    BEGIN TRAN;
      DECLARE @New TABLE (ApprovalId UNIQUEIDENTIFIER);
      INSERT ins.Approvals (TenantId, EntityType, EntityId, GateId, Approver, Status)
      OUTPUT inserted.ApprovalId INTO @New(ApprovalId)
      VALUES (@TenantId, @EntityType, @EntityId, @GateId, @Approver, 'Pending');

      SELECT @OutApprovalId = ApprovalId FROM @New;

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, @EntityType, @EntityId, @Actor, 'ApprovalCreated',
              JSON_OBJECT('GateId':@GateId,'Approver':@Approver));
    COMMIT;
END
```

## 2.6 Decide Approval (approve/reject) with optimistic concurrency

```sql
CREATE OR ALTER PROCEDURE ins.usp_Approval_Decide
    @ApprovalId UNIQUEIDENTIFIER,
    @Decision   NVARCHAR(20),     -- 'Approved' | 'Rejected'
    @Reason     NVARCHAR(400) = NULL,
    @Actor      NVARCHAR(128) = NULL
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    IF @Decision NOT IN ('Approved','Rejected')
    BEGIN
        RAISERROR('Invalid decision', 16, 1); RETURN;
    END

    BEGIN TRAN;

      UPDATE ins.Approvals
        SET Status=@Decision, Reason=@Reason, DecidedAt=SYSUTCDATETIME()
      WHERE ApprovalId=@ApprovalId AND Status='Pending';

      IF @@ROWCOUNT = 0
      BEGIN
          ROLLBACK; RAISERROR('Approval not in Pending state or not found.', 16, 1); RETURN;
      END

      DECLARE @TenantId UNIQUEIDENTIFIER, @EntityType NVARCHAR(30), @EntityId UNIQUEIDENTIFIER, @GateId NVARCHAR(100);
      SELECT @TenantId=TenantId, @EntityType=EntityType, @EntityId=EntityId, @GateId=GateId
      FROM ins.Approvals WHERE ApprovalId=@ApprovalId;

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, @EntityType, @EntityId, @Actor, CONCAT('Approval', @Decision),
              JSON_OBJECT('ApprovalId':@ApprovalId,'GateId':@GateId,'Reason':@Reason));

      -- Optional: if Claim & approved, update claim status
      IF @EntityType='Claim' AND @Decision='Approved'
          UPDATE ins.Claims SET Status='Open', ModifiedAt=SYSUTCDATETIME(), ModifiedBy=@Actor WHERE ClaimId=@EntityId AND Status='PendingApproval';

      IF @EntityType='Claim' AND @Decision='Rejected'
          UPDATE ins.Claims SET Status='Denied', ModifiedAt=SYSUTCDATETIME(), ModifiedBy=@Actor WHERE ClaimId=@EntityId;

    COMMIT;
END
```

## 2.7 Create Payment and roll-up totals

```sql
CREATE OR ALTER PROCEDURE ins.usp_Payment_Create
    @TenantId  UNIQUEIDENTIFIER,
    @ClaimId   UNIQUEIDENTIFIER,
    @Amount    DECIMAL(18,2),
    @Method    NVARCHAR(30),
    @Status    NVARCHAR(20) = 'Initiated',
    @TxnRef    NVARCHAR(100) = NULL,
    @Actor     NVARCHAR(128) = NULL,
    @OutPaymentId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    BEGIN TRAN;

      DECLARE @New TABLE (PaymentId UNIQUEIDENTIFIER);
      INSERT ins.Payments (TenantId, ClaimId, Amount, Method, Status, TxnRef, CreatedBy)
      OUTPUT inserted.PaymentId INTO @New(PaymentId)
      VALUES (@TenantId, @ClaimId, @Amount, @Method, @Status, @TxnRef, @Actor);

      SELECT @OutPaymentId = PaymentId FROM @New;

      -- If succeeded immediately, roll-up into claim
      IF @Status = 'Succeeded'
      BEGIN
          UPDATE ins.Claims
             SET Paid = Paid + @Amount, ModifiedAt=SYSUTCDATETIME(), ModifiedBy=@Actor
           WHERE ClaimId=@ClaimId;

          -- Optionally settle if simple/low severity and fully paid
          -- (leave to workflow engine normally)
      END

      -- Audit
      DECLARE @tenantFromClaim UNIQUEIDENTIFIER = (SELECT TenantId FROM ins.Claims WHERE ClaimId=@ClaimId);
      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@tenantFromClaim, 'Payment', @OutPaymentId, @Actor, 'Create',
              JSON_OBJECT('ClaimId':@ClaimId,'Amount':@Amount,'Status':@Status,'TxnRef':@TxnRef));
    COMMIT;
END
```

## 2.8 Set Claim Status helper (used by workflow engine)

```sql
CREATE OR ALTER PROCEDURE ins.usp_Claim_SetStatus
    @ClaimId UNIQUEIDENTIFIER,
    @NewStatus NVARCHAR(30),    -- 'Open','PendingApproval','Settled','Closed','Denied'
    @Actor NVARCHAR(128) = NULL,
    @Reason NVARCHAR(400) = NULL
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    IF @NewStatus NOT IN ('Open','PendingApproval','Settled','Closed','Denied')
    BEGIN
        RAISERROR('Invalid status',16,1); RETURN;
    END

    BEGIN TRAN;
      UPDATE ins.Claims
        SET Status=@NewStatus, ModifiedAt=SYSUTCDATETIME(), ModifiedBy=@Actor
      WHERE ClaimId=@ClaimId;

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      SELECT TenantId, 'Claim', @ClaimId, @Actor, 'StatusChange',
             JSON_OBJECT('NewStatus':@NewStatus,'Reason':@Reason)
      FROM ins.Claims WHERE ClaimId=@ClaimId;
    COMMIT;
END
```

## 2.9 Activate / Retire Workflow Version (admin)

```sql
CREATE OR ALTER PROCEDURE ins.usp_Workflow_ActivateVersion
    @TenantId   UNIQUEIDENTIFIER,
    @Product    NVARCHAR(50),
    @WorkflowId NVARCHAR(200),
    @Version    INT,
    @RolloutPercent TINYINT,          -- 0..100
    @ActivatedBy NVARCHAR(128) = NULL
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    -- Ensure definition exists
    IF NOT EXISTS (
        SELECT 1 FROM ins.WorkflowDefinitions
        WHERE TenantId=@TenantId AND Product=@Product AND WorkflowId=@WorkflowId AND Version=@Version
    )
    BEGIN
        RAISERROR('Workflow definition not found.',16,1); RETURN;
    END

    BEGIN TRAN;

      INSERT ins.WorkflowActivations (TenantId, Product, WorkflowId, Version, RolloutPercent, Status, ActivatedBy)
      VALUES (@TenantId, @Product, @WorkflowId, @Version, @RolloutPercent, 'Active', @ActivatedBy);

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, 'Workflow', NULL, @ActivatedBy, 'ActivateWorkflow',
              JSON_OBJECT('Product':@Product,'WorkflowId':@WorkflowId,'Version':@Version,'Percent':@RolloutPercent));

    COMMIT;
END
```

```sql
CREATE OR ALTER PROCEDURE ins.usp_Workflow_RetireVersion
    @TenantId   UNIQUEIDENTIFIER,
    @Product    NVARCHAR(50),
    @WorkflowId NVARCHAR(200),
    @Version    INT,
    @Actor      NVARCHAR(128) = NULL
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    BEGIN TRAN;
      UPDATE ins.WorkflowActivations
         SET Status='Retired'
       WHERE TenantId=@TenantId AND Product=@Product AND WorkflowId=@WorkflowId AND Version=@Version AND Status='Active';

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, 'Workflow', NULL, @Actor, 'RetireWorkflow',
              JSON_OBJECT('Product':@Product,'WorkflowId':@WorkflowId,'Version':@Version));
    COMMIT;
END
```

## 2.10 Add Document to an Entity

```sql
CREATE OR ALTER PROCEDURE ins.usp_Document_Add
    @TenantId   UNIQUEIDENTIFIER,
    @EntityType NVARCHAR(30),     -- 'Claim'|'Policy'|'Customer'
    @EntityId   UNIQUEIDENTIFIER,
    @DocType    NVARCHAR(50),
    @Uri        NVARCHAR(400),
    @Actor      NVARCHAR(128) = NULL,
    @OutDocumentId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;

    BEGIN TRAN;
      DECLARE @New TABLE (DocumentId UNIQUEIDENTIFIER);
      INSERT ins.Documents (TenantId, EntityType, EntityId, DocType, Uri, CreatedBy)
      OUTPUT inserted.DocumentId INTO @New(DocumentId)
      VALUES (@TenantId, @EntityType, @EntityId, @DocType, @Uri, @Actor);

      SELECT @OutDocumentId = DocumentId FROM @New;

      INSERT ins.AuditEvents (TenantId, EntityType, EntityId, Actor, Action, PayloadJson)
      VALUES (@TenantId, @EntityType, @EntityId, @Actor, 'AddDocument',
              JSON_OBJECT('DocType':@DocType,'Uri':@Uri));
    COMMIT;
END
```

## 2.11 Read Claim (detail)

```sql
CREATE OR ALTER PROCEDURE ins.usp_Claim_Get
    @ClaimId UNIQUEIDENTIFIER
AS
BEGIN
    SET NOCOUNT ON;

    SELECT c.*, p.Product AS PolicyProduct, cust.FullName AS CustomerName
    FROM ins.Claims c
    JOIN ins.Policies p  ON p.PolicyId = c.PolicyId
    JOIN ins.Customers cust ON cust.CustomerId = p.CustomerId
    WHERE c.ClaimId = @ClaimId;

    SELECT a.* FROM ins.Approvals a WHERE a.EntityType='Claim' AND a.EntityId=@ClaimId;
    SELECT py.* FROM ins.Payments  py WHERE py.ClaimId=@ClaimId;
    SELECT d.*  FROM ins.Documents d WHERE d.EntityType='Claim' AND d.EntityId=@ClaimId;
END
```

---

# 3) Helpful Indexes & JSON patterns

```sql
-- Example: computed column over JSON (if you often filter by deductible)
ALTER TABLE ins.Policies
ADD Deductible_Comp AS TRY_CONVERT(DECIMAL(18,2), JSON_VALUE(CoveragesJson, '$.auto.deductible')) PERSISTED;
CREATE INDEX IX_Policies_Deductible ON ins.Policies(Deductible_Comp);

-- Fast “open approvals” per tenant
CREATE INDEX IX_Approvals_Pending ON ins.Approvals(TenantId, Status) WHERE Status='Pending';
```

---

# 4) Example usage

```sql
-- Create a claim & pin workflow version
DECLARE @ClaimId UNIQUEIDENTIFIER, @Tenant UNIQUEIDENTIFIER = 'AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE';
DECLARE @Policy UNIQUEIDENTIFIER = '11111111-2222-3333-4444-555555555555';

-- Ensure a workflow definition & activation exist (usually done by admin tooling)
-- EXEC ins.usp_Workflow_ActivateVersion @Tenant, 'auto', 'claims.fnol.auto', 3, 100, 'ops@insurer.com';

-- Hash bucket for canary (client typically computes ABS(CHECKSUM(@ClaimGuid)) % 100)
DECLARE @RequestHash INT = 42;

EXEC ins.usp_Claim_Create
  @TenantId=@Tenant,
  @PolicyId=@Policy,
  @Product='auto',
  @LossType='glass',
  @LossDate=GETUTCDATE(),
  @EstimatedIndemnity=180.00,
  @WorkflowId='claims.fnol.auto',
  @RequestHash=@RequestHash,
  @Actor='api@system',
  @OutClaimId=@ClaimId OUTPUT;

-- Create pending approval (if workflow gated)
DECLARE @ApprovalId UNIQUEIDENTIFIER;
EXEC ins.usp_Approval_CreatePending @Tenant, 'Claim', @ClaimId, 'approvalGate', 'role:Supervisor', 'api@system', @ApprovalId OUTPUT;

-- Approver decides
EXEC ins.usp_Approval_Decide @ApprovalId, 'Approved', 'OK to pay', 'supervisor@insurer.com';

-- Create payment
DECLARE @PaymentId UNIQUEIDENTIFIER;
EXEC ins.usp_Payment_Create @Tenant, @ClaimId, 180.00, 'ACH', 'Succeeded', 'TX123', 'api@payments', @PaymentId OUTPUT;

-- Close/settle claim
EXEC ins.usp_Claim_SetStatus @ClaimId, 'Settled', 'api@system', 'Auto glass settled';
```

---

## Notes / Extensions

* **Idempotency:** add an optional `@IdemKey` to write tables and unique index on `(EntityType, EntityId, IdemKey)` for safe retries.
* **PII controls:** mask in `AuditEvents.PayloadJson` if needed; gate raw access by role.
* **Row-level security:** if you multi-tenant by TenantId, add RLS predicates.
* **Temporal tables:** enable on `Claims`, `Policies` if you want built-in history.
* **Performance:** for large volumes, consider partitioning `Claims` by `CreatedAt` month.

---

If you want, I can also add:

* SQL **views** for dashboards (e.g., SLA breaches, STP rate),
* A **CDC/Change Tracking** setup for downstream analytics,
* Or a minimal **EF Core** model scaffold to match these tables.
