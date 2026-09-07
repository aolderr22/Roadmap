# Phase 1 — Healthcare Document Intelligence & RAG

## Project Goal

Build a production-oriented healthcare document retrieval system using Microsoft Azure.

The system will ingest healthcare documents, index and retrieve relevant information, and use a foundation model to produce grounded answers with citations.

The goal is to learn the fundamental infrastructure behind a modern RAG / AI engineering system rather than simply building a chatbot.

> **Important:** Microsoft Foundry, Foundry IQ, and Azure AI Search are evolving rapidly. Treat the Azure portal UI and individual configuration screens as implementation details. Prefer the current Microsoft documentation when a portal label differs from this roadmap.

---

# 1. Target Architecture

Build toward:

```text
Healthcare Documents
        │
        ▼
Azure Blob Storage
        │
        ▼
Knowledge Source
        │
        ▼
Azure AI Search
        │
        ├── Keyword Search
        ├── Vector Search
        ├── Hybrid Search
        └── Semantic Reranking
        │
        ▼
Knowledge Base
        │
        ▼
Foundry IQ
        │
        ▼
Foundry Agent
        │
        ▼
Foundation Model
        │
        ▼
Grounded Answer + Citations
```

Current Microsoft documentation describes Foundry IQ as a managed knowledge layer for agents. Azure AI Search provides the agentic retrieval engine underneath, including query planning, subqueries, retrieval, reranking, and references for citations.

---

# 2. Recommended Azure Resources

| Resource                   | Suggested Name          |
| -------------------------- | ----------------------- |
| Resource Group             | `rg-healthcare-ai`      |
| Microsoft Foundry resource | `foundry-healthcare-ai` |
| Foundry project            | `proj-healthcare-ai`    |
| Azure AI Search            | `search-healthcare-ai`  |
| Storage Account            | `sthealthcaredocs`      |
| Blob Container             | `healthcare-documents`  |
| Model deployment           | `gpt-4.1-mini`          |
| Knowledge source           | `ks-healthcare-docs`    |
| Knowledge base             | `kb-healthcare`         |
| Foundry agent              | `agent-healthcare-rag`  |

Use a consistent Azure region where practical.

Resource names may need to be modified to satisfy Azure naming restrictions.

---

# 3. Azure Blob Storage

Use Azure Blob Storage as the persistent document repository.

## Tasks

1. Create an Azure Storage Account.
2. Create a container named `healthcare-documents`.
3. Upload the healthcare documents.
4. Organize documents logically.
5. Add useful metadata where appropriate.
6. Configure the required Microsoft Entra/RBAC permissions.
7. Connect the storage data to Azure AI Search through an appropriate knowledge source.

Example:

```text
healthcare-documents/
│
├── cardiology/
│   ├── heart-disease-guidelines.pdf
│   └── cardiovascular-treatment.pdf
│
├── diabetes/
│   ├── diabetes-guidelines.pdf
│   └── diabetes-treatment.pdf
│
└── general/
    └── healthcare-policy.pdf
```

Azure AI Search currently supports indexed knowledge sources backed by Azure Blob Storage and ADLS Gen2. Indexed knowledge sources use Azure AI Search indexing infrastructure to ingest and refresh content.

The objective is to establish persistent document storage rather than relying exclusively on manually uploaded files.

---

# 4. Microsoft Foundry

Create the Foundry environment.

## Tasks

1. Create the Microsoft Foundry resource.
2. Create the Foundry project.
3. Enable the project managed identity.
4. Deploy a suitable chat model.
5. Record the deployment name.
6. Create the healthcare agent.

Initial model:

```text
gpt-4.1-mini
```

The model is only one component of the system.

Understand the separation:

```text
Foundation Model
       │
       ▼
Generates language

Azure AI Search
       │
       ▼
Retrieves knowledge

Foundry IQ
       │
       ▼
Provides managed knowledge access

Foundry Agent
       │
       ▼
Uses model + knowledge + tools
```

The specific model used may change as the project evolves.

---

# 5. Azure AI Search

Create an Azure AI Search service that supports the agentic-retrieval scenario.

## Tasks

1. Create the Search service.
2. Enable a system-assigned managed identity.
3. Configure Microsoft Entra/RBAC authentication.
4. Assign the required RBAC roles.
5. Connect the Search service to the required Azure services.
6. Verify identity-based access.
7. Create the knowledge source.
8. Create the knowledge base.

Understand the retrieval stack:

```text
Keyword Search
      +
Vector Search
      +
Hybrid Search
      +
Semantic Reranking
      ↓
Agentic Retrieval
```

Current Azure AI Search documentation describes agentic retrieval as a multi-query retrieval pipeline. A knowledge base can decompose complex questions into subqueries, execute them against knowledge sources, use keyword/vector/hybrid retrieval, rerank results, and retain source references.

---

# 6. Knowledge Sources

Create one or more knowledge sources.

Initial knowledge source:

```text
ks-healthcare-docs
```

Use the knowledge-source type appropriate to the current Microsoft Foundry/Azure AI Search portal.

For this project, prefer an indexed source backed by Azure Blob Storage once the basic RAG pipeline is working.

Understand:

```text
Blob Storage
      ↓
Knowledge Source
      ↓
Azure AI Search Index
```

A knowledge base can reference multiple knowledge sources. This means the project should be designed so additional document collections can be added later without rebuilding the entire Azure environment.

---

# 7. Embeddings

Learn what embeddings are and why vector retrieval works.

Understand:

```text
Document
   ↓
Chunks
   ↓
Embedding Model
   ↓
Vectors
   ↓
Vector Index
```

And:

```text
User Question
   ↓
Embedding Model
   ↓
Query Vector
   ↓
Vector Search
   ↓
Relevant Content
```

Learn:

* Embeddings
* Vector representations
* Dimensions
* Similarity
* Cosine similarity
* Approximate nearest-neighbor search
* Vector indexes
* Metadata filtering

Do not manually implement the entire embedding pipeline initially.

First understand what the managed Azure pipeline is doing.

---

# 8. Chunking

Understand how documents become retrievable pieces of information.

Study:

* Text extraction
* Chunk boundaries
* Chunk size
* Chunk overlap
* Context preservation
* Metadata
* Parent/child relationships
* Retrieval granularity

Conceptually:

```text
PDF
 ↓
Text Extraction
 ↓
Chunking
 ↓
Embedding
 ↓
Index
```

Test how document structure and chunking influence retrieval quality.

The objective is not merely to know that "chunking happens."

The objective is to understand why poor chunking can produce poor RAG results.

---

# 9. Vector Database / Vector Store Knowledge

Understand the role of a vector database/vector index.

Learn:

* Embeddings
* Vector similarity
* Approximate nearest-neighbor search
* Vector indexes
* Metadata filtering
* Hybrid search
* Reranking

Understand that Azure AI Search can provide vector retrieval and therefore serves as the retrieval/vector-search layer for this project.

Do not add Pinecone to the primary project yet.

Later create a separate experiment:

```text
Azure AI Search
       vs
Pinecone
```

Compare:

* Architecture
* Indexing
* Metadata filtering
* Hybrid retrieval
* Embeddings
* Operational complexity
* Cost
* Azure integration
* Developer experience

The goal is to understand the technology rather than accumulate services.

---

# 10. Knowledge Base

Create:

```text
kb-healthcare
```

Connect the healthcare knowledge source(s).

Understand:

```text
Knowledge Source(s)
        ↓
Knowledge Base
        ↓
Agentic Retrieval
        ↓
Retrieved Knowledge
```

A knowledge base is the object that orchestrates retrieval across configured knowledge sources. Current Azure AI Search documentation describes the knowledge base as a top-level object for agentic retrieval.

---

# 11. Foundry IQ

Use Foundry IQ as the managed knowledge layer for the agent.

Understand:

```text
Knowledge Sources
       ↓
Knowledge Base
       ↓
Foundry IQ
       ↓
Foundry Agent
```

Foundry IQ is designed to provide reusable, permission-aware knowledge to agents and can connect knowledge bases to multiple agents.

Do not think of Foundry IQ as another vector database.

Instead:

```text
Azure AI Search
    =
Retrieval / search infrastructure

Foundry IQ
    =
Managed knowledge layer

Foundry Agent
    =
Agentic application layer
```

---

# 12. Foundry Agent

Create:

```text
agent-healthcare-rag
```

Connect the healthcare knowledge base.

Configure the agent to:

* Use the healthcare knowledge base.
* Ground answers in retrieved information.
* Provide citations.
* Avoid unsupported claims.
* State when information cannot be found.
* Follow the project's healthcare safety instructions.

The agent is a document-information assistant, not a medical diagnosis system.

---

# 13. RAG Testing

Create test questions covering several categories.

## Document A

Ask questions whose answers exist only in Document A.

## Document B

Ask questions whose answers exist only in Document B.

## Multiple Documents

Ask questions requiring information from multiple documents.

## No Answer

Ask questions whose answers do not exist in the knowledge base.

## Semantic Retrieval

Ask questions using different wording from the source documents.

## Keyword Retrieval

Ask questions containing important exact terminology.

## Multi-Part Questions

Ask questions requiring multiple pieces of evidence.

## Follow-Up Questions

Test whether the agent maintains appropriate conversational context.

---

# 14. RAG Evaluation

Create a small evaluation dataset.

For example:

```text
questions.json
```

Track:

```text
Question
Expected Answer
Expected Source
Retrieved Source
Retrieved Content
Actual Answer
Citation
Correct?
Grounded?
```

Evaluate:

### Retrieval

* Was the correct source retrieved?
* Was relevant content retrieved?
* Were irrelevant results returned?

### Generation

* Is the answer correct?
* Is the answer grounded?
* Does it contain unsupported claims?
* Are citations appropriate?

### Failure Handling

* Does the agent correctly say when the knowledge base lacks the answer?
* Does it avoid hallucinating?

---

# 15. Retrieval Observability

Learn how to inspect what the retrieval system is actually doing.

Investigate:

* Query plans
* Subqueries
* Retrieved documents
* Retrieved chunks
* Ranking
* Reranking
* Citations
* Latency
* Token usage

Azure AI Search's current agentic-retrieval tooling exposes retrieval activity that can be used to understand how a query was processed.

This is important because an AI engineer needs to diagnose:

```text
Bad Answer
    ↓
Was retrieval bad?
    ↓
Was the prompt bad?
    ↓
Was the model bad?
    ↓
Was the source document bad?
```

---

# 16. Security

Use Microsoft Entra ID and managed identities wherever practical.

Learn:

* RBAC
* Managed identities
* Least privilege
* Microsoft Entra authentication
* API keys
* Environment variables
* Secrets
* Azure Key Vault

Prefer:

```text
Managed Identity
```

over long-lived API keys for Azure-to-Azure production authentication where supported.

Never commit:

```text
.env
API keys
Passwords
Connection strings
Secrets
```

to Git.

---

# 17. Python Application

Create a Python client.

Suggested repository:

```text
azure-healthcare-rag/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   └── agent.py
│
├── tests/
│
├── .env
├── .gitignore
├── requirements.txt
└── README.md
```

Initially:

```text
Terminal
   ↓
Python
   ↓
Foundry Agent
   ↓
Foundry IQ
   ↓
Azure AI Search
```

The Python application should eventually become the backend of the application rather than relying on the Foundry portal UI.

---

# 18. FastAPI

Expose the AI application through HTTP.

Create:

```text
GET  /health
POST /ask
```

Example:

```json
{
    "question": "What does the healthcare documentation say about..."
}
```

Return structured data such as:

```json
{
    "answer": "...",
    "sources": []
}
```

Architecture:

```text
Angular
   ↓
FastAPI
   ↓
Python AI Application
   ↓
Foundry Agent
   ↓
Foundry IQ
   ↓
Azure AI Search
```

Test the API through Swagger/OpenAPI.

---

# 19. Docker

Containerize the FastAPI application.

Learn:

* Dockerfile
* Images
* Containers
* Environment variables
* Ports
* `.dockerignore`
* Container networking
* Health checks

Architecture:

```text
Angular
   ↓
FastAPI Container
   ↓
Foundry
   ↓
Azure AI Search
```

---

# 20. Azure Deployment

Deploy the containerized backend to Azure.

Recommended first target:

```text
Azure Container Apps
```

Learn:

```text
Docker
   ↓
Container
   ↓
Azure Container Apps
```

Do not begin with Kubernetes.

Kubernetes/AKS should come later after Docker and Azure Container Apps are understood.

---

# 21. Monitoring

Add application and infrastructure monitoring.

Investigate:

* Azure Monitor
* Application Insights
* Application logs
* Request latency
* Error rates
* Retrieval latency
* Model latency
* Token usage
* Search failures

Learn how to diagnose production failures.

---

# 22. Healthcare Safety

Use only synthetic, public, or appropriately licensed documents.

Do not use real patient records or protected health information for this learning project.

The application should:

* Ground answers in the provided sources.
* Provide citations.
* Avoid unsupported medical advice.
* Avoid pretending to diagnose patients.
* State when information is unavailable.
* Clearly distinguish source information from generated explanation.

---

# 23. Phase 1 Deliverables

By the end of Phase 1:

```text
[ ] Azure Resource Group

[ ] Microsoft Foundry
[ ] Foundry Project
[ ] Foundation Model Deployment
[ ] Foundry Agent

[ ] Azure Blob Storage
[ ] Healthcare Document Container

[ ] Azure AI Search
[ ] Managed Identity
[ ] RBAC

[ ] Knowledge Source
[ ] Knowledge Base
[ ] Foundry IQ

[ ] Vector Retrieval
[ ] Hybrid Retrieval
[ ] Semantic Reranking

[ ] Embedding Understanding
[ ] Chunking Understanding

[ ] RAG Evaluation Dataset
[ ] Retrieval Evaluation

[ ] Python Client
[ ] FastAPI API

[ ] Docker Container

[ ] Azure Deployment

[ ] Monitoring
```

---

# 24. Final Phase 1 Architecture

```text
                 Healthcare Documents
                         │
                         ▼
                 Azure Blob Storage
                         │
                         ▼
                  Knowledge Source
                         │
                         ▼
                Azure AI Search
                         │
             ┌───────────┼───────────┐
             │           │           │
          Keyword      Vector     Hybrid
             │           │           │
             └───────────┼───────────┘
                         │
                  Semantic Ranking
                         │
                         ▼
                   Knowledge Base
                         │
                         ▼
                    Foundry IQ
                         │
                         ▼
                   Foundry Agent
                         │
                         ▼
                  Foundation Model
                         │
                         ▼
              Grounded Answer
                 + Citations
                         │
                         ▼
                      FastAPI
                         │
                         ▼
                  Angular Client
```

---

# 25. Future Phases

## Phase 2 — AI Application Engineering

Learn and implement:

* LangChain
* LangGraph
* State management
* Conversation state
* Working memory
* Long-term memory
* Tool calling
* MCP
* Structured outputs
* Error handling
* Retries
* Agent evaluation

---

## Phase 3 — Advanced Retrieval

Experiment with:

* Custom chunking
* Metadata filtering
* Hybrid search
* Query rewriting
* Query decomposition
* Reranking
* Embedding model comparison
* Retrieval evaluation
* Azure AI Search vs Pinecone
* Custom vector database experiments

---

## Phase 4 — Agent Engineering

Build multi-agent workflows.

Explore:

* Agent orchestration
* Planner/executor patterns
* Specialized agents
* Tool-using agents
* MCP servers
* Human-in-the-loop workflows
* Long-running workflows
* Persistent state

Example:

```text
                    Orchestrator
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Retrieval Agent  Analysis Agent  Citation Agent
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    Final Answer
```

---

## Phase 5 — Production AI Engineering

Learn:

* CI/CD
* Infrastructure as Code
* Azure Key Vault
* Managed identities
* Private networking
* Application Insights
* Distributed tracing
* Load testing
* Security
* Cost optimization
* Model evaluation
* RAG evaluation
* Prompt/version management
* Production deployment

---

## Phase 6 — Kubernetes

After Docker and Azure Container Apps:

```text
Docker
   ↓
Kubernetes
   ↓
Azure Kubernetes Service
   ↓
Scalable AI Application
```

Learn:

* Pods
* Deployments
* Services
* Ingress
* ConfigMaps
* Secrets
* Horizontal scaling
* Health probes
* Observability

---

# 27. Learning Philosophy

This roadmap is intentionally incremental.

Do not build every component simultaneously.

Use:

```text
Learn
  ↓
Build
  ↓
Test
  ↓
Understand
  ↓
Document
  ↓
Improve
```

Do not add a technology merely because it is popular.

Every technology should answer a question:

> **What engineering problem does this technology solve?**

---

# 28. Progress Tracking

After beginning the project, maintain a separate file:

```text
progress.txt
```

Use it to record:

* What has been completed.
* What is currently working.
* What failed.
* Errors encountered.
* Azure resources created.
* Configuration decisions.
* Authentication/RBAC decisions.
* Important discoveries.
* Code changes.
* Current project state.
* Next recommended step.

Example:

```text
DATE: 2026-09-XX

COMPLETED:
- Created resource group.
- Created Foundry project.
- Created Azure AI Search.
- Configured managed identity.
- Created knowledge source.
- Connected knowledge base.

CURRENT:
- Python client working.
- FastAPI not started.

PROBLEMS:
- ...

DECISIONS:
- ...

NEXT:
- ...
```

`LearningRoadmap.md` describes **where the project is going**.

`progress.txt` describes **where the project actually is**.

---

# Phase 1 Objective

By completing Phase 1, be able to explain and demonstrate:

* RAG
* Embeddings
* Chunking
* Vector retrieval
* Hybrid retrieval
* Semantic ranking
* Azure AI Search
* Knowledge sources
* Knowledge bases
* Foundry IQ
* Foundry agents
* Foundation models
* Managed identities
* RBAC
* Python AI applications
* FastAPI
* Docker
* Azure deployment
* RAG evaluation
* Monitoring
* Basic AI safety

The final result should be more than a chatbot.

It should be a **small but realistic AI engineering system** that demonstrates the ability to design, build, evaluate, secure, and deploy an AI-powered application.
