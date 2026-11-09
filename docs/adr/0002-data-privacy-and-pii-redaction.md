# ADR-0002: Data Privacy & PII Redaction Standard

* **Status:** Proposed
* **Date:** 2026-11-09
* **Owners:** `[InfoSec / Data Governance / Enterprise Architecture]`
* **Reviewers:** `[Legal / Compliance / Platform Engineering]`

## 1. Context & Problem Statement
As engineering teams adopt Large Language Models (LLMs) and vector search, sensitive enterprise data—including customer Personally Identifiable Information (PII), Protected Health Information (PHI), credentials, and proprietary source code—risks being included in model prompts, system logs, and vector embeddings.

Without an automated privacy layer before external model egress, the enterprise faces severe risks:
* **Regulatory Violations:** Unmasked personal data sent to third-party APIs can violate GDPR, HIPAA, CCPA, and internal data residency policies.
* **Data Leakage in Logs & Traces:** Raw prompts stored in observability platforms or AI Gateway logs expose sensitive customer records to unauthorized internal personnel.
* **Embedding Inversion Risks:** Storing unredacted sensitive data inside vector databases creates long-lived compliance liabilities and complicates "Right to be Forgotten" deletion requests.

We need a standardized, automated privacy enforcement mechanism that intercepts and sanitizes sensitive data before it reaches external model providers or persistent AI storage.

## 2. Decision Drivers
* **Zero-Trust Egress:** Sensitive data (PII, PCI, API secrets) must never leave the enterprise network boundary unmasked unless explicitly authorized under a compliant Data Processing Agreement (DPA).
* **Reversible Masking (Tokenization):** Where business logic requires the LLM to reference entities (e.g., customer support workflows), redaction must support reversible placeholder substitution (`<PERSON_1>`) rather than destructive deletion.
* **Low Latency Impact:** Inline scanning and redaction must add `< 40ms` p95 latency to gateway requests.
* **Zero Data Training:** Upstream LLM providers must be contractually and technically configured for Zero-Data-Retention (ZDR) and zero model training on enterprise payloads.

## 3. Considered Options

| Option | Pros | Cons |
| :--- | :--- | :--- |
| **Option A: Inline Gateway Redaction via Self-Hosted NLP**<br>*(e.g., Microsoft Presidio, spaCy, or DeBERTa NER models integrated into the AI Gateway)* | • Data never leaves enterprise VPC during scanning<br>• No per-token third-party scanning fees<br>• Supports reversible entity tokenization in local memory/Redis | • Requires GPU/CPU sizing and model tuning to reduce false positives<br>• Platform team maintains entity recognition rules |
| **Option B: Cloud-Native DLP APIs**<br>*(e.g., AWS Comprehend PII, Google Cloud Sensitive Data Protection, Azure AI Language)* | • High accuracy out of the box with managed updates<br>• Native compliance certifications (HIPAA, SOC2, FedRAMP) | • Adds external network hop latency<br>• Significant per-character/per-unit API cost at high volume |
| **Option C: Commercial AI Security Proxy**<br>*(e.g., Lakera Guard, Nightfall AI, Protect AI)* | • Combines PII detection with prompt injection and jailbreak defense<br>• Rich compliance reporting dashboards | • High enterprise licensing cost<br>• Adds another third-party vendor to the critical inference path |

## 4. Decision
We will adopt **`[Option A / Option B / Option C — e.g., Option A: Self-hosted Microsoft Presidio deployed as a sidecar/plugin within the Centralized AI Gateway]`** for all outbound prompt traffic and vector ingestion pipelines.

* **Mandatory Inline Scanning:** All requests routed through the AI Gateway (`ADR-0001`) must pass through the PII/Secrets scanner before upstream dispatch, unless the target model is hosted strictly inside an approved internal VPC boundary.
* **Reversible Tokenization:** Sensitive entities detected in prompts are replaced with deterministic session tokens (e.g., `[EMAIL_1]`, `[SSN_1]`) stored in an encrypted, short-lived cache (`TTL <= 15 minutes`) and re-hydrated only in the final response returned to authorized callers.
* **Sanitized Observability:** All gateway logs, OpenTelemetry traces, and evaluation datasets must record only the masked prompt payload.
* **Contractual ZDR Baseline:** No external model endpoint may be enabled in the AI Gateway without a signed DPA and verified Zero-Data-Retention configuration.

## 5. Consequences & Trade-offs
**Positive:**
* **Uniform Compliance:** Product teams get automatic PII and secret protection simply by using the centralized AI Gateway, eliminating custom regex filtering per app.
* **Safe Telemetry:** Engineering and FinOps teams can inspect logs and debug prompt failures without exposing customer data.

**Negative & Mitigations:**
* **False Positives Breaking Context:** Over-aggressive redaction may mask non-sensitive domain terms (e.g., internal product codes mistaken for IDs) and degrade LLM output quality. — *Mitigation:* Maintain a version-controlled allowlist per application domain and provide a documented security exception workflow for approved internal-only endpoints.
* **Added Latency on Large Payloads:** Scanning multi-thousand-token RAG contexts increases response time. — *Mitigation:* Enforce PII scrubbing at document ingestion time before data enters the vector database so retrieval chunks are already clean.

## 6. Compliance & Implementation Checklist
* [ ] Deploy `[Presidio / Cloud DLP]` service in the same network availability zones as the AI Gateway to minimize network latency.
* [ ] Configure secret detection rules (API keys, private keys, JWTs, database connection strings) to block requests immediately with HTTP `400` rather than silently masking.
* [ ] Verify that OpenTelemetry trace exporters strip or hash raw prompt bodies before shipping to `[Datadog / Splunk]`.
* [ ] Establish a quarterly InfoSec audit of gateway routing tables to confirm active ZDR flags on all external provider keys.
