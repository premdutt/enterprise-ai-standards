# ADR-0001: Centralized Enterprise AI Gateway

* **Status:** Proposed
* **Date:** 2026-11-09
* **Owners:** `[Enterprise Architecture / Platform Engineering]`
* **Reviewers:** `[InfoSec / FinOps / AppSec]`

## 1. Context & Problem Statement
Multiple product and internal engineering teams are integrating Large Language Models (LLMs) and embedding APIs into their applications. Currently, teams connect directly to external model providers (e.g., OpenAI, Anthropic, Google, AWS Bedrock) using isolated SDKs and scattered API keys.

This decentralized approach introduces critical enterprise risks:
* **Security & Compliance Blind Spots:** API keys are duplicated across repositories, and there is no central enforcement of Zero-Data-Retention (ZDR) policies or PII redaction.
* **Vendor Lock-in:** Application business logic is tightly coupled to provider-specific request and response schemas.
* **Lack of Cost Visibility:** Token consumption cannot be cleanly attributed to specific business units, products, or cost centers.
* **Reliability Risks:** Individual teams must independently implement rate-limit handling, retries, and multi-provider fallbacks.

We need a standardized architectural control plane to govern all inbound and outbound AI model traffic across the organization.

## 2. Decision Drivers
* **Unified Interface:** Application teams must interact with a single, OpenAI-compatible API schema regardless of the underlying model provider.
* **Zero-Trust Security:** Eliminate raw provider API keys in application code by authenticating via enterprise Identity Provider (IdP) or service principals.
* **Observability & Chargeback:** Capture 100% of token usage, latency, and error metrics tagged by `team_id`, `app_id`, and `environment`.
* **Low Latency Overhead:** Gateway processing must add `< 25ms` p99 latency to non-streaming and time-to-first-token (TTFT) metrics.
* **High Availability:** Support automatic failover across regions or providers during upstream outages.

## 3. Considered Options

| Option | Pros | Cons |
| :--- | :--- | :--- |
| **Option A: Open-Source Proxy Self-Hosted**<br>*(e.g., LiteLLM, Portkey OSS, Envoy AI Gateway)* | • Full control over data plane & custom plugins<br>• Zero vendor markup on token volume<br>• Strong multi-provider schema translation | • Platform team owns scaling, patching, and uptime<br>• Requires building or integrating custom admin UI & RBAC |
| **Option B: Cloud-Native AI Gateway**<br>*(e.g., AWS Bedrock Gateway, Azure API Management + AI, GCP Apigee/Vertex)* | • Native IAM, VPC peering, and compliance inheritance<br>• Managed scaling and SLA backed by cloud vendor<br>• Unified cloud billing | • Weak support for external models outside that cloud<br>• Less flexible routing and fallback logic across clouds |
| **Option C: Commercial SaaS AI Gateway**<br>*(e.g., Portkey Enterprise, Kong AI Gateway, Cloudflare AI Gateway)* | • Fastest time to production with rich FinOps dashboards<br>• Built-in semantic caching, guardrails, and prompt management | • Additional per-request or volume-based licensing cost<br>• Introduces a third-party vendor into the critical data path |

## 4. Decision
We will adopt **`[Option A / Option B / Option C — e.g., Option A: Self-hosted LiteLLM Proxy deployed on internal Kubernetes]`** as the mandatory egress point for all generative AI traffic because it satisfies multi-provider portability while keeping our data plane inside our existing enterprise network boundary.

* **Standard Protocol:** All internal applications must use standard OpenAI-compatible client SDKs pointed to `https://ai-gateway.[company].internal/v1`.
* **Direct Egress Block:** Network security groups and egress firewalls will block direct outbound traffic from application workloads to known external LLM provider endpoints (`://openai.com`, `://anthropic.com`, etc.).
* **Authentication & Chargeback:** Applications must authenticate using short-lived internal virtual keys mapped to a valid `cost_center_id`.

## 5. Consequences & Trade-offs
**Positive:**
* **Instant Portability:** Switching an application from one model to another (such as during a price drop or outage) requires only a gateway configuration change with zero application code changes.
* **Centralized Guardrails:** Security and FinOps can enforce token budgets, rate limits, and PII scrubbing globally in one place.

**Negative & Mitigations:**
* **Single Point of Failure:** If the gateway goes down, all dependent AI features fail. — *Mitigation:* Deploy across multiple availability zones with horizontal autoscaling, circuit breakers, and a documented emergency bypass procedure for Tier-0 services.
* **Feature Lag on Bleeding-Edge APIs:** New provider-specific features (such as proprietary audio/video endpoints or custom tool-calling formats) may lag behind the unified schema. — *Mitigation:* Provide an isolated, audited passthrough route for approved R&D sandboxes.

## 6. Compliance & Implementation Checklist
* [ ] Integrate enterprise SSO/OIDC for developer self-service virtual key generation.
* [ ] Configure OpenTelemetry export to `[Datadog / Splunk / Prometheus]` for latency and token cost dashboards.
* [ ] Attach verified Zero-Data-Retention (ZDR) agreements to all upstream provider accounts configured in the gateway.
* [ ] Publish a 30-day migration guide and SDK wrapper examples for Python, TypeScript, and Java.
