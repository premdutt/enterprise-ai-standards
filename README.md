# Enterprise AI Architecture Decision Records (ADRs)

This repository contains the approved architectural standards and technical decisions governing AI implementation across the organization.

## How to Propose a New Standard
1. Copy `docs/adr/templates/adr-template.md` to `docs/adr/XXXX-your-title.md`.
2. Fill out all sections, including security and cost trade-offs.
3. Add your entry to the index table below with status **Proposed**.
4. Open a Pull Request. Required reviewers from Architecture, InfoSec, and FinOps will be assigned automatically.

## AI Standards Index

| ADR | Title | Status | Domain |
| :--- | :--- | :--- | :--- |
| [ADR-0001](docs/adr/0001-ai-centralized-ai-gateway.md) | Centralized Enterprise AI Gateway | Proposed | Platform & Access |
| [ADR-0002](docs/adr/0002-ai-data-privacy-and-pii-redaction.md) | Data Privacy & PII Redaction | Proposed | Security & Compliance |
| [ADR-0003](docs/adr/0003-ai-rag-and-knowledge-grounding.md) | RAG & Knowledge Grounding | Proposed | Data & Retrieval |
| [ADR-0004](docs/adr/0004-ai-evaluation-and-runtime-guardrails.md) | AI Evaluation & Runtime Guardrails | Planned | Quality & Safety |
| [ADR-0005](docs/adr/0005-ai-observability-tracing-and-finops.md) | AI Observability, Tracing & FinOps | Planned | Operations & Cost |
