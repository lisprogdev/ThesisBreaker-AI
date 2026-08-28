# TECH.md — ThesisBreaker AI

> **Project:** ThesisBreaker AI  
> **Track:** Sectors Hackathon 2026 — Track 1: AI Agents & Assistants  
> **Core concept:** AI research agent that challenges an investor's thesis by finding supporting evidence, counter-evidence, contradictions, assumptions, and research blind spots using Sectors data as a core data source.

---

## 1. Purpose of This Document

This document is the **technical specification and technology guide for vibe coding** ThesisBreaker AI.

Use this document as the reference for AI coding assistants and development agents. The implementation should follow the architecture, stack, boundaries, conventions, and priorities defined here.

The goal is to build a **real working MVP**, not a static mockup.

---

# 2. Core Technology Stack

## Main Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                        REACT FRONTEND                        │
│                                                             │
│  Dashboard • Thesis Editor • Agent Progress • Reports       │
│  Evidence Explorer • History                                │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTPS / REST API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       LARAVEL BACKEND                        │
│                                                             │
│  Authentication                                              │
│  Thesis Management                                           │
│  AI Agent Orchestration                                      │
│  Sectors Data Integration                                    │
│  Report Generation                                           │
│  Queue & Background Jobs                                     │
└───────────────┬───────────────────────────────┬─────────────┘
                │                               │
                ▼                               ▼
┌────────────────────────┐          ┌────────────────────────┐
│     MYSQL DATABASE      │          │    EXTERNAL SERVICES   │
│                        │          │                        │
│ Managed with phpMyAdmin│          │ Sectors API / MCP      │
│                        │          │ LLM API                │
└────────────────────────┘          └────────────────────────┘
```

---

# 3. Required Primary Technologies

## 3.1 Backend

### Laravel

**Language:** PHP 8.3+  
**Framework:** Laravel 12 or latest stable version compatible with the project

Laravel is responsible for:

- REST API
- Authentication
- User management
- Thesis CRUD
- Agent orchestration
- AI provider integration
- Sectors API or MCP integration
- Background jobs
- Report persistence
- Validation
- Rate limiting
- Error handling
- Database access

### Required Laravel Components

- Laravel API resources
- Laravel Sanctum for authentication
- Eloquent ORM
- Form Request validation
- Jobs and Queues
- Events and Listeners where useful
- HTTP Client for external APIs
- Cache abstraction
- Logging

---

## 3.2 Frontend

### React

**React:** latest stable version  
**Build tool:** Vite  
**Language:** TypeScript

React is responsible for:

- User interface
- Dashboard
- Thesis creation
- Conversational or guided research workflow
- Real-time-like agent progress display
- Thesis report visualization
- Evidence comparison
- Research history

### Recommended Frontend Libraries

#### Routing

```text
react-router-dom
```

#### API / Server State

```text
@tanstack/react-query
axios
```

#### Forms

```text
react-hook-form
zod
@hookform/resolvers
```

#### Styling

```text
Tailwind CSS
```

#### Icons

```text
lucide-react
```

#### Charts

```text
recharts
```

Use charts only where they genuinely improve understanding.

#### Utility

```text
clsx
tailwind-merge
```

---

## 3.3 Database

### MySQL

Use MySQL 8+ as the primary relational database.

### phpMyAdmin

phpMyAdmin is used for:

- Viewing database tables
- Inspecting records
- Debugging development data
- Manually checking relationships when necessary

**Important:** phpMyAdmin is a database management interface, not the database itself. The application connects directly to MySQL through Laravel.

---

# 4. AI and Agent Architecture

ThesisBreaker AI must not be implemented as a simple chatbot with one prompt.

The product should use a **multi-step agent workflow**.

## Core Agent Flow

```text
USER WRITES A THESIS
        │
        ▼
┌───────────────────┐
│ THESIS PARSER     │
│ AGENT             │
└─────────┬─────────┘
          │
          ▼
Extract:
• Main claim
• Assumptions
• Companies
• Sectors
• Metrics
• Time horizon
          │
          ▼
┌───────────────────┐
│ RESEARCH PLANNER  │
│ AGENT             │
└─────────┬─────────┘
          │
          ▼
Decides what information is required
          │
          ▼
┌─────────────────────────────────────┐
│        SECTORS DATA RETRIEVAL       │
│      Sectors REST API / MCP         │
└──────────────┬──────────────────────┘
               │
      ┌────────┴────────┐
      ▼                 ▼
┌─────────────┐   ┌─────────────┐
│ BULL AGENT  │   │ BEAR AGENT  │
│             │   │             │
│ Find data   │   │ Find data   │
│ supporting  │   │ challenging │
│ the thesis  │   │ the thesis  │
└──────┬──────┘   └──────┬──────┘
       └────────┬────────┘
                ▼
      ┌───────────────────┐
      │ CONTRADICTION     │
      │ AGENT             │
      └─────────┬─────────┘
                ▼
      ┌───────────────────┐
      │ BLIND SPOT        │
      │ ANALYSIS          │
      └─────────┬─────────┘
                ▼
      ┌───────────────────┐
      │ FINAL SYNTHESIS   │
      │ REPORT            │
      └───────────────────┘
```

---

# 5. Sectors Integration

Sectors data is a **core dependency**, not a decorative API call.

If Sectors data is removed, ThesisBreaker AI should lose its core research and evidence-verification functionality.

## Integration Layer

Create a dedicated Laravel service layer:

```text
app/Services/Sectors/
├── SectorsClient.php
├── SectorsResearchService.php
├── SectorsCompanyService.php
├── SectorsFinancialService.php
├── SectorsMarketService.php
└── SectorsDataNormalizer.php
```

## Responsibilities

The Sectors integration layer should:

1. Receive research requirements from the agent workflow.
2. Select the appropriate Sectors endpoint or MCP tool.
3. Fetch relevant market or company data.
4. Normalize external responses.
5. Return structured data to the research agents.
6. Preserve source metadata for evidence traceability.
7. Handle API errors and rate limits.
8. Cache repeated requests where appropriate.

## Never Do This

Do not:

- Make one token API call only to satisfy the hackathon requirement.
- Hide Sectors usage behind fake or hardcoded data.
- Use Sectors data only for a decorative chart.
- Present fabricated data as Sectors data.

---

# 6. LLM Integration

The LLM provider must be abstracted behind a service interface.

Recommended structure:

```text
app/Contracts/AI/
└── AIProviderInterface.php

app/Services/AI/
├── AIOrchestrator.php
├── LLMService.php
├── PromptBuilder.php
├── StructuredOutputService.php
└── Agents/
    ├── ThesisParserAgent.php
    ├── ResearchPlannerAgent.php
    ├── BullResearchAgent.php
    ├── BearResearchAgent.php
    ├── ContradictionAgent.php
    ├── BlindSpotAgent.php
    └── SynthesisAgent.php
```

The actual LLM provider must be configurable through environment variables.

Example:

```env
AI_PROVIDER=openai
AI_MODEL=...
AI_API_KEY=...
```

Do not hardcode secrets.

---

# 7. Structured AI Outputs

AI responses should use structured JSON internally whenever possible.

Do not rely entirely on unstructured prose between agents.

Example thesis parser output:

```json
{
  "main_claim": "Example thesis",
  "assumptions": [
    {
      "statement": "Example assumption",
      "importance": "high"
    }
  ],
  "entities": {
    "companies": [],
    "sectors": []
  },
  "metrics_needed": [],
  "time_horizon": "medium_term"
}
```

Laravel should validate and persist normalized results.

---

# 8. Backend Project Structure

Recommended Laravel structure:

```text
backend/
├── app/
│   ├── Contracts/
│   │   ├── AI/
│   │   └── Sectors/
│   │
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── AuthController.php
│   │   │   │   ├── ThesisController.php
│   │   │   │   ├── ResearchController.php
│   │   │   │   └── ReportController.php
│   │   ├── Requests/
│   │   └── Resources/
│   │
│   ├── Jobs/
│   │   ├── RunThesisAnalysisJob.php
│   │   └── GenerateReportJob.php
│   │
│   ├── Models/
│   │   ├── User.php
│   │   ├── Thesis.php
│   │   ├── ResearchRun.php
│   │   ├── AgentRun.php
│   │   ├── Evidence.php
│   │   ├── Contradiction.php
│   │   └── ThesisReport.php
│   │
│   ├── Services/
│   │   ├── AI/
│   │   ├── Sectors/
│   │   └── Research/
│   │
│   └── Support/
│
├── database/
│   ├── migrations/
│   ├── factories/
│   └── seeders/
│
├── routes/
│   └── api.php
│
└── tests/
```

---

# 9. Frontend Project Structure

Recommended React structure:

```text
frontend/
├── src/
│   ├── api/
│   │   ├── client.ts
│   │   ├── auth.ts
│   │   ├── theses.ts
│   │   └── research.ts
│   │
│   ├── components/
│   │   ├── common/
│   │   ├── layout/
│   │   ├── thesis/
│   │   ├── research/
│   │   └── report/
│   │
│   ├── features/
│   │   ├── auth/
│   │   ├── thesis/
│   │   ├── research/
│   │   └── reports/
│   │
│   ├── hooks/
│   ├── lib/
│   ├── pages/
│   │   ├── LandingPage.tsx
│   │   ├── LoginPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── NewThesisPage.tsx
│   │   ├── ResearchPage.tsx
│   │   └── ReportPage.tsx
│   │
│   ├── routes/
│   ├── types/
│   ├── App.tsx
│   └── main.tsx
│
└── vite.config.ts
```

---

# 10. Database Design

## users

```text
id
name
email
password
created_at
updated_at
```

## theses

```text
id
user_id
title
original_text
status
created_at
updated_at
```

Suggested status:

```text
draft
processing
completed
failed
```

## research_runs

Each execution of the AI workflow.

```text
id
thesis_id
status
started_at
completed_at
error_message
created_at
updated_at
```

## agent_runs

Track each agent's execution.

```text
id
research_run_id
agent_name
status
input_data
output_data
started_at
completed_at
created_at
updated_at
```

## evidence

```text
id
research_run_id
type
claim
summary
source_type
source_reference
source_data
confidence
created_at
updated_at
```

Evidence type:

```text
supporting
contradicting
neutral
```

## contradictions

```text
id
research_run_id
title
description
severity
related_assumption
created_at
updated_at
```

## thesis_reports

```text
id
thesis_id
research_run_id
summary
verdict
confidence_score
report_data
created_at
updated_at
```

---

# 11. Authentication

Use:

```text
Laravel Sanctum
```

For the MVP:

- Register
- Login
- Logout
- Get current user

The React frontend should store authentication safely according to the selected Sanctum approach.

Do not implement unnecessary enterprise-level authentication complexity.

---

# 12. API Design

Base URL:

```text
/api/v1
```

## Authentication

```text
POST   /auth/register
POST   /auth/login
POST   /auth/logout
GET    /auth/me
```

## Theses

```text
GET    /theses
POST   /theses
GET    /theses/{id}
PATCH  /theses/{id}
DELETE /theses/{id}
```

## Research

```text
POST /theses/{id}/analyze
GET  /research-runs/{id}
GET  /research-runs/{id}/progress
```

## Reports

```text
GET /theses/{id}/report
```

---

# 13. Research Workflow State

The frontend must clearly show that the AI is performing a real multi-step workflow.

Example:

```text
✓ Understanding your thesis
✓ Extracting assumptions
◉ Planning research
○ Searching Sectors data
○ Building the bull case
○ Challenging the thesis
○ Detecting contradictions
○ Finding blind spots
○ Writing the final report
```

This is important for:

- Product usability
- Video demonstration
- Communicating technical depth
- Making the agent workflow understandable

Do not fake progress. Progress states should be connected to actual backend job or workflow status.

---

# 14. MVP Pages

## 14.1 Landing Page

Purpose:

Explain what ThesisBreaker AI does.

Main message:

> Don't just find reasons you're right. Find reasons you might be wrong.

Sections:

- Hero
- How it works
- Multi-agent workflow
- Example thesis analysis
- Disclaimer
- CTA

---

## 14.2 Authentication

Simple:

- Sign up
- Sign in

Do not overdesign this area.

---

## 14.3 Dashboard

Show:

- Recent theses
- Analysis status
- Number of reports
- Quick action: New Thesis

---

## 14.4 New Thesis Page

The main product entry point.

Fields:

```text
Title
Your market thesis
Optional context
```

Primary action:

```text
Break My Thesis
```

The UI should explain that the AI will actively search for evidence both for and against the thesis.

---

## 14.5 Research Progress Page

Show the actual agent workflow.

Recommended layout:

```text
LEFT
• Thesis
• Extracted assumptions

CENTER
• Live workflow / agent progress

RIGHT
• Data and evidence discovered
```

---

## 14.6 Final Report Page

Must clearly separate:

### Supporting Evidence

```text
🟢 Evidence supporting the thesis
```

### Counter-Evidence

```text
🔴 Evidence challenging the thesis
```

### Contradictions

```text
⚡ Internal or data-based contradictions
```

### Blind Spots

```text
🟡 Important areas not sufficiently considered
```

### Final Analysis

A neutral synthesis.

Do not use language such as:

```text
BUY
SELL
STRONG BUY
GUARANTEED PROFIT
```

Use:

```text
Research signal
Evidence strength
Areas requiring further investigation
Conditions that could weaken the thesis
```

---

# 15. UI and Design Direction

## Product Style

ThesisBreaker AI should feel:

- Premium
- Modern
- Analytical
- Fintech
- AI-native
- Focused
- Professional

Avoid:

- Generic chatbot UI
- Excessive gradients
- Crypto casino aesthetics
- Too many neon colors
- Overloaded dashboards
- Unnecessary animations

## Suggested Visual Direction

Primary:

```text
Near black / deep navy
```

Accent:

```text
Electric blue
Cyan
```

Supporting semantic states:

```text
Green  = supporting evidence
Red    = counter-evidence
Amber  = warning / blind spot
Blue   = neutral / AI processing
```

Use the existing ThesisBreaker AI logo provided for branding.

---

# 16. Environment Variables

Backend `.env` should contain secrets only.

Example:

```env
APP_NAME="ThesisBreaker AI"
APP_ENV=local
APP_DEBUG=true

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=thesisbreaker_ai
DB_USERNAME=root
DB_PASSWORD=

SECTORS_API_KEY=
SECTORS_BASE_URL=

AI_PROVIDER=
AI_API_KEY=
AI_MODEL=

QUEUE_CONNECTION=database
CACHE_STORE=database
```

Never commit `.env`.

Commit:

```text
.env.example
```

---

# 17. Queue Strategy

AI analysis can take longer than normal API requests.

Use Laravel Jobs.

Suggested flow:

```text
POST /theses/{id}/analyze
        │
        ▼
Create ResearchRun
        │
        ▼
Dispatch RunThesisAnalysisJob
        │
        ▼
API immediately returns research_run_id
        │
        ▼
React polls progress endpoint
        │
        ▼
Agents complete
        │
        ▼
Final report saved
```

For the MVP, polling is sufficient.

Do not add WebSockets unless genuinely necessary.

---

# 18. Caching

Cache expensive or repeated external data requests where appropriate.

Potential targets:

- Repeated company lookups
- Market reference data
- Repeated Sectors API requests

Do not cache user-specific thesis results incorrectly.

---

# 19. Error Handling

Every major external integration must handle:

- Invalid API response
- Timeout
- Rate limit
- Missing data
- LLM failure
- Queue failure

The UI should show useful messages such as:

```text
We couldn't complete this research run.
Please try again.
```

Do not expose:

- API keys
- Stack traces
- Internal prompts
- Sensitive server details

---

# 20. Security Rules

Required:

- Validate every request
- Use Laravel authentication middleware
- Never expose Sectors API keys to React
- Never expose LLM API keys to React
- Keep external API calls in Laravel
- Use `.env`
- Use `.gitignore`
- Sanitize and validate inputs
- Rate-limit sensitive endpoints

---

# 21. Financial Disclaimer

The application must include a visible disclaimer.

Suggested text:

> ThesisBreaker AI is an information and research tool. It does not provide financial, investment, legal, or tax advice. All outputs are generated for research purposes and should not be treated as a recommendation to buy, sell, or hold any financial instrument.

This should appear:

- In the application
- On the report page
- Where appropriate in the demo

---

# 22. Development Priorities

Build in this order.

## Phase 1 — Foundation

```text
1. Initialize repository
2. Create Laravel backend
3. Create React + Vite frontend
4. Configure MySQL
5. Connect phpMyAdmin for development
6. Configure API communication
7. Configure environment variables
```

## Phase 2 — Core Product

```text
1. Authentication
2. Thesis CRUD
3. Sectors integration
4. Thesis parser agent
5. Research planner
6. Bull agent
7. Bear agent
8. Final synthesis
```

## Phase 3 — ThesisBreaker Differentiation

```text
1. Contradiction detection
2. Blind spot analysis
3. Evidence traceability
4. Research progress UI
5. Final Thesis Health Report
```

## Phase 4 — Hackathon Polish

```text
1. Error handling
2. Loading states
3. Empty states
4. Demo seed data
5. README
6. Screenshots
7. API key cleanup
8. Final repository review
```

---

# 23. Suggested Repository Layout

Use one repository:

```text
thesisbreaker-ai/
├── README.md
├── TECH.md
├── .gitignore
├── .env.example
├── backend/
├── frontend/
└── docs/
    ├── architecture.md
    ├── api.md
    └── demo-flow.md
```

For the hackathon, this makes it easier for judges to inspect the complete project and commit history.

---

# 24. Vibe Coding Rules

When using AI coding assistants:

1. Build the real functionality, not fake screens.
2. Do not generate an entire uncontrolled application in one step.
3. Implement one module at a time.
4. Test every integration before moving forward.
5. Keep Sectors integration visible and traceable.
6. Prefer simple architecture over unnecessary microservices.
7. Do not hardcode fake market analysis as if it were live data.
8. Do not expose API keys.
9. Keep commits meaningful.
10. Ensure every commit is created within the official hackathon build period.

---

# 25. Definition of Done for MVP

The MVP is considered ready when a user can:

```text
1. Create an account
2. Log in
3. Write a market thesis
4. Start an AI analysis
5. See the agent workflow progress
6. Have the system use Sectors data as a core source
7. View supporting evidence
8. View counter-evidence
9. View contradictions
10. View blind spots
11. Receive a neutral final research synthesis
12. View evidence and source context
```

---

# 26. Final Technical Principle

## Build the smallest real product that proves the idea.

The most important experience is:

```text
USER HAS A BELIEF
        ↓
USER WRITES A THESIS
        ↓
AI BREAKS IT INTO ASSUMPTIONS
        ↓
AI RESEARCHES WITH SECTORS DATA
        ↓
AI LOOKS FOR REASONS THE USER MAY BE RIGHT
        ↓
AI ACTIVELY LOOKS FOR REASONS THE USER MAY BE WRONG
        ↓
AI EXPOSES CONTRADICTIONS AND BLIND SPOTS
        ↓
USER RECEIVES A TRACEABLE, NEUTRAL RESEARCH REPORT
```

**ThesisBreaker AI is not an investment advisor.**

**It is an AI research adversary designed to challenge assumptions before the user becomes too confident in them.**
