# AgentVault Product Requirements Document

**Status:** Draft for review  
**Version:** 1.0  
**Date:** 2026-11-18  
**Product:** AgentVault  

## 1. Executive Summary

AgentVault is a developer-oriented security and governance platform for agentic AI applications. It enables an AI agent to use external tools while giving organizations a centralized way to authenticate agents, authorize actions, enforce policies, require human approval for sensitive operations, protect credentials, and audit execution.

The product will include:

1. A working multi-tool project assistant that can perform useful, multi-step tasks.
2. A security control plane that governs the assistant's tool calls.
3. A controlled comparison between the same workflow running without and with AgentVault protection.

AgentVault is not intended to be only a chatbot, a basic retrieval application, or an SDK. Its primary value is controlled execution: an agent may reason freely, but every protected external action must cross a deterministic security boundary before execution.

## 2. Vision

Make autonomous AI actions observable, authorized, and accountable by default.

AgentVault should demonstrate that agent capability and execution safety can coexist. Developers should be able to build useful agents without handing unrestricted credentials and operational authority to an LLM.

## 3. Problem Statement

AI agents can plan and execute multi-step workflows across APIs, repositories, email systems, databases, and developer tools. Tool access makes agents useful, but it also creates risks:

- The agent may access a resource outside the user's authorization.
- A prompt injection may influence the agent to perform an unintended action.
- A destructive operation may be selected or repeated accidentally.
- Sensitive data may be sent to an unauthorized destination.
- Service credentials may be exposed to the model or stored insecurely.
- Retries may duplicate an external side effect.
- Different users or organizations may be insufficiently isolated.
- Operators may be unable to reconstruct what happened and why.

Most agent frameworks optimize reasoning and tool selection. They do not, by themselves, provide a complete authorization, approval, credential, audit, and evaluation layer.

AgentVault addresses this gap by placing a centralized execution-control layer between the agent and protected tools.

## 4. Goals

### 4.1 Product Goals

- Provide a usable multi-step agentic project assistant.
- Support baseline and protected execution modes using equivalent workflows.
- Enforce deterministic authorization for every protected tool call.
- Support `ALLOW`, `DENY`, and `REQUIRE_APPROVAL` policy decisions.
- Require approval for sensitive operations such as sending email.
- Prevent raw service credentials from reaching the LLM.
- Record sufficient audit data to reconstruct each execution.
- Measure security, reliability, latency, and task-completion differences using real experiments.
- Establish an architecture that can later be reused by other agent applications.

### 4.2 Learning and Engineering Goals

The project should demonstrate practical experience with:

- Agent orchestration and tool calling.
- API and OAuth integrations.
- Secure gateway design.
- RBAC and resource-level authorization.
- Policy evaluation.
- Human-in-the-loop workflows.
- Multi-tenant data isolation.
- Audit logging and observability.
- Security testing and prompt-injection evaluation.
- Deployment and production operations.

### 4.3 Non-Goals for the Initial Release

The initial release will not attempt to:

- Support unrestricted autonomous access to arbitrary services.
- Expose unrestricted SQL execution to an agent.
- Guarantee that an LLM cannot be prompt-injected.
- Replace a dedicated enterprise IAM, SIEM, or secrets-management product.
- Provide full cloud-infrastructure control.
- Support large-scale multi-agent orchestration.
- Become a universal proxy for every MCP server on day one.
- Introduce microservices before modular boundaries are proven.

## 5. Target Users and Personas

### 5.1 Developer

Builds and integrates AI agents with tools. Needs a clear tool contract, predictable authorization behavior, useful SDK/API boundaries, and local development support.

### 5.2 Security Engineer

Defines policies, reviews risky actions, investigates suspicious behavior, and verifies that enforcement cannot be bypassed.

### 5.3 Platform or Engineering Lead

Wants to provide agent capabilities without distributing unrestricted credentials. Needs execution visibility, organization isolation, and measurable security controls.

### 5.4 Authorized Operator

Reviews approval requests for sensitive actions. Needs to see exactly what will happen, to which resource, with which arguments, and why approval is required.

### 5.5 Evaluator or Researcher

Runs repeatable baseline-versus-protected scenarios and compares security and operational metrics without fabricated results.

## 6. Primary Use Case

The initial application is a project assistant. A user asks:

> Review my GitHub project, check recent project-related emails, summarize pending work, and send an update to my team.

The agent should:

1. Understand the request.
2. Retrieve authorized GitHub issues or pull-request information.
3. Search and read relevant emails.
4. Produce a project-status summary.
5. Prepare an email draft.
6. Request approval before sending the email when policy requires it.
7. Send only the exact approved message to the approved recipients.
8. Return a clear result and execution history.

The same scenario must run in both execution modes.

## 7. Execution Modes

### 7.1 Baseline Mode

Baseline mode runs the agent with its normal configured tool integrations. It is an experimental comparison baseline, not an intentionally unsafe production deployment.

Requirements:

- Use restricted test accounts and minimum necessary scopes.
- Never print or place raw credentials in prompts or logs.
- Record execution history and outcomes.
- Clearly label the mode as baseline in the UI and stored records.

Flow:

```text
User -> Agent -> Tool Selection -> External Tool -> Tool Result -> Agent -> User
```

### 7.2 Protected Mode

Protected mode routes every supported external tool call through AgentVault.

Flow:

```text
User
  -> Agent
  -> AgentVault Gateway
  -> Agent Authentication
  -> Schema Validation
  -> Policy Evaluation
  -> Risk Analysis (advisory)
  -> Approval, if required
  -> Credential Retrieval
  -> Secure Tool Execution
  -> Result Validation and Redaction
  -> Agent
  -> User
```

Protected mode must:

- Verify the calling agent and user context.
- Resolve organization and resource scope.
- Validate the tool and arguments.
- Apply deterministic policy rules.
- Never allow an AI risk score to grant permission.
- Pause for approval when required.
- Revalidate the exact approved action before execution.
- Record decisions and execution outcomes.

### 7.3 Comparison Requirements

Both modes should use, wherever practical:

- The same agent logic.
- The same model configuration.
- The same tool descriptions.
- Equivalent test identities and datasets.
- The same scenario definitions.

The system must distinguish differences caused by security enforcement from unrelated differences in model or data configuration.

## 8. Product Scope

### 8.1 Initial Agent Capabilities

The project assistant will support:

- Natural-language requests.
- Multi-step planning and execution.
- Tool/function calling.
- Context and tool-result handling.
- Recoverable tool failures.
- A final response with execution status.
- Execution history.

### 8.2 Initial Integrations

#### GitHub

Initial operations:

- List authorized repositories.
- Read repository metadata.
- List and read issues.
- Read pull-request metadata.
- Create an issue where permitted.
- Add an issue comment where permitted.

Destructive repository operations, such as deletion, are evaluation scenarios and must not be enabled as ordinary MVP capabilities.

#### Email

Initial operations:

- Search messages.
- Read selected messages.
- Summarize conversations.
- Prepare drafts.
- Send a message only when explicitly authorized and policy-approved.

Email access must use OAuth with minimum necessary scopes. Recipients, subject, body, and attachments must be visible in an approval request when sending requires approval.

#### Controlled REST Tools

Provide a small registry of custom REST tools. Each registered tool must have:

- Name and stable identifier.
- Description.
- HTTP method and endpoint template.
- Input schema.
- Authentication reference.
- Risk classification.
- Required permissions.
- Allowed resource scope.

#### Database Tools

Support controlled read operations and narrowly defined writes where needed for the demo. Use predefined queries or typed operations. Do not provide arbitrary SQL execution to the agent.

#### MCP

MCP support is an architectural extension point in the MVP. A first implementation may register a limited, explicitly supported MCP tool set, but the system must document where enforcement occurs and must not advertise unsupported universal interception.

## 9. AgentVault Security Capabilities

### 9.1 Agent Identity

Each registered agent must have:

- Unique ID.
- Name and description.
- Owning organization.
- Owner or creator.
- Status.
- Registered tools.
- Assigned permissions.
- Creation and update timestamps.

Agent credentials must be scoped, revocable, and distinct from external service credentials.

### 9.2 Authentication

The platform must authenticate:

- Human users.
- Agent callers.
- Internal service requests where applicable.

Authentication must establish a trusted execution context containing user, organization, agent, and correlation identifiers.

### 9.3 Authorization

The MVP must support:

- Organization-level tenant isolation.
- Basic RBAC.
- Resource-level permission checks.
- Tool and operation permissions.
- Explicit deny behavior.

ABAC-style conditions may be introduced where resource, sensitivity, environment, or ownership attributes are necessary. Authorization must be evaluated server-side and must not rely on the model's claims.

### 9.4 Deterministic Policy Engine

Every protected tool call must be evaluated against:

- User identity.
- Agent identity.
- Organization.
- Tool and operation.
- Target resource.
- Arguments.
- Permissions.
- Resource sensitivity.
- Applicable policy rules.

Possible decisions:

- `ALLOW`
- `DENY`
- `REQUIRE_APPROVAL`

Example policies:

- Allow an agent to read issues from an approved repository.
- Deny access to a private repository outside the user's organization.
- Allow email reads but require approval to send.
- Deny repository deletion.
- Allow database reads only from approved tables or predefined query operations.

Policy evaluation must be deterministic, independently testable, explainable, and fail closed when required context is missing.

### 9.5 Advisory Risk Assessment

The platform may use an AI-assisted risk component to identify:

- Prompt-injection indicators.
- Unusual action sequences.
- Suspicious destinations.
- Unexpected action parameters.
- Potential data-exfiltration behavior.

It may return a score, category, explanation, indicators, and recommendation. It must not override a mandatory deny, grant an absent permission, or replace deterministic policy evaluation.

### 9.6 Human Approval

Approval is required for configured sensitive actions, including email sending in the initial product.

An approval request must include:

- Request ID and execution ID.
- Requesting user and agent.
- Tool and operation.
- Target resource.
- Exact arguments or a safe, reviewable representation.
- Risk and policy explanations.
- Expiration time.

Approval states:

- `PENDING`
- `APPROVED`
- `REJECTED`
- `EXPIRED`
- `CANCELLED`
- `EXECUTED`

An approval must be bound to the exact action, resource, and arguments. Any change requires a new policy decision and approval. The system must revalidate authorization and approval immediately before execution.

### 9.7 Secure Execution Gateway

The gateway must:

- Authenticate the request.
- Validate tool existence and input schema.
- Resolve credentials without exposing them to the agent.
- Evaluate policy.
- Create and manage approval requests.
- Execute only authorized operations.
- Apply timeouts and bounded retries.
- Support idempotency for side-effecting operations where the external service permits it.
- Validate, redact, and size-limit tool responses.
- Return structured outcomes.
- Emit audit events for each meaningful state transition.

There must be no alternate protected execution path that bypasses the gateway.

### 9.8 Credential Protection

Requirements:

- Never place raw credentials in prompts, tool descriptions, normal application logs, or audit payloads.
- Do not store plaintext service credentials in the application database.
- Use environment-managed secrets for local development and a dedicated secret manager in production.
- Use OAuth and minimum scopes where available.
- Support revocation and rotation as an operational requirement.
- Log credential access as metadata, never as secret material.

### 9.9 Audit Logging

Protected execution records must capture:

- User, agent, and organization IDs.
- Execution and correlation IDs.
- Tool and operation.
- Target resource.
- Redacted arguments.
- Policy decision and explanation.
- Risk assessment result, if present.
- Approval state and actor.
- Start, completion, and failure timestamps.
- Execution status and error category.
- Idempotency information where relevant.

Audit records should be append-only from the application perspective, searchable, tenant-scoped, and protected from unauthorized modification.

### 9.10 Behavior Monitoring

The MVP should produce security events for:

- Repeated denied calls.
- Unexpected resource access.
- Unusual tool-call frequency.
- Repeated approval rejection.
- Suspicious action sequences.
- Possible data-exfiltration attempts.

Advanced anomaly detection can remain out of scope; the initial implementation may use deterministic thresholds and event rules.

## 10. Functional Requirements

### 10.1 User and Organization Management

- A user can register or sign in.
- A user can create or join an organization.
- All organization-owned data is tenant-scoped.
- A user can be assigned a role.
- Unauthorized cross-organization reads and writes are denied.

### 10.2 Agent Management

- An authorized user can register an agent.
- A user can view, update, disable, and revoke an agent.
- A user can assign tools and permissions to an agent.
- Disabled agents cannot start protected executions.

### 10.3 Tool Registry

- An authorized user can register a tool definition.
- The system validates tool schemas and required metadata.
- A tool can be enabled or disabled.
- A tool can declare operation risk and required permissions.
- Tool definitions are versioned or changes are auditable.

### 10.4 Workflow Execution

- A user can submit a natural-language task.
- A user can choose baseline or protected mode.
- The agent can perform multiple tool calls in one execution.
- The system displays execution progress and final status.
- Tool failures are represented explicitly.
- A failed or denied action cannot be reported as successfully completed.

### 10.5 Policy Management

- An authorized user can create, update, enable, disable, and inspect policies.
- Policies can target agents, users, tools, operations, resources, or organizations.
- The policy engine returns one of the three defined decisions.
- Policy decisions include an explanation suitable for audit and debugging.
- Missing authorization context must not silently default to allow.

### 10.6 Approval Management

- A sensitive action creates an approval request.
- The authorized reviewer can inspect exact action details.
- The reviewer can approve or reject.
- Requests expire and cannot be executed after expiration.
- Changed arguments invalidate prior approval.
- Approval actions are audited.

### 10.7 Audit and Monitoring

- A user can search execution history within authorized tenant scope.
- A user can filter by mode, status, tool, policy decision, agent, and time.
- Sensitive values are redacted.
- Security events are visible separately from ordinary execution records.

### 10.8 Evaluation

- An evaluator can define or select a scenario.
- A scenario can run in both modes.
- Inputs, expected outcomes, actual outcomes, and timing are stored.
- Metrics are calculated from execution records.
- Results can be compared by scenario and mode.
- The system does not allow manually entered values to masquerade as execution metrics.

## 11. Evaluation Plan

### 11.1 Initial Scenarios

1. Unauthorized GitHub repository access.
2. Unauthorized email sending.
3. Prompt injection contained in an email.
4. Attempted repository deletion.
5. Unauthorized database access.
6. Cross-user or cross-organization access.
7. Repeated unauthorized tool calls.
8. Modified arguments after approval.
9. Duplicate execution after a retry.
10. Unexpected access to sensitive information.

Scenarios must use isolated test accounts, synthetic or approved data, and reversible operations wherever possible.

### 11.2 Metrics

- Unauthorized attempts.
- Unauthorized actions executed.
- Unauthorized actions blocked.
- Policy decision correctness.
- Approval enforcement correctness.
- Prompt-injection success rate.
- False-positive rate.
- False-negative rate.
- Legitimate task completion rate.
- Tool execution latency.
- Protected-mode latency overhead.
- Execution failure rate.
- Audit coverage.
- Duplicate side effects.

Metrics must define numerator, denominator, scenario population, and collection method before experiments run.

### 11.3 Dashboard

The comparison dashboard should show:

- Total tool calls.
- Allowed, denied, and approval-required calls.
- Approved, rejected, expired, and cancelled requests.
- Security violations.
- Suspicious events.
- Execution latency.
- Task completion.
- Baseline-versus-protected comparisons.

All displayed values must be derived from stored execution and evaluation records.

## 12. Non-Functional Requirements

### 12.1 Security

- Fail closed for missing authorization context.
- Enforce tenant boundaries on every organization-owned query.
- Validate all tool inputs.
- Redact secrets and sensitive values.
- Use least privilege for integrations.
- Ensure approval is action-specific and time-bounded.
- Preserve an auditable decision trail.

### 12.2 Reliability

- A tool timeout must produce an explicit failure.
- Retries must be bounded and classified as safe or unsafe.
- Side-effecting operations must support idempotency keys where possible.
- Partial workflow completion must be visible.
- Approval and execution state transitions must be consistent.

### 12.3 Performance

The first release should establish measured baselines rather than promise arbitrary scale. It must measure:

- Gateway overhead for allowed calls.
- Approval workflow latency.
- Policy evaluation latency.
- Audit-write latency.
- End-to-end task latency.

### 12.4 Privacy and Data Handling

- Store only data needed for execution, audit, and evaluation.
- Redact or hash sensitive arguments when full values are unnecessary.
- Define retention rules before production deployment.
- Do not use customer content for model training without explicit authorization.

### 12.5 Operability

- Use structured logs with correlation IDs.
- Expose health and readiness information.
- Track failures and policy denials.
- Support database migrations.
- Provide local Docker-based development.
- Document configuration and secret requirements.

### 12.6 Maintainability

- Begin as a modular backend rather than premature microservices.
- Keep policy evaluation independent from the agent's reasoning code.
- Use typed request and response contracts.
- Cover security-sensitive logic with automated tests.
- Keep product, architecture, database, and API documentation synchronized.

## 13. Proposed Technical Direction

These are working recommendations and must be confirmed during HLD:

- Backend: Python, FastAPI, Pydantic, SQLAlchemy, and Alembic.
- Agent orchestration: a provider-neutral tool-calling layer, with LangGraph evaluated for workflow state management.
- Database: PostgreSQL.
- Background work: introduce Redis and a job worker only where approval, evaluation, or integration workloads require it.
- Frontend: Next.js and TypeScript.
- Authentication: OAuth 2.0 for external integrations and a suitable application authentication mechanism.
- Authorization: application policy engine first; evaluate OPA only when its operational value exceeds added complexity.
- Observability: structured logs, metrics, traces, and OpenTelemetry where useful.
- Deployment: Docker Compose for local development; cloud deployment after the core behavior is tested.

Technology selection must follow requirements rather than adopting every proposed component at once.

## 14. High-Level Architecture

The initial system should be a modular application with these logical boundaries:

1. Web dashboard.
2. User and organization management.
3. Agent orchestration.
4. Agent identity management.
5. Tool registry.
6. AgentVault gateway.
7. Policy engine.
8. Risk assessment adapter.
9. Approval service.
10. Secure tool executor.
11. Credential provider.
12. Audit and execution history.
13. Monitoring and evaluation service.
14. PostgreSQL.
15. Optional Redis or background worker.
16. External integrations.

These boundaries should be independently testable without requiring each to be a deployable microservice. The gateway must be the enforcement boundary for protected operations.

## 15. MVP Definition

### 15.1 MVP Must Include

- One general-purpose project assistant.
- Multi-step tool execution.
- GitHub read operations and limited issue operations.
- Email search, read, draft, and approval-controlled send.
- A small set of controlled REST tools.
- Restricted database operations.
- Baseline and protected execution modes.
- Agent registration.
- Tool registry.
- Agent authentication.
- Basic RBAC and resource-level authorization.
- Deterministic policy decisions.
- Human approval workflow.
- Secure credential isolation.
- Centralized audit logs.
- Execution history.
- Repeatable baseline-versus-protected evaluation.
- Core comparison dashboard.
- Automated unit, integration, and security tests.

### 15.2 MVP Acceptance Criteria

The MVP is acceptable when:

1. A user can run the primary project-assistant workflow in baseline mode.
2. The same workflow can run in protected mode without changing the agent's core reasoning flow.
3. Every protected tool call produces an authorization decision.
4. Unauthorized resources are denied.
5. Sending email pauses for approval when configured.
6. Changing approved recipients or content invalidates the approval.
7. Credentials do not appear in prompts, normal logs, or audit records.
8. Execution history reconstructs the tool-call lifecycle.
9. Evaluation scenarios run in both modes with stored results.
10. Dashboard metrics are calculated from actual records.
11. Cross-organization access tests pass.
12. Failure, timeout, retry, and duplicate-execution behavior is documented and tested.

## 16. Post-MVP Scope

Defer the following until the core control plane is reliable:

- Advanced anomaly detection and adaptive behavioral models.
- Broad cloud-infrastructure actions.
- Large-scale multi-agent coordination.
- Universal MCP interception.
- Enterprise SSO, SCIM, and advanced compliance reporting.
- Complex ABAC policy authoring UI.
- Distributed microservice deployment.
- High-volume event streaming.
- Multi-region availability.
- Automated remediation.
- Extensive third-party integrations.

## 17. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Prompt injection changes agent behavior | Unauthorized or harmful tool requests | Treat model output as untrusted; enforce deterministic policy and least privilege; test injected content |
| Gateway bypass | Protected actions escape controls | Make protected tools available only through gateway-managed interfaces; test alternate paths |
| Incorrect resource resolution | Cross-tenant or over-broad access | Resolve and authorize canonical resource identifiers server-side |
| Approval confusion | Reviewer approves an action different from the one executed | Bind approval to a canonical action hash and revalidate exact arguments |
| Duplicate side effects | Repeated emails, issues, or updates | Idempotency keys, bounded retries, and explicit duplicate detection |
| Credential leakage | Account compromise | Secret manager, minimum scopes, redaction, and no credentials in prompts |
| AI risk false positives or negatives | Poor user experience or missed threats | Keep AI advisory; enforce deterministic policy; measure both error types |
| Integration instability | Failed or inconsistent workflows | Timeouts, explicit errors, contract tests, and test accounts |
| Overengineering | Slow delivery and unclear boundaries | Modular monolith first; defer infrastructure until required |
| Misleading comparison | Invalid security claims | Identical scenarios, controlled identities, stored raw outcomes, and defined metrics |
| Sensitive audit data retention | Privacy exposure | Redaction, retention policy, access controls, and documented data minimization |

AgentVault must not claim to make an agent universally safe. It controls defined execution paths and policies; unsupported tools, compromised external systems, incorrect policy configuration, and model-generated content remain limitations.

## 18. Development Lifecycle

Development will proceed phase by phase, with review before moving to the next major phase:

1. **Product discovery and PRD** — confirm scope, personas, use cases, MVP, and success metrics.
2. **Personas and user flows** — document journeys and approval interactions.
3. **High-level design** — define components, boundaries, data flow, and failure handling.
4. **Low-level design** — define modules, interfaces, state transitions, retries, and idempotency.
5. **Database design** — define entities, tenant boundaries, indexes, constraints, audit storage, and evaluation storage.
6. **API and integration contracts** — define authenticated endpoints, schemas, errors, tool contracts, and future SDK/MCP boundaries.
7. **Implementation planning** — create milestones, dependencies, acceptance criteria, and repository conventions.
8. **Baseline application** — build and test the multi-tool assistant with safe restricted credentials.
9. **AgentVault control plane** — add identity, registry, policy, approvals, gateway, credentials, audit, and monitoring.
10. **Comparative evaluation** — run repeatable scenarios and measure security and operational impact.
11. **Quality assurance** — complete unit, integration, security, prompt-injection, recovery, performance, and tenant-isolation testing.
12. **Deployment and operations** — containerization, configuration, secrets, CI/CD, cloud deployment, monitoring, backups, scaling, and runbooks.

No phase should be considered complete without its documented acceptance criteria and review.

## 19. Success Metrics

### Product Success

- A new user can complete the primary workflow with limited setup.
- Developers can register and invoke a controlled tool without reading platform internals.
- Reviewers can understand and decide an approval request quickly.
- Operators can reconstruct a protected execution from audit records.

### Security Success

- Unauthorized test actions are blocked in protected mode.
- Approval-bound argument changes are rejected.
- Cross-tenant access tests consistently fail closed.
- No tested protected execution bypasses the gateway.
- No credentials appear in prohibited surfaces.

### Engineering Success

- Security-sensitive modules have meaningful automated coverage.
- Evaluation results are reproducible.
- Protected-mode overhead is measured and documented.
- Known limitations are visible rather than hidden behind success-shaped fallbacks.

The project must report actual values from experiments rather than defining unsupported numerical guarantees before measurement.

## 20. Product Decisions Requiring Review

Before HLD work, the following decisions should be confirmed:

1. Which initial email provider should be supported?
2. Which LLM provider or provider abstraction is preferred for the first implementation?
3. Should the MVP support one organization per user initially, or organization membership and invitations from the beginning?
4. Which exact GitHub operations are required for the first demo?
5. Which database domain and synthetic dataset should be used for restricted database tools?
6. Should baseline mode be available in the production UI, or only in an isolated evaluation environment?
7. Which users may approve sensitive actions: the request owner, an organization admin, or a dedicated reviewer role?
8. What audit-data retention period is appropriate for local development and a future deployment?
9. Should MCP be implemented in the MVP or limited to a documented extension boundary?
10. What is the first deployment target: local-only, a single cloud environment, or both?
11. Which threat scenarios are release blockers if the MVP fails them?
12. What latency overhead is acceptable for protected execution after the baseline is measured?

## 21. Next Step

Review this PRD and confirm changes to the product scope, MVP, integrations, and decisions above. After approval, the next deliverable should be the user personas and user-flow document, followed by the high-level design.
