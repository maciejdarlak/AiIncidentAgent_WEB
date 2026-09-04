# AI Incident Agent

AI-assisted IT incident analysis application built with ASP.NET Core, PostgreSQL and OpenAI.

The application analyzes new IT incidents using historical support tickets and AI-generated recommendations.

## Live Demo

The application is publicly available at:

https://darlak-ai.onrender.com

<img width="1920" height="1200" alt="1" src="https://github.com/user-attachments/assets/9547e8ac-2710-46c0-99f3-6a4d8b1583de" />

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
- Search for historical incidents based on the new incident title
- Retrieval of up to 5 matching historical tickets
- Limited data projection before sending historical data to AI
- Basic PII sanitization for email addresses and phone numbers
- AI analysis using historical incidents as additional context
- Probable cause recommendation
- Suggested solution
- Recommended support team
- Local AI support through Ollama
- Cloud AI support through OpenAI API
- Docker deployment
- Public deployment on Render

---

## How it works

```text
User
  ↓
Web Frontend
  ↓
ASP.NET Core API
  ↓
Ticket Search Service
  ↓
Repository
  ↓
PostgreSQL
  ↓
Up to 5 matching historical tickets
  ↓
Data Sanitization
  ↓
OpenAI
  ↓
Incident Analysis
  ↓
Web Frontend
```

When a new incident is submitted, the application searches the historical ticket database.

Currently, the incident title is compared with the `Subject` and `Body` fields of historical tickets using case-insensitive text matching.

Up to 5 matching tickets are retrieved from PostgreSQL. Only selected information is passed to the AI:

- Subject
- Description
- Historical solution
- Queue

The data is sanitized before being included in the AI prompt.

The new incident remains the primary source of information, while historical tickets are used only as additional reference material.

---

## Current search limitation

The current historical ticket search is intentionally simple.

It uses text matching and does not yet calculate semantic similarity between incidents.

The next step is to improve filtering and ranking and introduce semantic search using embeddings, allowing the application to find related incidents based on meaning rather than only matching text.

---

## Technology stack

**Backend**
- C#
- .NET 8
- ASP.NET Core Web API
- Entity Framework Core

**Database**
- PostgreSQL
- Npgsql

**AI**
- OpenAI API – cloud environment
- Ollama – local development
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

Email addresses and phone numbers are removed from the historical data before it is sent to the AI.

---

## Planned development

- Improved historical ticket filtering and ranking
- Semantic search using embeddings
- Better selection of relevant AI context
- Structured AI responses
- Agent tools and automated actions
- Application and AI activity logging
- Agent action audit history
- Human approval for selected actions
- Improved error handling and security
- Automated tests

The goal is to evolve the application from an AI-assisted incident analyzer into an AI Incident Agent supporting L1 incident triage.

---

## Project status

The core end-to-end flow is operational:

```text
Frontend → ASP.NET Core → PostgreSQL → OpenAI → Frontend
```

The application is under active development.

## Source code

The source code is maintained in a private repository.

**Source code can be shared on request for recruitment or technical evaluation purposes.**
