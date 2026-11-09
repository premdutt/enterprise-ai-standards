# ADR-0004: AI Evaluation & Runtime Guardrails

* **Status:** Planned
* **Date:** 2026-11-09
* **Owners:** `[AI Platform / Application Engineering / Enterprise Architecture]`
* **Reviewers:** `[InfoSec / Responsible AI / Product Risk]`

## 1. Context & Problem Statement
AI features can produce inaccurate, unsafe, biased, or policy-violating outputs, and behavior can change when prompts, models, retrieval indexes, or tools change. Conventional unit tests do not adequately measure these probabilistic behaviors. Teams also lack a consistent way to block unsafe requests and responses at runtime or to determine when a model or prompt change is safe to release.

We need a repeatable evaluation lifecycle and layered runtime controls for generative AI applications, including RAG workflows (`ADR-0003`) and traffic routed through the Centralized AI Gateway (`ADR-0001`).

## 2. Decision Drivers
* **Risk-based quality gates:** Release decisions must use representative, versioned evaluation datasets and thresholds appropriate to each use case and its impact.
* **Runtime defense in depth:** Validate inputs, model outputs, retrieved context, and tool calls; do not rely on system prompts as the sole safety control.
* **Measurable behavior:** Track task quality, groundedness, policy compliance, refusal behavior, latency, and cost as applicable to the application.
* **Privacy and reproducibility:** Evaluation data must follow data-handling requirements in `ADR-0002`, and results must identify the tested model, prompt, policy, and dataset versions.
* **Operationally safe releases:** Support staged rollout, monitoring, rollback, and human escalation when automated checks cannot establish acceptable risk.

## 3. Considered Options

| Option | Pros | Cons |
| :--- | :--- | :--- |
| **Option A: Application-specific evaluation and controls** | • Teams can tailor tests and policies to their use cases<br>• Minimal shared-platform dependencies | • Inconsistent coverage and reporting<br>• Duplicated implementation and difficult enterprise oversight |
| **Option B: Central evaluation and guardrail platform** | • Consistent policy enforcement, reporting, and release gates<br>• Shared tooling and reusable evaluators | • Can constrain specialized use cases<br>• Central platform becomes an operational dependency and requires clear ownership |
| **Option C: Provider-native evaluation and safety features** | • Close integration with hosted models<br>• Lower initial setup effort for supported features | • Provider-specific behavior and limited portability<br>• Coverage may not include retrieval, application logic, or cross-provider comparison |

## 4. Decision
We will establish a **shared evaluation and guardrail standard** with centrally governed baseline controls and application-specific risk policies. The enterprise implementation and enforcement points remain planned; production standards must satisfy the following requirements before approval.

* **Evaluation before release:** Material changes to a model, system prompt, retrieval configuration, or tool permissions must be evaluated against a versioned, representative test set before production rollout. CI/CD release gates must include task-specific quality and safety thresholds defined and approved by the application owner and risk stakeholders.
* **RAG evaluation:** RAG applications must measure retrieval quality and answer grounding, including metrics such as `Recall@K`, `MRR`, citation validity, and faithfulness, as relevant to the use case (`ADR-0003`).
* **Runtime controls:** Apply input screening, output policy checks, structured-output validation, and tool authorization at the appropriate gateway or application boundary. Tool access must use explicit allowlists and least privilege. High-impact actions require confirmation or human review; validation failures must not be silently treated as safe output.
* **Risk-based handling:** Define behavior for blocked, uncertain, or out-of-policy results, including safe refusal, escalation, and fallback paths. Applications must not present model-generated content as verified when required checks have failed.
* **Production monitoring and rollback:** Use staged rollout for material changes, monitor quality and safety signals in production, and retain a tested rollback path to a previously approved configuration.
* **No single-score approval:** Automated or model-graded evaluation can inform a release decision but cannot replace human review for high-impact use cases. Evaluators must be validated against human judgments and checked for known blind spots.

## 5. Consequences & Trade-offs
**Positive:**
* **Earlier risk detection:** Repeatable evaluations can catch regressions before release and compare candidate configurations against a known baseline.
* **Consistent protections:** Shared baseline controls reduce the likelihood that an individual application omits essential runtime checks.
* **Clearer release accountability:** Versioned results and approved thresholds make quality and safety decisions reviewable.

**Negative & Mitigations:**
* **Evaluation coverage is incomplete:** Test sets cannot anticipate every real-world input, and model-graded checks may be unreliable. — *Mitigation:* Combine automated tests with human review, production monitoring, adversarial testing, and periodic dataset refreshes.
* **Guardrails can block valid requests or miss unsafe ones:** Screening introduces false positives and false negatives. — *Mitigation:* Measure both, tune per use case, provide an appeal or escalation path, and retain stricter controls for high-impact actions.
* **Evaluation adds release effort and compute cost:** Running large datasets or multiple candidate models can slow delivery and increase spend. — *Mitigation:* Use a tiered suite, run fast checks on each change, and reserve broader evaluations for material or high-risk changes.

## 6. Compliance & Implementation Checklist
* [ ] Define risk tiers, accountable owners, and approval requirements for AI use cases, including high-impact applications.
* [ ] Select and version evaluation datasets; document provenance, access controls, retention, and PII handling in accordance with `ADR-0002`.
* [ ] Implement CI/CD evaluation gates with per-use-case thresholds and retain model, prompt, dataset, and evaluator versions with results.
* [ ] Implement runtime input/output checks, least-privilege tool allowlists, and human confirmation for consequential actions.
* [ ] Establish production quality and safety monitoring, incident response, and rollback procedures, coordinated with `ADR-0005`.
* [ ] Validate automated evaluators against human-reviewed samples and schedule periodic red-team and regression exercises.