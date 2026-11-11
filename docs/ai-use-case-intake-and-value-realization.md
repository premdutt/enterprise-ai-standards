# AI Use-Case Intake and Value Realization

This playbook explains how to capture, estimate, prioritize, and validate the business value of AI use cases from intake through operation. It complements the [enterprise AI vision and strategy](enterprise-ai-vision-and-strategy.md), which sets the strategic direction and lifecycle expectations.

Use the process proportionately to the use case. It does not replace risk classification, security and privacy review, architecture conformance, procurement, or other required approvals. Portfolio governance retains decisions about investment, funding, and decision authority. Until owners establish common thresholds, teams should document their assumptions and rationale rather than treating an illustrative score or estimate as an approval.

## Principles

* Start with a business problem and a named business owner; consider whether a non-AI approach is simpler or more effective.
* Use the same metric definitions for the initial estimate, pilot success criteria, and production measurement wherever practical.
* Make estimates transparent: record sources, periods, assumptions, uncertainty, and whether an expected benefit is cash savings or released capacity.
* Balance value against effort, risk, and data readiness. A high estimated value does not waive controls or make a use case eligible for production.
* Compare actual results with estimates and use the evidence to scale, fix, stop, and improve future estimates.

## Roles

| Role | Contribution |
| :--- | :--- |
| **Business or product owner** | Defines the problem and intended outcome; validates baseline and benefit assumptions; owns business adoption and benefit realization. |
| **Use-case lead** | Completes the intake, coordinates delivery and measurement, maintains assumptions and evidence, and brings gate recommendations to the accountable decision-makers. |
| **Delivery and data teams** | Estimate feasibility and effort, confirm data access and readiness, build and evaluate the solution, and instrument measurements. |
| **Risk and control stakeholders** | Assess risk, data use, security, privacy, legal, and other obligations within their mandates. |
| **Portfolio governance** | Prioritizes investment and resolves funding or gate decisions under the organization's established decision rights. |

These are responsibilities, not newly assigned approval authorities. Confirm who makes each decision under the applicable portfolio and risk governance.

## Process

### 1. Submit the use case and establish a baseline

Describe the task as it is performed today, including who performs it, how often, and the expected outcome. Use a representative measurement period and cite the source of the baseline (for example, time study, workflow data, or a documented estimate). If reliable data is not available, mark the value as an assumption, state how it will be validated, and avoid presenting it as measured fact.

Capture at minimum:

| Intake field | What to record |
| :--- | :--- |
| Use case and business problem | Task, pain point, intended outcome, and why AI may be appropriate. |
| Business owner and use-case lead | Accountable business contact and person coordinating the intake. |
| Users and affected parties | Who performs the task and who may be affected by its output. |
| Current task time | Typical active time per task, unit of measure, and how it was measured. |
| People and frequency | Number of people doing the task and tasks per person per week, month, or year. Avoid double-counting. |
| Current quality | Error, rework, defect, or other outcome rate, with definition and source. Mark as unavailable if not measured. |
| Data and dependencies | Required data, access and quality status, integrations, and known constraints. |
| Initial risk and autonomy | Known data sensitivity, impact, affected population, and whether the system proposes or takes actions. Complete formal classification through the required process. |
| Effort and cost assumptions | Expected delivery effort, operating cost, dependencies, and estimate confidence. |
| Evidence and assumptions | Sources, measurement period, assumptions, uncertainty, and validation plan. |

### 2. Estimate potential value

Estimate the opportunity over a stated period, normally using annualized figures to compare proposals. Use realistic expected task time after AI, not an idealized best case. Include adoption explicitly: the benefit applies only to the share of eligible work expected to use the solution.

For time saved:

```text
Annual hours released =
  (current hours per task - expected hours per task with AI)
  × tasks per person per year
  × number of people
  × expected adoption rate

Estimated annual capacity value =
  annual hours released × applicable fully loaded hourly cost
```

Use consistent units and do not allow negative time savings to count as a benefit. If only some tasks are eligible, include the eligible share in the adoption rate or task volume, not both.

For quality, estimate avoided rework or errors only when there is a credible baseline and a defensible estimate of change and unit cost:

```text
Estimated quality value =
  eligible annual tasks
  × (current error/rework rate - expected rate with AI)
  × cost per error or rework event
```

Avoid double-counting time already included in the time-savings estimate. Keep other benefits (such as improved service or reduced risk) separate and describe their evidence and valuation method rather than assigning unsupported dollar amounts.

Report released capacity separately from cash savings. Time released is capacity value; it becomes cash savings only when an actual budget, staffing cost, or other cash expense is reduced. Subtract incremental AI, integration, support, and other operating costs when presenting net financial value. State the estimate period and assumptions clearly.

#### Worked example

Suppose 50 planners each perform a task once per week. It takes five hours today and is expected to take one hour with AI, with an assumed fully loaded labor cost of $75 per hour. If all weekly tasks are eligible and expected adoption is 100%:

```text
Hours released = (5 - 1) × 1 × 50 × 52 × 100% = 10,400 hours/year
Capacity value = 10,400 × $75 = $780,000/year
```

This is an illustrative capacity-value estimate, not a forecast of cash savings or a commitment of realized benefits. Replace assumptions with measured task volumes, validated expected time, realistic adoption, and relevant costs. If adoption were 60%, the estimated release would be 6,240 hours, or $468,000 in annual capacity value, before operating costs.

### 3. Assess and prioritize

Compare candidate use cases consistently across the following dimensions:

| Dimension | Consider |
| :--- | :--- |
| **Estimated value** | Magnitude, evidence quality, likely adoption, and strategic fit. |
| **Effort** | Delivery and integration complexity, time to pilot, skills, and ongoing operational burden. |
| **Risk** | Potential impact, data sensitivity, autonomy, reversibility, and outstanding control requirements. Use the formal risk process; do not reduce risk to a value score. |
| **Data readiness** | Data access, quality, permissions, provenance, freshness, and preparation effort. |

Record a comparative rationale and any conditions or evidence needed before proceeding. High value, manageable effort, acceptable risk, and ready data are favorable signals, not an automatic approval. Do not prescribe universal weights or numeric cutoffs until the responsible governance owners establish them.

### 4. Set pilot success criteria

Before building or piloting, agree with the business owner on:

* The baseline period and metric definitions.
* The user group, eligible task volume, pilot duration, and expected adoption.
* Target task time and quality outcomes, including acceptable trade-offs.
* Risk, safety, and operational evaluation criteria and required human review.
* AI and delivery costs to include, measurement owner, and evidence source.
* Conditions for a scale, fix-and-retest, or stop recommendation.

Preserve a comparable baseline through the pilot. Record material changes to task mix, process, staffing, model, prompts, data, or controls that may affect the comparison.

### 5. Measure actuals and make a gate recommendation

At agreed gates, compare pilot or production results with the estimate using the same definitions where possible. Measure:

* Actual task time and eligible task volume.
* Actual adoption and the proportion of eligible users or tasks using the capability.
* Actual error, rework, or other relevant quality outcome.
* Realized capacity value and any cash savings separately.
* AI, integration, support, and operating costs.
* Reliability, risk and safety events, control effectiveness, and user feedback.

Recommend one of the following, with evidence and an accountable owner:

| Recommendation | When it may fit | Next step |
| :--- | :--- | :--- |
| **Scale** | Results are consistent with the intended outcome, value is supported by evidence, users are adopting the solution, and required controls remain effective. | Request the required approvals and operational capacity to expand; continue monitoring value, risk, quality, and cost. |
| **Fix and remeasure** | Results are below expectation, but identified causes are addressable and a time-bounded corrective action can test whether the intended outcome is achievable. | Document the gap, corrective action, owner, and next measurement point; do not expand beyond approved limits. |
| **Stop or retire** | Value is immaterial or not substantiated, the solution is not viable, or risk cannot be acceptably controlled. | Follow portfolio governance for funding and communicate operational, access, and data-retention actions. |

These are decision guides, not universal numeric thresholds. Continue to apply required safety, risk, security, privacy, and production approval gates regardless of business-value results.

### 6. Feed learning back into the portfolio

Record estimates and actuals, including unsuccessful pilots, so future proposals can use better assumptions about task times, adoption, delivery effort, data preparation, quality, and operating cost. Reprioritize the pipeline through portfolio governance as new evidence becomes available. If a use case stops, any released funding is considered for other eligible work through the applicable budgeting and portfolio process; this playbook does not itself reallocate funds.

## Reusable Intake Record

Copy this checklist into the organization's intake or portfolio tooling and adapt it to local governance:

```text
Use case:
Business problem and intended outcome:
Business owner:
Use-case lead:
Users and affected parties:
Current task time and measurement source:
People performing the task:
Task frequency and measurement period:
Current error/rework rate and source:
Expected task time with AI and supporting evidence:
Expected adoption and basis:
Estimated annual capacity value:
Estimated quality value (method and assumptions):
Cash savings (if any, with evidence):
AI, delivery, and operating costs:
Data readiness and dependencies:
Initial risk/autonomy observations; formal classification reference:
Effort and confidence:
Priority rationale and unresolved assumptions:
Pilot success criteria, duration, and measurement owner:
Actual results and estimate comparison:
Gate recommendation (scale / fix / stop), evidence, owner, and date:
Follow-up actions and next review:
```

## Related Guidance

For enterprise lifecycle and governance context, see the [enterprise AI vision and strategy](enterprise-ai-vision-and-strategy.md). For architectural decisions and standards, see the [ADR index](../README.md#ai-standards-index).
