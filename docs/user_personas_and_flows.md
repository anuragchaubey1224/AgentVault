# AgentVault: User Personas and Flows


**Related document:** [PRD](./prd.md)

## 1. Product Actors

| Persona | Primary responsibility | Main needs |
|---|---|---|
| **Developer** | Builds agents and registers tools | Clear contracts, easy integration, predictable failures |
| **Platform Admin** | Manages organization, agents, tools, and access | Tenant isolation, roles, credential control |
| **Security Engineer** | Defines policies and investigates risks | Deterministic enforcement, audit trail, security events |
| **Approval Reviewer** | Reviews sensitive actions | Exact action details, safe approve/reject experience |
| **Agent User** | Runs project-assistant tasks | Useful results, execution visibility, safe automation |
| **Evaluator** | Compares baseline and protected runs | Repeatable scenarios, trustworthy metrics |

> In the MVP, one person may perform multiple roles. Roles are separated here to keep responsibilities clear.

## 2. Role Relationship

```mermaid
flowchart LR
    U[Agent User] --> A[Agentic Assistant]
    D[Developer] --> T[Tool Registry]
    PA[Platform Admin] --> I[Identity & Organization]
    PA --> T
    S[Security Engineer] --> P[Policy Engine]
    R[Approval Reviewer] --> H[Approval Service]
    E[Evaluator] --> X[Evaluation Dashboard]

    A --> G[AgentVault Gateway]
    T --> G
    I --> G
    P --> G
    H --> G
    G --> EX[External Tools]
```

## 3. Core User Journey

```mermaid
flowchart TD
    S((Start)) --> R[Register / Sign in]
    R --> O[Create or join organization]
    O --> C[Connect GitHub and email]
    C --> AG[Register project assistant]
    AG --> P[Configure tools and policies]
    P --> M{Choose execution mode}
    M --> B[Baseline run]
    M --> V[Protected run]
    B --> H[View execution history]
    V --> H
    H --> E[Compare results]
    E --> F((Finish))
```

## 4. Onboarding Flow

```mermaid
flowchart TD
    S[User signs in] --> O{Organization exists?}
    O -- No --> CO[Create organization]
    O -- Yes --> J[Select organization]
    CO --> R[Assign role]
    J --> R
    R --> I[Connect integrations]
    I --> G{Connection valid?}
    G -- No --> ER[Show error and retry]
    ER --> I
    G -- Yes --> A[Register agent]
    A --> T[Select tools and resource scope]
    T --> D[Ready for first workflow]
```

### Onboarding outcomes

- The user belongs to an organization.
- Integrations use minimum required scopes.
- The agent has a unique identity and limited permissions.
- No raw external credential is shown to the agent or stored in normal application data.

## 5. Agent Registration and Configuration

```mermaid
flowchart LR
    U[Authorized user] --> F[Agent name and description]
    F --> T[Select registered tools]
    T --> R[Set resource scope]
    R --> P[Attach policies]
    P --> C[Create scoped agent credential]
    C --> Q{Validation passed?}
    Q -- No --> E[Show validation errors]
    Q -- Yes --> A[Agent becomes active]
```

## 6. Primary Workflow: Project Status Update

### 6.1 Shared agent workflow

```mermaid
flowchart TD
    U[User request] --> N[Agent understands and plans]
    N --> GH[Read authorized GitHub issues / PRs]
    GH --> EM[Search and read relevant emails]
    EM --> SU[Generate project status summary]
    SU --> DR[Prepare email draft]
    DR --> M{Send email?}
    M -- No --> OUT[Return summary and draft]
    M -- Yes --> MODE{Execution mode}
    MODE --> BASE[Baseline integration]
    MODE --> PROT[Protected gateway]
    BASE --> SEND[Send using restricted test account]
    PROT --> AUTH[Policy and approval flow]
    SEND --> OUT2[Return result]
    AUTH --> OUT2
```

### 6.2 Baseline mode

```mermaid
sequenceDiagram
    participant U as User
    participant A as Agent
    participant T as External Tool

    U->>A: Submit project-status request
    A->>T: Request tool operation
    T-->>A: Tool result
    A-->>U: Summary, draft, and execution status
```

Baseline mode is used for comparison and must still use restricted accounts, safe scopes, and execution logging.

### 6.3 Protected mode

```mermaid
sequenceDiagram
    participant U as User
    participant A as Agent
    participant G as AgentVault Gateway
    participant P as Policy Engine
    participant H as Reviewer
    participant T as External Tool

    U->>A: Submit project-status request
    A->>G: Tool request + execution context
    G->>G: Authenticate agent and validate schema
    G->>P: Evaluate user, agent, resource, and action
    P-->>G: ALLOW / DENY / REQUIRE_APPROVAL

    alt ALLOW
        G->>T: Execute with protected credential
        T-->>G: Result
        G-->>A: Validated and redacted result
    else DENY
        G-->>A: Structured denial
    else REQUIRE_APPROVAL
        G->>H: Create exact-action approval request
        H-->>G: Approve or reject
        G->>P: Revalidate exact approved action
        G->>T: Execute only if still valid
        T-->>G: Result
        G-->>A: Result or explicit failure
    end

    A-->>U: Final response and execution status
```

## 7. Approval Flow

```mermaid
stateDiagram-v2
    [*] --> PENDING: Sensitive action detected
    PENDING --> APPROVED: Reviewer approves exact action
    PENDING --> REJECTED: Reviewer rejects
    PENDING --> EXPIRED: Approval timeout
    PENDING --> CANCELLED: Execution cancelled
    APPROVED --> REVALIDATING: Before execution
    REVALIDATING --> EXECUTED: Policy + action hash valid
    REVALIDATING --> REJECTED: Arguments/resource changed
    EXECUTED --> [*]
    REJECTED --> [*]
    EXPIRED --> [*]
    CANCELLED --> [*]
```

### Reviewer must see

- Requesting user and agent.
- Tool and operation.
- Resource and recipients.
- Exact email subject, body, and attachments where applicable.
- Policy decision and risk explanation.
- Expiration time.

Approval authorizes only the exact reviewed action. Any changed argument requires a new decision.

## 8. Policy Decision Flow

```mermaid
flowchart TD
    R[Tool request] --> V[Validate tool and arguments]
    V --> C[Build trusted execution context]
    C --> P[Evaluate deterministic policies]
    P --> D{Decision}
    D -- ALLOW --> X[Execute]
    D -- DENY --> N[Block and audit]
    D -- REQUIRE_APPROVAL --> H[Create approval request]
    H --> A{Approved?}
    A -- No --> N2[Reject / expire / cancel and audit]
    A -- Yes --> RV[Revalidate exact action]
    RV --> X
    X --> L[Audit result]
```

**Rule:** AI risk analysis can provide warnings, but it cannot grant permission or override a deterministic deny.

## 9. Audit and Monitoring Flow

```mermaid
flowchart LR
    G[Gateway events] --> AU[Audit event store]
    P[Policy decisions] --> AU
    H[Approval events] --> AU
    T[Tool outcomes] --> AU
    AU --> Q[Search and filters]
    AU --> SE[Security events]
    AU --> M[Metrics]
    Q --> D[Execution dashboard]
    SE --> D
    M --> CD[Comparison dashboard]
```

Every protected request should be traceable using an execution ID and correlation ID.

## 10. Evaluation Flow

```mermaid
flowchart TD
    S[Select security scenario] --> I[Prepare same input and test identity]
    I --> B[Run baseline]
    I --> P[Run protected]
    B --> RB[Store baseline outcome]
    P --> RP[Store protected outcome]
    RB --> C[Calculate metrics]
    RP --> C
    C --> V[Compare security, latency, and completion]
    V --> R[Review limitations and findings]
```

### Initial scenarios

- Unauthorized repository access.
- Unauthorized email sending.
- Prompt injection in email content.
- Cross-organization access.
- Modified arguments after approval.
- Duplicate side effect after retry.

## 11. Important UX Rules

1. Always show whether the run is **Baseline** or **Protected**.
2. Show tool-call progress without exposing secrets.
3. Show clear `allowed`, `denied`, `approval required`, `failed`, and `completed` states.
4. Never present a denied or failed action as successful.
5. Make approval details exact and reviewable.
6. Keep audit and evaluation data tenant-scoped.
7. Explain policy decisions in understandable language.

## 12. MVP Flow Coverage

| Flow | MVP status |
|---|---|
| Sign in and organization setup | Required |
| Connect GitHub and email | Required |
| Register agent and tools | Required |
| Configure basic policies | Required |
| Run baseline workflow | Required |
| Run protected workflow | Required |
| Approve or reject email send | Required |
| Search execution history | Required |
| Compare baseline and protected results | Required |
| Advanced anomaly detection | Post-MVP |
| Universal MCP interception | Post-MVP |

