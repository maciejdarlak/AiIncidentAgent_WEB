# AI Incident Agent

AI-powered IT incident triage application built with ASP.NET Core, PostgreSQL, pgvector, OpenAI and Ollama.

The application analyzes new IT incidents using RAG and semantic search over more than 16,000 historical support tickets, generates structured incident analysis, proposes an agent action and supports human approval before the action is accepted.

## Live Demo

The application is publicly available at:

https://darlak-ai.onrender.com

<img width="1920" height="1200" alt="1" src="https://github.com/user-attachments/assets/d33b45e5-6431-41d8-9cdc-6faf3c381214" />

### Example incident

**Incident title**
```text
Invoice payment problem
```

**Description**
```text
The customer cannot pay an invoice and receives an error during the payment process.
```

**Error code**
```text
PAYMENT_ERROR
```

**Service**
```text
Billing
```

Click **Analyze incident** to generate the analysis and proposed agent action.

---

## Key Features

- ASP.NET Core Web API
- PostgreSQL database with 16,000+ historical IT support tickets
- RAG-based incident analysis
- pgvector semantic search
- 768-dimensional vector embeddings
- HNSW vector index with cosine distance
- Retrieval of the 5 most relevant historical incidents
- Structured LLM output
- Incident category and priority classification
- AI-generated agent action proposal
- Persistent agent action audit data
- Human-in-the-loop approval workflow
- `Proposed → Approved` action lifecycle
- Approval timestamp (`ApprovedAt`)
- Application and agent activity logging
- Input validation
- Request size limits
- API rate limiting
- Basic PII sanitization
- Prompt-injection hardening
- Automated tests
- Local AI with Ollama
- Production AI with OpenAI
- Docker deployment
- Public deployment on Render

---

## Architecture

```text
                    NEW INCIDENT
                         │
                         ▼
                  Web Frontend
                         │
                         ▼
                 ASP.NET Core API
                         │
                         ▼
                  Input Validation
                         │
                         ▼
                   Embedding Model
                         │
                         ▼
                 Query Vector (768)
                         │
                         ▼
              PostgreSQL + pgvector
                         │
                  HNSW / Cosine
                         │
                         ▼
             5 Similar Historical
                    Incidents
                         │
                         ▼
                 PII Sanitization
                         │
                         ▼
                     RAG Prompt
                         │
                         ▼
                        LLM
                         │
                         ▼
              Structured AI Analysis
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       Incident Analysis       Agent Action
                                    │
                                    ▼
                                Proposed
                                    │
                             Human Approval
                                    │
                                    ▼
                                Approved
                                    │
                                    ▼
                              PostgreSQL
```

---

## How It Works

When a new incident is submitted, the application combines its title, description, error code and service information.

The incident is converted into a 768-dimensional embedding.

PostgreSQL with pgvector performs semantic similarity search against embeddings generated for more than 16,000 historical support tickets.

An HNSW index using `vector_cosine_ops` supports efficient vector retrieval.

The five most semantically similar historical incidents are retrieved.

Only selected historical information is used:

- Subject
- Description
- Historical solution
- Queue

Historical content is sanitized before being added to the LLM context.

The retrieved incidents are then used as RAG context for the language model.

The LLM returns structured information including:

- Probable cause
- Suggested solution
- Recommended team
- Incident category
- Priority
- Proposed agent action

The backend validates and deserializes the AI response into strongly typed .NET models.

---

## Agent Action Workflow

The application goes beyond incident analysis by proposing an operational action.

```text
AI Analysis
     │
     ▼
Proposed Agent Action
     │
     ▼
Stored in PostgreSQL
     │
     ▼
Human Review
     │
     ▼
Approve
     │
     ▼
Status = Approved
ApprovedAt = UTC timestamp
```

Agent actions contain:

- Action type
- Target
- Reason
- Execution instructions
- Status
- Creation timestamp
- Approval timestamp

Actions are intentionally **not automatically executed**.

The current implementation uses a human-in-the-loop workflow where the AI proposes an action and a user explicitly approves it.

External execution through systems such as ServiceNow or Jira can be added later through dedicated tool integrations.

---

## Semantic Search and RAG

Historical ticket search uses embeddings instead of keyword matching.

```text
Historical Tickets
       │
       ▼
Embedding Model
       │
       ▼
768-dimensional vectors
       │
       ▼
PostgreSQL + pgvector
       │
       ▼
HNSW Index
```

For every new incident:

```text
New Incident
     │
     ▼
Embedding
     │
     ▼
Query Vector
     │
     ▼
Cosine Distance
     │
     ▼
Top 5 Historical Incidents
     │
     ▼
RAG Context
     │
     ▼
LLM
```

All 16,000+ historical tickets in the production environment have embeddings generated with OpenAI `text-embedding-3-small`.

Local development uses Ollama with `nomic-embed-text`.

Embeddings stored in each environment are generated using the same embedding model used for queries in that environment.

---

## Structured AI Output

The LLM is instructed to return structured JSON rather than unrestricted free text.

Example:

```json
{
  "probableCause": "...",
  "suggestedSolution": "...",
  "recommendedTeam": "...",
  "category": "...",
  "priority": "...",
  "action": {
    "actionType": "...",
    "target": "...",
    "reason": "...",
    "howToExecute": "...",
    "status": "Proposed"
  }
}
```

The backend deserializes this response into strongly typed C# models before returning the result to the frontend.

---

## Security

The application includes several protection layers.

### Input validation

ASP.NET Core model validation limits incoming incident data:

- Title: 3–200 characters
- Description: 5–5000 characters
- Error code: maximum 100 characters
- Service: maximum 100 characters

### Request protection

The analyze endpoint includes:

- Request size limit
- Rate limiting
- Automatic validation through `[ApiController]`

### AI data protection

Historical ticket data is treated as untrusted input.

Before being sent to the LLM:

- Only selected database fields are retrieved
- Email addresses are removed
- Phone numbers are removed
- Prompt rules instruct the model not to follow instructions contained inside incident data
- Incident content cannot intentionally redefine the application's system behavior

Prompt-injection protection is treated as a defense-in-depth mechanism rather than an absolute security guarantee.

### Secrets

API keys and database credentials are provided through environment configuration and are not stored in source code.

The AI model has no direct database access.

```text
PostgreSQL
     │
     ▼
Repository Layer
     │
     ▼
Selected Fields
     │
     ▼
AI Data Sanitizer
     │
     ▼
RAG Context
     │
     ▼
LLM
```

---

## Observability

Important application operations are logged by the ASP.NET Core backend.

Examples include:

```text
Incident analysis started
Semantic search completed
Incident analysis completed
Agent action approved
```

Logs include operational information such as:

- Number of retrieved historical tickets
- Generated action ID
- Recommended action target
- Action status
- Approval timestamp

Database operations are also visible through Entity Framework Core logging.

---

## Automated Tests

The solution includes an xUnit test project.

Current automated tests cover:

- `Proposed → Approved` workflow
- Approval timestamp
- Attempt to approve an already approved action
- Approval of a non-existing action
- Required incident fields
- Minimum title length
- Maximum title length
- Minimum description length
- Maximum description length
- Maximum error code length
- Maximum service length

Current test suite:

```text
11 Passed
0 Failed
```

Tests use a dedicated PostgreSQL test database with the same pgvector-based schema and EF Core migrations as the application.

---

## Technology Stack

### Backend

- C#
- .NET 8
- ASP.NET Core Web API
- Entity Framework Core
- Repository pattern
- Dependency Injection
- Async/Await

### Database

- PostgreSQL
- Npgsql
- pgvector
- HNSW vector indexing
- Entity Framework Core migrations

### AI / RAG

- OpenAI API
- `text-embedding-3-small`
- Ollama
- `qwen3`
- `nomic-embed-text`
- OllamaSharp
- Semantic Search
- Retrieval-Augmented Generation (RAG)

### Frontend

- HTML
- CSS
- JavaScript

### Infrastructure

- Docker
- Render
- GitHub

### Quality and Security

- xUnit
- ASP.NET Core validation
- Rate limiting
- Request size limiting
- PII sanitization
- Prompt-injection hardening
- Structured logging

---

## Local and Production Architecture

The application uses provider abstractions so local and production environments can use different AI services without changing the core application logic.

```text
LOCAL

ASP.NET Core
     │
     ├── IAiClient
     │      └── Ollama / qwen3
     │
     ├── IEmbeddingClient
     │      └── Ollama / nomic-embed-text
     │
     └── PostgreSQL + pgvector


PRODUCTION

ASP.NET Core
     │
     ├── IAiClient
     │      └── OpenAI
     │
     ├── IEmbeddingClient
     │      └── OpenAI / text-embedding-3-small
     │
     └── PostgreSQL + pgvector
```

The main abstractions are:

- `IAiClient`
- `IEmbeddingClient`
- `ITicketRepository`

This separates application logic from specific AI providers and database implementation details.

---

## Production Status

The complete incident analysis and approval flow is operational in production.

```text
Incident
   │
   ▼
OpenAI Embedding
   │
   ▼
Semantic Search
   │
   ▼
5 Historical Tickets
   │
   ▼
RAG
   │
   ▼
OpenAI Analysis
   │
   ▼
Structured Result
   │
   ▼
Proposed Agent Action
   │
   ▼
PostgreSQL
   │
   ▼
Human Approval
   │
   ▼
Approved
```

Production currently includes:

- 16,000+ historical tickets
- Complete OpenAI embeddings for the historical dataset
- pgvector semantic retrieval
- HNSW indexing
- OpenAI-based incident analysis
- Agent action persistence
- Human approval workflow
- Application logging
- Security controls
- Automated tests
- Public web interface

---

## Future Development

Possible next steps include:

- ServiceNow or Jira tool integration for approved actions
- Authentication and authorization
- More advanced audit history
- Distributed tracing and metrics
- Improved observability dashboards
- Additional integration and end-to-end tests
- More advanced PII detection
- Evaluation dataset for measuring RAG quality
- AI response quality and retrieval metrics

---

## Project Goal

The project demonstrates how traditional .NET backend engineering can be combined with modern LLM application development.

It combines:

```text
.NET Backend Engineering
        +
PostgreSQL / Vector Search
        +
RAG
        +
LLM Integration
        +
Agent Workflow
        +
Human-in-the-Loop
        +
Security
        +
Observability
```

The goal is to build a practical AI-assisted L1 incident triage system rather than a standalone chatbot.

---

## Source Code

The production source code is maintained in a private repository.

**Source code can be shared on request for recruitment or technical evaluation purposes.**
