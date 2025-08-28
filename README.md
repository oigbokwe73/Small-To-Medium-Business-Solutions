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
