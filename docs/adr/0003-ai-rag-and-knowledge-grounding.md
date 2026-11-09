# ADR-0003: RAG & Knowledge Grounding Standard

* **Status:** Proposed
* **Date:** 2026-11-09
* **Owners:** `[Data Engineering / AI Platform / Enterprise Architecture]`
* **Reviewers:** `[InfoSec / Application Engineering Leads]`

## 1. Context & Problem Statement
Product teams are building Retrieval-Augmented Generation (RAG) workflows to ground Large Language Models in internal enterprise documents, wikis, policies, and customer records. Currently, teams independently select embedding models, vector databases, chunking methods, and retrieval pipelines.

This fragmented approach creates significant operational and security risks:
* **Access Control Leakage:** Naive vector search retrieves top-k chunks globally without verifying whether the requesting user actually has permission to view the underlying source document.
* **Hallucinations & Unverifiable Answers:** Responses lack consistent inline citations, making it impossible for end users to verify claims against authoritative internal sources.
* **Embedding Lock-in & Re-indexing Overhead:** Teams use incompatible embedding dimensions and unversioned vector indexes, forcing full pipeline rebuilds whenever an embedding model changes.
* **Stale Knowledge:** Documents updated or deleted in source systems (e.g., Confluence, SharePoint, Google Drive) remain outdated in vector stores.

We need a standardized RAG architecture that enforces document-level security, verifiable citations, hybrid retrieval quality, and automated lifecycle sync.

## 2. Decision Drivers
* **Permission-Aware Retrieval (ACL Enforcement):** Vector queries must strictly respect source-system Access Control Lists (ACLs) so users only retrieve chunks from documents they are authorized to read.
* **Verifiable Attribution:** User-facing RAG applications must return structured source citations (document title, URI, and section/page reference) for generated claims.
* **Hybrid Search Accuracy:** Retrieval pipelines must combine dense semantic vector search with sparse keyword search (BM25) and cross-encoder reranking to handle exact product codes, acronyms, and legal terms.
* **Zero-Downtime Re-indexing:** Vector collections must include embedding model version metadata to allow blue/green index migrations without service disruption.

## 3. Considered Options

| Option | Pros | Cons |
| :--- | :--- | :--- |
| **Option A: Unified Enterprise Vector Store on Existing Postgres/Search Stack**<br>*(e.g., PostgreSQL + pgvector, OpenSearch, or Elasticsearch)* | • Leverages existing operational expertise, backups, and IAM<br>• Native metadata filtering and hybrid BM25 + vector queries in one engine<br>• No new database vendor required | • Requires careful index tuning (HNSW) at >50M vectors<br>• Teams must build or standardize ingestion connectors |
| **Option B: Dedicated Managed Vector Database**<br>*(e.g., Pinecone, Weaviate Cloud, Qdrant Cloud)* | • Purpose-built vector scaling and low-latency filtering<br>• Built-in hybrid search and multi-tenancy namespaces | • Introduces another external data store requiring InfoSec review<br>• Additional SaaS spend and data synchronization complexity |
| **Option C: Fully Managed Cloud RAG Platform**<br>*(e.g., AWS Bedrock Knowledge Bases, Azure AI Search, Google Vertex AI Search)* | • Turnkey ingestion connectors, chunking, and managed ACL sync<br>• Tight integration with cloud IAM and compliance boundaries | • High vendor lock-in to a single cloud provider<br>• Less granular control over custom chunking and reranking models |

## 4. Decision
We will adopt **`[Option A / Option B / Option C — e.g., Option A: Standardized RAG Ingestion Pipeline backed by PostgreSQL (pgvector) / OpenSearch]`** as the enterprise standard for knowledge grounding.

* **Approved Embedding & Reranking Models:** All embedding and reranking calls must route through the Centralized AI Gateway (`ADR-0001`) using approved standard models (`[e.g., text-embedding-3-large / BGE-M3]` and `[e.g., Cohere Rerank / BGE-Reranker]`).
* **Mandatory Security Metadata:** Every indexed vector chunk must store `tenant_id`, `acl_principals` (user/group IDs), `source_uri`, `last_modified_at`, and `embedding_model_version`. Pre-filtering by `acl_principals` is mandatory at query time.
* **Standardized Chunking & Hybrid Retrieval:** Ingestion pipelines must use structure-aware recursive chunking (`512–1024 tokens` with `10–15%` overlap) rather than fixed character splits, and combine dense + lexical retrieval followed by a reranker before passing top-k context to the LLM.
* **Citation & Abstention Contract:** System prompts must instruct models to cite source IDs inline (`[doc_id]`) and explicitly abstain ("I do not have enough verified information to answer this") when retrieved context falls below the relevance threshold.

## 5. Consequences & Trade-offs
**Positive:**
* **Eliminates Data Exposure:** Enforcing ACL pre-filtering at the vector query layer prevents cross-user and cross-department data leaks.
* **Higher Answer Precision:** Hybrid search plus reranking substantially reduces hallucinations on domain-specific acronyms, SKUs, and policy numbers compared to pure vector similarity.

**Negative & Mitigations:**
* **Increased Retrieval Latency:** Adding hybrid search and a reranking step adds `80ms–150ms` to end-to-end query latency. — *Mitigation:* Stream LLM tokens immediately after retrieval completes and cache frequent semantic queries at the AI Gateway.
* **ACL Sync Complexity:** Keeping permissions synchronized between source systems and the vector database requires webhook or CDC listeners. — *Mitigation:* Enforce a maximum `15-minute` incremental sync SLA and hard-delete vectors immediately upon source document deletion events.

## 6. Compliance & Implementation Checklist
* [ ] Publish shared Python/TypeScript ingestion library supporting structure-aware chunking, PII scrubbing (`ADR-0002`), and mandatory ACL metadata schema.
* [ ] Configure automated Change Data Capture (CDC) or scheduled incremental sync jobs for `[Confluence / SharePoint / Google Drive / S3]`.
* [ ] Implement automated retrieval evaluation metrics (`Recall@K`, `MRR`, and `Faithfulness`) in CI/CD before deploying RAG pipelines to production (`ADR-0004`).
* [ ] Verify index namespace isolation and encryption-at-rest keys with InfoSec.
