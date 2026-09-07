# Phase 1 — Healthcare Document Intelligence & RAG

## Project Goal

Build a production-oriented healthcare document retrieval system using Microsoft Azure.

The system should ingest healthcare documents, index and retrieve relevant information, and use a foundation model to produce grounded answers with citations.

The goal of Phase 1 is to learn the fundamental infrastructure behind a modern **RAG / AI engineering system**, not simply to make a chatbot.

---

# 1. Project Architecture

Build the following architecture:

```text
                         Healthcare PDFs
                               │
                               ▼
                    Azure Blob Storage
                               │
                               ▼
                       Knowledge Source
                               │
                               ▼
                     Azure AI Search
                  ┌────────────┴────────────┐
                  │                         │
             Keyword Search            Vector Search
                  │                         │
                  └────────────┬────────────┘
                               │
                         Hybrid Retrieval
                               │
                               ▼
                         Foundry IQ
                               │
                               ▼
                        Knowledge Base
                               │
                               ▼
                       Microsoft Foundry
                               │
                               ▼
                          AI Agent
                               │
                               ▼
                       Foundation Model
                               │
                               ▼
                    Grounded Answer + Citations
```

Azure AI Search is the retrieval layer. Foundry IQ provides the managed knowledge layer above it. Microsoft documents Foundry IQ as using Azure AI Search's agentic retrieval capabilities for query planning, retrieval, reranking, and grounded responses.

---

# 2. Azure Resources

Create the following resources.

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

---

# 3. Azure Blob Storage

Use Azure Blob Storage as the project's document repository.

## Tasks

1. Create an Azure Storage Account.
2. Create a blob container named `healthcare-documents`.
3. Upload the healthcare PDFs.
4. Organize documents using sensible naming conventions.
5. Add useful metadata where appropriate.

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

The objective is to establish **persistent document storage** rather than relying on files uploaded directly into the Search service.

Azure AI Search currently supports Blob Storage and ADLS Gen2 as indexed knowledge sources and can automatically create the indexing pipeline for Blob content.

---

# 4. Microsoft Foundry

Create the Foundry environment.

## Tasks

1. Create the Foundry resource.
2. Create the Foundry project.
3. Enable the project managed identity.
4. Deploy a suitable foundation model.
5. Record the model deployment name.
6. Create the healthcare RAG agent.

Recommended initial model:

```text
gpt-4.1-mini
```

Do not treat the model as the RAG system.

Understand the separation:

```text
Model
   ↓
Generates language

Azure AI Search
   ↓
Retrieves knowledge

Foundry IQ
   ↓
Coordinates knowledge retrieval

Agent
   ↓
Uses the model + tools + knowledge
```

---

# 5. Azure AI Search

Create a dedicated Azure AI Search service capable of supporting the agentic-retrieval scenario.

## Tasks

1. Create the Search service.
2. Enable system-assigned managed identity.
3. Enable Microsoft Entra/RBAC authentication.
4. Configure the necessary RBAC roles.
5. Connect the Search service to Foundry.
6. Verify that the identities can access the required resources.

Understand the three major retrieval concepts:

```text
Keyword Search
      +
Vector Search
      +
Semantic / Reranking
      ↓
Hybrid Retrieval
```

Azure AI Search supports keyword, vector, and hybrid retrieval, and its agentic retrieval layer can decompose complex queries into subqueries and aggregate results.

---

# 6. Embeddings

Learn what embeddings are and why they are necessary for semantic retrieval.

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
Vector Similarity
   ↓
Relevant Chunks
```

You do not necessarily need to manually implement embeddings in this phase.

The goal is to understand what the managed Azure pipeline is doing underneath the abstraction.

---

# 7. Chunking

Study and test document chunking.

Understand:

* Why documents are divided into chunks
* Chunk size
* Chunk overlap
* Context preservation
* Metadata
* Parent/child relationships
* Retrieval granularity

Conceptually:

```text
Large PDF
   ↓
Text extraction
   ↓
Chunking
   ↓
Embedding
   ↓
Index
```

Test whether changing document structure or chunking affects retrieval quality.

Foundry IQ/Azure AI Search can automate chunking and vectorization for indexed knowledge sources, but an AI engineer should understand what is happening rather than treating it as magic.

---

# 8. Vector Database / Vector Store Knowledge

Understand the role of a vector database/vector index.

Learn the concepts behind:

* Embeddings
* Vector representations
* Cosine similarity
* Approximate nearest-neighbor search
* Vector indexes
* Metadata filtering
* Hybrid search

Understand that:

```text
Azure AI Search
```

can provide vector retrieval and therefore can serve as the retrieval/vector-search layer for this project.

Do **not** add Pinecone yet.

Later, create a separate experiment comparing:

```text
Azure AI Search
        vs
Pinecone
```

The objective is to understand when each technology makes sense.

---

# 9. Foundry IQ Knowledge Base

Create:

```text
kb-healthcare
```

Connect the healthcare knowledge source.

The knowledge base should be capable of querying the healthcare documents and returning grounded content.

Understand the relationship:

```text
Knowledge Source
       ↓
Knowledge Base
       ↓
Foundry IQ
       ↓
Agent
```

A Foundry IQ knowledge base can reference multiple knowledge sources and can be shared by multiple agents.

---

# 10. Foundry Agent

Create:

```text
agent-healthcare-rag
```

Connect:

```text
agent-healthcare-rag
        ↓
kb-healthcare
        ↓
Azure AI Search
        ↓
Healthcare Documents
```

Configure the agent to:

* Use the healthcare knowledge base
* Ground responses in retrieved information
* Provide citations
* Avoid inventing medical information
* Clearly state when information cannot be found

This is a **document retrieval and knowledge assistant**, not a medical diagnosis system.

---

# 11. Retrieval Testing

Create a small evaluation dataset.

For example:

```text
questions.json
```

containing questions such as:

```text
1. Answer exists in document A.
2. Answer exists in document B.
3. Answer requires documents A and B.
4. Similar wording but different answer.
5. Question requiring semantic retrieval.
6. Question requiring keyword retrieval.
7. Question with no answer in the documents.
8. Ambiguous question.
9. Multi-part question.
10. Follow-up question.
```

For each question record:

```text
Question
Expected answer
Retrieved document
Retrieved chunk
Actual answer
Citation
Correct?
```

---

# 12. RAG Evaluation

Do not stop when the chatbot produces an answer.

Evaluate:

### Retrieval quality

* Did the correct document get retrieved?
* Did the correct chunk get retrieved?
* Were irrelevant chunks retrieved?

### Generation quality

* Is the answer correct?
* Is it grounded?
* Does it contain unsupported claims?
* Does it cite the correct source?

### Failure behavior

Test questions that cannot be answered from the knowledge base.

The agent should not simply invent an answer.

---

# 13. Security

Use Microsoft Entra ID and managed identities wherever practical.

Understand:

```text
User
 ↓
Entra ID
 ↓
Managed Identity
 ↓
Azure Resource
```

Learn:

* RBAC
* Managed identities
* Least privilege
* Key-based authentication
* Secrets
* Environment variables
* Azure Key Vault

Do not commit:

```text
.env
API keys
passwords
connection strings
```

to Git.

---

# 14. Python Experiment

Create a small Python client that calls the Foundry agent.

Repository:

```text
azure-rag-healthcare/
```

Suggested structure:

```text
azure-rag-healthcare/
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

---

# 15. FastAPI

Expose the AI application through an HTTP API.

Create:

```text
POST /ask
```

Example:

```json
{
    "question": "What does the document say about..."
}
```

Return:

```json
{
    "answer": "...",
    "sources": []
}
```

Also create:

```text
GET /health
```

Test the API using Swagger/OpenAPI.

Architecture:

```text
Client
  ↓
FastAPI
  ↓
Python AI Application
  ↓
Foundry Agent
  ↓
RAG
```

---

# 16. Containerization

Create a Docker image for the FastAPI application.

Learn:

* Dockerfile
* Images
* Containers
* Environment variables
* Ports
* Container networking
* `.dockerignore`

Architecture:

```text
Angular / Client
       ↓
FastAPI Container
       ↓
Foundry
       ↓
Azure AI Search
       ↓
Healthcare Documents
```

---

# 17. Azure Deployment

Deploy the containerized FastAPI application to Azure.

Recommended first deployment target:

```text
Azure Container Apps
```

Do not start with Kubernetes.

First understand:

```text
Docker
  ↓
Container
  ↓
Azure Container Apps
```

Then later experiment with:

```text
Docker
  ↓
AKS / Kubernetes
```

---

# 18. Observability

Add basic application monitoring.

Learn:

* Application logs
* Request latency
* Error rates
* Token usage
* Retrieval latency
* Search failures
* Model failures

Investigate:

```text
Application Insights
Azure Monitor
```

The objective is to understand how an AI application behaves after deployment.

---

# 19. Healthcare Safety

Because the project uses healthcare documents, explicitly implement responsible-AI boundaries.

The application should:

* Ground responses in provided sources.
* Provide citations.
* Avoid pretending to be a medical professional.
* Avoid making unsupported diagnoses.
* Indicate when information is unavailable.
* Clearly distinguish document information from generated explanation.

Use synthetic, public, or otherwise appropriately licensed documents for the project.

Do not use real patient records or protected health information for experimentation.

---

# 20. Phase 1 Deliverables

At the end of Phase 1, you should have:

```text
[✓] Azure Resource Group

[✓] Microsoft Foundry
[✓] Foundry Project
[✓] Foundation Model Deployment
[✓] Foundry Agent

[✓] Azure Blob Storage
[✓] Healthcare Document Container

[✓] Azure AI Search
[✓] RBAC
[✓] Managed Identity

[✓] Knowledge Source
[✓] Knowledge Base
[✓] Foundry IQ

[✓] Vector / Semantic Retrieval

[✓] Chunking Understanding

[✓] RAG Evaluation Dataset

[✓] Python Client

[✓] FastAPI API

[✓] Docker Container

[✓] Azure Deployment

[✓] Basic Monitoring
```

---

# Phase 1 Final Architecture

```text
                         ┌─────────────────────┐
                         │ Healthcare Documents│
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Azure Blob        │
                         │     Storage         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Knowledge Source    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │  Azure AI Search    │
                         │                     │
                         │ Keyword             │
                         │ Vector              │
                         │ Hybrid              │
                         │ Semantic Retrieval  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     Foundry IQ      │
                         │   Knowledge Base    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Foundry Agent     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Foundation Model    │
                         │    gpt-4.1-mini     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Grounded Response   │
                         │   + Citations       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      FastAPI        │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Angular / Web App   │
                         └─────────────────────┘
```

---

# What This Phase Teaches

By completing Phase 1, you should be able to explain:

* What RAG is.
* Why RAG is useful.
* What embeddings are.
* What a vector store/index does.
* How chunking affects retrieval.
* How hybrid search works.
* What Azure AI Search does.
* What Foundry IQ does.
* What a knowledge source is.
* What a knowledge base is.
* What an AI agent does.
* How a foundation model fits into the architecture.
* How managed identities work.
* How Azure RBAC works.
* How Python communicates with Azure AI services.
* How FastAPI exposes an AI application.
* How Docker packages the application.
* How to deploy the application to Azure.
* How to evaluate RAG quality.
* How to monitor an AI application.
* How to design basic safety boundaries for healthcare AI.

---

# Important: Do Not Build Everything at Once

This is a **roadmap**, not a requirement to create every resource immediately.

Build incrementally:

```text
Stage 1
Azure + Foundry + AI Search
        ↓
Stage 2
Blob Storage + RAG
        ↓
Stage 3
Retrieval evaluation
        ↓
Stage 4
Python
        ↓
Stage 5
FastAPI
        ↓
Stage 6
Docker
        ↓
Stage 7
Azure deployment
        ↓
Stage 8
Monitoring
```

Only move forward when the previous stage works.

---

# Future Phases

## Phase 2 — AI Application Engineering

Add:

* LangChain
* LangGraph
* Explicit application state
* Conversation state
* Working memory
* Episodic memory
* Semantic memory
* Tool calling
* MCP
* Structured outputs
* Retry/error handling
* Agent evaluation

Architecture:

```text
Angular
   ↓
FastAPI
   ↓
LangGraph
   ↓
State
   ├── Working Memory
   ├── Episodic Memory
   └── Semantic Memory
   ↓
Agent
   ↓
Foundry / RAG
```

---

## Phase 3 — Advanced Retrieval

Experiment with:

* Custom chunking
* Metadata filtering
* Hybrid retrieval
* Reranking
* Query rewriting
* Query decomposition
* Retrieval evaluation
* Embedding model comparison
* Azure AI Search vs Pinecone
* Custom vector database experiments

The goal is to understand **why retrieval works or fails**, not merely configure a managed service.

---

## Phase 4 — Agent Engineering

Add:

* Multiple specialized agents
* Agent orchestration
* Planner/executor patterns
* Tool use
* MCP servers
* Human-in-the-loop workflows
* Long-running workflows
* State persistence

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

Add:

* CI/CD
* Infrastructure as Code
* Azure Key Vault
* Private networking
* Managed identities
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

Only after Docker and Azure Container Apps are comfortable:

```text
Docker
   ↓
Kubernetes
   ↓
AKS
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

# Final Project Goal

The eventual project should evolve into:

```text
             Angular Application
                     │
                     ▼
                  FastAPI
                     │
                     ▼
                LangGraph
                     │
            ┌────────┼─────────┐
            │        │         │
            ▼        ▼         ▼
         Memory    Tools     Agents
            │        │         │
            └────────┼─────────┘
                     ▼
              Foundry Agent
                     │
                     ▼
                Foundry IQ
                     │
                     ▼
             Azure AI Search
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       Blob       Vector      Metadata
      Storage      Index       Filters
          │
          ▼
   Healthcare Documents
```

The result is no longer simply a "RAG chatbot."

It becomes a **full AI-engineering system** demonstrating cloud infrastructure, retrieval, LLMs, agents, Python, APIs, state, memory, containers, deployment, evaluation, security, and observability.

**Phase 1 objective: build the foundation before adding complexity.**
