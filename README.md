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

  ```
