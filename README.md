# Small-To-Medium-Business-Solutions


\here you go — a clean end-to-end sequence showing **AI-generated UI → user fills form → REST API runs JSON workflow**.

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant AG as AI UI Agent
  participant UR as UI Renderer Browser/App
  participant APIM as API Gateway / Auth
  participant WF as Workflow API Functions
  participant WS as Workflow Store JSON
  participant DB as SQL Requests/Approvals
  participant TR as Trace Store JSONL
  participant SCH as Scheduler Svc
  participant NT as Notification Svc

  %% --- UI generation ---
  U->>AG: "Start new request" intent: create_request
  AG->>WS: GET /workflows/{id}/active rules + uiModel
  WS-->>AG: workflow JSON + uiModel
  AG->>UR: Emit component spec fields, showIf, validations
  UR-->>U: Auto-generated intake form displayed

  %% --- Form completion & validation ---
  U->>UR: Fill fields, attach files
  UR->>UR: Client-side validation from uiModel/schema
  UR->>APIM: POST /requests {payload}

  %% --- API entry & workflow pinning ---
  APIM->>WF: Forward request JWT, tenanth
  WF->>WS: Load active workflow pin workflowId@version
  WF->>DB: INSERT Request status=Created, versionRef
  WF->>TR: Append trace "created", input snapshot

 %% --- Workflow evaluation ---
  WF->>WF: Evaluate steps (switch/compute/route)
  WF->>TR: Append trace("classify","price","route")

alt Approval gate required?
    WF->>DB: INSERT PendingApproval (gateId, approvers)
    WF->>NT: Notify approvers (email/Teams)
    WF-->>UR: 202 Accepted (awaiting approval, SLA shown)
    Note over U,UR: UI shows "Awaiting approval" + SLA clock
    %% Approver action (out-of-band)
    NT-->>U: Approval deep link (for approver persona)
    U->>APIM: POST /requests/{id}/actions/approve|reject
    APIM->>WF: Resume gate (decision captured)
    WF->>DB: UPDATE Approval + Request status
    WF->>TR: Append trace("approval", decision)
  else No approval needed
    Note over WF: Gate skipped continue execution
  end

%% --- Scheduling / external calls ---
  WF->>SCH: POST /schedule/suggest {team, skills, sla}
  SCH-->>WF: Candidate slots
  WF->>SCH: POST /schedule/book {slot}
  SCH-->>WF: eventId
  WF->>TR: Append trace("schedule", eventId)

  %% --- Notifications & response ---
  WF->>NT: Send confirmations (customer/agent)
  WF->>DB: UPDATE Request (status=Scheduled/InProgress)
  WF-->>APIM: 201 Created (id, priority, SLA, schedule)
  APIM-->>UR: Response
  UR-->>U: Show confirmation (tracking, SLA timer)

  %% --- Closure (later) ---
  U->>APIM: POST /requests/{id}/close {resolution}
  APIM->>WF: Close request
  WF->>DB: UPDATE Request (Closed)
  WF->>TR: Append trace("closed")
  WF->>NT: Send survey/invoice

  ```

Got it—here’s a step-by-step table that pairs with the sequence diagram (AI-generated UI → user fills form → REST API executes JSON workflow). I grouped steps by phase and kept fields you’ll likely need in implementation.

| #   | Phase          | Actor → Target                  | Action / Endpoint                                | Key Inputs                             | Writes / Updates                      | Success Output                       | Failure / Notes                                |                      |
| --- | -------------- | ------------------------------- | ------------------------------------------------ | -------------------------------------- | ------------------------------------- | ------------------------------------ | ---------------------------------------------- | -------------------- |
| 1   | UI generation  | User → AI UI Agent              | Start intent `create_request`                    | Tenant, user profile, locale           | —                                     | Agent session started                | Could come from portal “New request” button    |                      |
| 2   | UI generation  | AI UI Agent → Workflow Store    | **GET** `/workflows/{id}/active`                 | workflowId, tenant                     | —                                     | JSON: `{ rules, uiModel, version }`  | Pin consistent version for the session         |                      |
| 3   | UI generation  | Workflow Store → Agent          | Return active workflow + `uiModel`               | —                                      | —                                     | Same as above                        | Consider ETag for caching                      |                      |
| 4   | UI generation  | Agent → UI Renderer             | Emit component spec (fields, showIf, validation) | `uiModel`                              | —                                     | Render spec (React tree / form meta) | Guardrails: only allowed field types           |                      |
| 5   | UI generation  | UI Renderer → User              | Display auto-generated intake form               | Spec from step 4                       | —                                     | Form visible                         | Prefill known values (user, BU, tier)          |                      |
| 6   | Form fill      | User → UI Renderer              | Enter values, attach files                       | Field values, files                    | Local form state                      | —                                    | Client hints for required/format               |                      |
| 7   | Form validate  | UI Renderer (client)            | Validate per schema                              | `uiModel.validation`                   | —                                     | Validated payload                    | Show inline errors; disable submit until valid |                      |
| 8   | Submit         | UI Renderer → API Gateway       | **POST** `/requests`                             | JSON payload, JWT, **Idempotency-Key** | —                                     | 2xx/202/201 passthrough              | Retry-safe with Idempotency-Key                |                      |
| 9   | AuthN/Z        | API Gateway → Workflow API      | Forward request                                  | JWT (tenant, roles), payload           | —                                     | Authorized call                      | Enforce scopes, rate limits                    |                      |
| 10  | Pin version    | Workflow API → Workflow Store   | Load active workflow                             | workflowId, tenant                     | —                                     | `{ rules, version }`                 | Pin `workflowId@version` on request            |                      |
| 11  | Persist create | Workflow API → SQL DB           | Insert `Requests` row                            | payload, versionRef                    | `Requests(...)`                       | Row created (id)                     | Use transactions / retries                     |                      |
| 12  | Audit          | Workflow API → Trace Store      | Append `"created"` snapshot                      | normalized payload, versionRef         | `DecisionTrace/{id}.jsonl`            | Trace line written                   | Mask PII by policy                             |                      |
| 13  | Evaluate       | Workflow API (engine)           | Step: **classify (switch)**                      | impact, tier, category                 | Update in-mem context                 | priority, `slaHours`                 | Validate expressions                           |                      |
| 14  | Evaluate       | Workflow API (engine)           | Step: **price (compute)**                        | items\[], tier                         | —                                     | `quote.totalBeforeTax`               | Use safe expression sandbox                    |                      |
| 15  | Evaluate       | Workflow API (engine)           | Step: **route**                                  | category, region                       | —                                     | team/queue                           | Default route if no match                      |                      |
| 16  | Audit          | Workflow API → Trace Store      | Append step traces                               | classify/price/route results           | DecisionTrace                         | Trace lines written                  | Correlate with step ids                        |                      |
| 17  | Gate?          | Workflow API (engine)           | Check **gate (approval)**                        | rule: e.g., price > 1000               | —                                     | Boolean                              | Branch below                                   |                      |
| 18A | Approval path  | Workflow API → SQL DB           | Insert `Approvals` pending                       | approvers, gateId                      | `Approvals(...)`                      | Pending approval row                 | Prevent self-approval                          |                      |
| 19A | Approval path  | Workflow API → Notif Svc        | Notify approvers                                 | request summary, deep link             | —                                     | Notification sent                    | Email/Teams/Slack per policy                   |                      |
| 20A | Approval path  | Workflow API → API Gateway → UI | **202 Accepted** “awaiting approval”             | request id, SLA, approvers             | —                                     | API 202 → UI message                 | UI shows SLA clock & status                    |                      |
| 21A | Approval path  | Approver → API Gateway          | **POST** \`/requests/{id}/actions/approve        | reject\`                               | decision, comment                     | —                                    | 200 OK                                         | Deep link from notif |
| 22A | Approval path  | API Gateway → Workflow API      | Resume gate                                      | decision payload                       | Update `Approvals`, `Requests.status` | Request unblocked                    | Log actor/time/reason                          |                      |
| 23  | Scheduling     | Workflow API → Scheduler Svc    | **POST** `/schedule/suggest`                     | team, skills, SLA window               | —                                     | Candidate slots                      | Timeout/retry with backoff                     |                      |
| 24  | Scheduling     | Workflow API → Scheduler Svc    | **POST** `/schedule/book`                        | chosen slot                            | —                                     | `{ eventId, start }`                 | Ensure idempotent booking                      |                      |
| 25  | Audit          | Workflow API → Trace Store      | Append `"schedule"` trace                        | eventId, slot                          | DecisionTrace                         | Trace line written                   | —                                              |                      |
| 26  | Persist status | Workflow API → SQL DB           | Update request status                            | `Scheduled`/`InProgress`               | `Requests.status`                     | Row updated                          | Concurrency checks                             |                      |
| 27  | Notify         | Workflow API → Notif Svc        | Send confirmations                               | customer/agent payloads                | —                                     | Messages sent                        | Include calendar ICS if relevant               |                      |
| 28  | Respond        | Workflow API → API Gateway → UI | **201 Created** summary                          | id, priority, SLA, schedule            | —                                     | API 201 → UI confirmation            | Include versionRef in meta                     |                      |
| 29  | Present        | UI Renderer → User              | Show confirmation                                | response payload                       | —                                     | Tracking view (SLA timer)            | Offer “Add details” / attachments              |                      |
| 30  | Close (later)  | Agent/User → API Gateway        | **POST** `/requests/{id}/close`                  | resolution, timeSpent                  | —                                     | 200 OK                               | Role-gated                                     |                      |
| 31  | Close (later)  | API Gateway → Workflow API      | Close workflow                                   | payload                                | Update `Requests.closedAt/status`     | Closed                               | —                                              |                      |
| 32  | Audit          | Workflow API → Trace Store      | Append `"closed"` trace                          | resolution summary                     | DecisionTrace                         | Trace line written                   | Retention policy applies                       |                      |
| 33  | Post-ops       | Workflow API → Notif/Billing    | Survey / invoice                                 | request summary                        | External systems                      | Sent                                 | Optional integrations                          |                      |

### Notes & implementation tips

* **Schema-first:** Derive both client-side and server-side validation from the same `uiModel` JSON (one source of truth).
* **Idempotency:** Use `Idempotency-Key` on all mutating POSTs to guard against retries/timeouts.
* **Pinning:** Persist `workflowId@version` on the request so replays/audits are reproducible.
* **Observability:** Correlate all steps with a `correlationId` spanning UI → API → engine → plugins.
* **Security:** Enforce roles (`Customer`, `Agent`, `Approver`, `WorkflowAdmin`, `Auditor`) at the gateway; block self-approval in the engine.
* **Canary:** If you roll out new workflow versions, hash request id to decide which version runs; store the result in `Requests.workflowVersionRef`.

