# AI Incident Agent

AI-assisted IT incident analysis application built with ASP.NET Core, PostgreSQL, pgvector and OpenAI.

The application analyzes new IT incidents using semantic search over more than 16,000 historical support tickets and generates structured AI recommendations.

## Live Demo

The application is publicly available at:

https://darlak-ai.onrender.com

<img width="1920" height="1200" alt="AI Incident Agent" src="https://github.com/user-attachments/assets/9547e8ac-2710-46c0-99f3-6a4d8b1583de" />

### Example incident

Use the following data to test the application:

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

Click **Analyze incident** to generate the analysis.

---

## Current functionality

The application currently supports:

- Web interface for entering IT incidents
- ASP.NET Core Web API backend
- PostgreSQL database with more than 16,000 historical support tickets
- pgvector integration
- Vector embeddings with 768 dimensions
- Semantic search based on cosine distance
- HNSW index for efficient vector search
- Retrieval of up to 5 semantically related historical tickets
- Limited data projection before sending historical data to AI
- Basic PII sanitization for email addresses and phone numbers
- Historical incidents used as additional AI context
- Structured AI response containing:
  - Probable cause
  - Suggested solution
  - Recommended support team
- Local AI support through Ollama
- Local embeddings through `nomic-embed-text`
- Cloud AI support through OpenAI API
- Cloud embeddings through `text-embedding-3-small`
- Docker deployment
- Public deployment on Render

---

## How it works

```text
New Incident
     ↓
Web Frontend
     ↓
ASP.NET Core API
     ↓
Embedding Model
     ↓
768-dimensional vector
     ↓
PostgreSQL + pgvector
     ↓
Semantic Search
     ↓
5 most relevant historical tickets
     ↓
Data Sanitization
     ↓
LLM
     ↓
Structured Incident Analysis
     ↓
Web Frontend
```

When a new incident is submitted, its content is converted into a vector embedding.

The application compares this vector with embeddings of historical tickets stored in PostgreSQL using pgvector and cosine distance.

The five most semantically similar historical incidents are selected as additional context for the AI.

Only selected historical information is passed to the model:

- Subject
- Description
- Historical solution
- Queue

The historical data is sanitized before being included in the AI prompt.

The new incident remains the primary source of information, while historical tickets provide additional context for the analysis.

---

## Semantic search

Historical ticket search uses vector embeddings instead of simple keyword matching.

```text
Historical ticket
      ↓
Embedding model
      ↓
768-dimensional vector
      ↓
PostgreSQL / pgvector
```

For a new incident:

```text
New incident
      ↓
Embedding model
      ↓
Query vector
      ↓
Cosine distance
      ↓
5 closest historical tickets
```

An HNSW index using `vector_cosine_ops` is used to support efficient similarity search.

Local development uses Ollama with `nomic-embed-text`.

The production environment uses OpenAI `text-embedding-3-small` configured to generate 768-dimensional vectors.

Embeddings stored in each environment are generated using the same embedding model used for queries in that environment.

---

## AI response

The AI returns a structured response instead of unrestricted free text:

```json
{
  "probableCause": "...",
  "suggestedSolution": "...",
  "recommendedTeam": "..."
}
```

The backend deserializes the response into a strongly typed .NET model before returning it to the frontend.

---

## Technology stack

**Backend**
- C#
- .NET 8
- ASP.NET Core Web API
- Entity Framework Core
- Repository pattern

**Database**
- PostgreSQL
- Npgsql
- pgvector
- HNSW vector index

**AI**
- OpenAI API – cloud environment
- `text-embedding-3-small` – cloud embeddings
- Ollama – local development
- `nomic-embed-text` – local embeddings
- OllamaSharp

**Frontend**
- HTML
- CSS
- JavaScript

**Infrastructure**
- Docker
- Render
- GitHub

---

## Data and security

The PostgreSQL database contains more than 16,000 historical English-language IT support tickets.

The AI model does not have direct access to the database. Database access is controlled by the .NET application through the repository layer.

```text
PostgreSQL
    ↓
Repository
    ↓
Selected ticket fields
    ↓
AI Data Sanitizer
    ↓
AI
```

Only selected historical ticket fields are used as AI context.

Email addresses and phone numbers are removed from historical data before it is sent to the AI.

API keys and database credentials are provided through environment configuration and are not stored in source code.

---

## Local and cloud architecture

```text
LOCAL
.NET API
   ↓
Ollama
   ├── qwen3
   └── nomic-embed-text
   ↓
PostgreSQL + pgvector

PRODUCTION
.NET API
   ↓
OpenAI
   ├── LLM
   └── text-embedding-3-small
   ↓
PostgreSQL + pgvector
```

The application uses abstractions such as `IAiClient` and `IEmbeddingClient`, allowing different AI providers to be used without changing the main application logic.

---

## Planned development

- Complete production embedding generation for the historical dataset
- Agent tools and automated actions
- Application and AI activity logging
- Agent action audit history
- Human approval for selected actions
- Improved prompt-injection protection
- Improved frontend output sanitization
- Rate limiting and authentication
- Improved error handling
- Automated tests
- Observability and monitoring

The goal is to evolve the application from an AI-assisted incident analyzer into an AI Incident Agent supporting L1 incident triage.

---

## Project status

The production application and structured OpenAI analysis are operational.

Semantic search with pgvector, 768-dimensional embeddings and HNSW indexing is implemented and tested locally.

The production database schema is prepared for semantic search, while generation of OpenAI embeddings for the historical production dataset is the remaining deployment step.

```text
Frontend
   ↓
ASP.NET Core
   ↓
Semantic Retrieval / PostgreSQL + pgvector
   ↓
OpenAI
   ↓
Structured Analysis
   ↓
Frontend
```

The application is under active development.

## Source code

The source code is maintained in a private repository.

**Source code can be shared on request for recruitment or technical evaluation purposes.**
