# ADR-0005: AI Observability, Tracing & FinOps

* **Status:** Planned
* **Date:** 2026-11-09
* **Owners:** `[AI Platform / SRE / FinOps]`
* **Reviewers:** `[InfoSec / Data Governance / Enterprise Architecture]`

## 1. Context & Problem Statement
AI request paths often cross application code, retrieval systems, model gateways, and external providers. Without a common telemetry model, teams cannot reliably diagnose latency and failures, understand model and token usage, or allocate spend to the responsible application and cost center. Capturing raw prompts and completions to improve debugging, however, can expose personal, confidential, or regulated data.

We need standardized, privacy-preserving observability and cost management that works across model providers, supports end-to-end troubleshooting, and enables both operational accountability and FinOps forecasting.

## 2. Decision Drivers
* **End-to-end visibility:** Correlate an AI operation across application, retrieval, gateway, and provider calls using consistent trace context.
* **Privacy by default:** Telemetry must not expose raw prompts, completions, credentials, or retrieved document contents; align with `ADR-0002`.
* **Provider portability:** Use vendor-neutral instrumentation and a common semantic schema where practical, rather than coupling applications to one monitoring backend.
* **Reliable usage and cost attribution:** Associate requests and token usage with approved application, team, environment, and cost-center identifiers.
* **Actionable operations:** Provide service-level indicators, alerts, retention controls, and auditability without unbounded telemetry volume or label cardinality.

## 3. Considered Options

| Option | Pros | Cons |
| :--- | :--- | :--- |
| **Option A: Provider-specific telemetry in each application** | • Fast to adopt for a single provider<br>• Exposes provider-specific usage and diagnostics | • Fragmented schemas and dashboards<br>• Makes provider changes and cross-application cost analysis difficult |
| **Option B: OpenTelemetry instrumentation with a centralized backend** | • Vendor-neutral traces, metrics, and context propagation<br>• Central policy for access, retention, dashboards, and alerting | • Requires shared schema, collector operations, and governance<br>• Provider-specific data may need normalization |
| **Option C: Commercial AI observability platform** | • AI-oriented dashboards and evaluation integrations may accelerate adoption<br>• Managed ingestion and analysis | • Additional vendor, data-sharing, and licensing review<br>• Potential lock-in and duplicate telemetry pipelines |

## 4. Decision
We will standardize on **OpenTelemetry-based instrumentation exported through approved enterprise collectors to a centrally governed observability backend**. The selected backend and rollout are planned; the following data and operational requirements apply to the implementation.

* **Trace the full operation:** Propagate trace and request identifiers across application, retrieval, gateway, and model-provider spans. Record model/provider identifiers, operation type, outcome, latency, retry/fallback events, and relevant evaluation or guardrail outcomes.
* **Use governed attributes:** Attach validated `team_id`, `app_id`, `environment`, and `cost_center_id` attributes to requests. Use bounded, non-sensitive labels; do not put user IDs, prompt text, document contents, secrets, or arbitrary request values in metric labels.
* **Protect content:** Do not collect raw prompts, model completions, retrieved passages, or tool payloads in standard logs, metrics, or traces. Record token counts and sanitized metadata only. Any exceptional content capture for debugging requires explicit approval, narrowly scoped access, documented purpose, and a short retention period consistent with `ADR-0002`.
* **Measure operations:** Provide dashboards and alerts for request volume, success and error rates, latency (including time-to-first-token for streaming), rate limits, retries, fallback usage, and guardrail events. Metrics should be continuously collected; traces may be sampled according to policy, with increased capture for errors and incidents where privacy controls permit.
* **Attribute usage and cost:** Capture provider-reported input/output token usage where available, model and pricing version, and estimated request cost. Reconcile estimates against provider invoices or billing exports and report by approved cost-center dimensions. Mark estimates as estimates when provider billing data is delayed or unavailable.
* **Control spend:** Establish budgets and alerts by application and cost center; support usage forecasting and investigation of material spend changes. Any request or token limits that affect runtime behavior must be configured and communicated to application owners.
* **Govern telemetry lifecycle:** Apply role-based access, encryption, retention limits, and audit logging to telemetry. Restrict high-cardinality dimensions and review dashboards, exporters, and retention policies periodically.

## 5. Consequences & Trade-offs
**Positive:**
* **Faster incident diagnosis:** Correlated traces distinguish application, retrieval, gateway, and provider latency or failure.
* **Privacy-aware operations:** Sanitized telemetry provides useful operational signals without making prompt contents a routine observability artifact.
* **Improved cost accountability:** Consistent usage dimensions support budgeting, forecasting, and chargeback across providers and applications.

**Negative & Mitigations:**
* **Instrumentation and schema require ongoing ownership:** Teams must emit consistent attributes and maintain collectors and dashboards. — *Mitigation:* Publish shared SDK/configuration, validate required attributes in CI, and assign platform ownership for the schema.
* **Sampling and provider reporting can limit precision:** Sampled traces are not a complete usage ledger, and provider token or cost data may arrive late or differ from estimates. — *Mitigation:* Use unsampled aggregate usage metrics where supported and reconcile cost reports against billing exports; do not calculate chargeback from sampled spans.
* **Telemetry still carries metadata risk and cost:** Identifiers, dimensions, and high-volume spans can expose operational context or increase storage spend. — *Mitigation:* Enforce attribute allowlists, access controls, retention limits, cardinality budgets, and sampling policies.

## 6. Compliance & Implementation Checklist
* [ ] Publish the AI telemetry schema and shared OpenTelemetry instrumentation for required trace and metric attributes.
* [ ] Configure collectors to redact or reject prohibited content fields before export and verify this behavior with automated tests.
* [ ] Establish dashboards and alerts for reliability, latency, guardrail outcomes, token usage, and spend by approved cost-center dimensions.
* [ ] Integrate provider usage and billing exports; document reconciliation cadence, pricing versioning, and estimation limitations.
* [ ] Define backend access controls, audit logging, retention, sampling, and cardinality policies with InfoSec and Data Governance.
* [ ] Validate that sampled tracing is not used as the source of truth for token totals or chargeback.