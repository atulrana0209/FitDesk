# FitFront — AI Front Desk for Boutique Gyms

[![FitFront CI & Evaluation Suite](https://github.com/atulrana0209/FitDesk/actions/workflows/ci.yml/badge.svg)](https://github.com/atulrana0209/FitDesk/actions/workflows/ci.yml)
[![Python 3.12 | 3.13 | 3.14](https://img.shields.io/badge/Python-3.12%20%7C%203.13%20%7C%203.14-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.141.1-009688.svg?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1.svg?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![React 19](https://img.shields.io/badge/React-19.0-61DAFB.svg?logo=react&logoColor=black)](https://react.dev/)
[![Ollama](https://img.shields.io/badge/Ollama-Local%20LLM-black.svg?logo=ollama&logoColor=white)](https://ollama.com/)

**FitFront** is an open-source, multi-tenant AI voice and chat front desk platform purpose-built for boutique gyms, fitness studios, and martial arts academies. It automates trial session bookings, class reservations, dynamic waitlist management, client recognition, and two-way SMS follow-ups—operating 24/7 over the phone via natural voice AI or in-browser through an interactive chat interface.

A single FitFront deployment functions as a centralized SaaS engine serving multiple independent fitness studios. Each gym operates within an isolated partition with its own branding, trainers, class catalogs, business hours, holiday calendars, pricing structures, and custom knowledge base stored in PostgreSQL.

---

## Table of Contents

- [A. Project Overview](#a-project-overview)
- [B. Features](#b-features)
- [C. Technology Stack](#c-technology-stack)
- [D. System Architecture](#d-system-architecture)
- [E. Project Structure](#e-project-structure)
- [F. Complete Setup and Installation Guide](#f-complete-setup-and-installation-guide)
- [G. Environment Variables and Configuration](#g-environment-variables-and-configuration)
- [H. How to Run and Use the Project](#h-how-to-run-and-use-the-project)
- [I. Internal Workflow: Step-by-Step](#i-internal-workflow-step-by-step)
- [J. API Documentation](#j-api-documentation)
- [K. Database Design and Data Models](#k-database-design-and-data-models)
- [L. Important Implementation Details](#l-important-implementation-details)
- [M. Error Handling and Troubleshooting](#m-error-handling-and-troubleshooting)
- [N. Testing and Quality Assurance](#n-testing-and-quality-assurance)
- [O. Security Considerations](#o-security-considerations)
- [P. Deployment and Production Considerations](#p-deployment-and-production-considerations)
- [Q. Future Improvements](#q-future-improvements)
- [R. Contributing](#r-contributing)
- [S. License and Acknowledgements](#s-license-and-acknowledgements)

---

## A. Project Overview

### The Problem
Boutique fitness studios lose up to **30–40% of inbound sales opportunities** when prospect calls ring through to voicemail during peak training hours, classes, or after closing. Front desk staff are frequently split between welcoming members, sanitizing equipment, and answering repetitive phone inquiries regarding class times, trial session pricing, and trainer availability.

### The Solution
FitFront provides an automated conversational front desk assistant that answers calls instantaneously, converses naturally, checks real-time coach calendars, enforces slot capacity limits, registers callers for classes or waitlists, and dispatches SMS confirmations within seconds.

### Target Users & Practical Use Cases
- **Boutique Gym Owners & Operators**: Centralize front desk scheduling, reduce missed-call churn, and eliminate front-desk staffing overhead during early mornings (5:00 AM) and late evenings.
- **Gym Clients & Inbound Callers**: Call anytime to check trainer openings, book a complimentary trial workout, reschedule a class, or ask studio policy and parking questions.
- **SaaS Providers**: Deploy a single multi-tenant instance to serve hundreds of gym clients with administrative oversight, usage metering, and automated isolation.

### Current Implementation Status & Limitations
- **Implemented**: Full FastAPI backend, async PostgreSQL persistence with SQLAlchemy 2.0, dual-engine scheduling (Google Calendar OAuth + Native SQL engine), Ollama local LLM integration (`qwen3:8b`, `qwen2.5:3b`) and Google Gemini fallback, Bolna voice telephony integration, Twilio SMS service, React 19 management dashboard, and automated evaluation and tenant-isolation benchmarks.
- **Current Limitations**:
  - Live phone audio requires either a public URL (via `localtunnel`/`ngrok`) or cloud deployment with Gemini/Bolna credentials.
  - Payment processing and membership subscription checkout links are handled via custom SMS/knowledge responses; direct credit card processing via Stripe is currently scheduled for future release.

---

## B. Features

### 1. Conversational Voice & Chat AI Front Desk
- **Interaction**: Inbound voice call via Bolna telephony or browser-based chat via the **Test Front Desk** (`/chat`).
- **Internal Mechanism**: Inbound user utterances stream into `process_message()` inside [`backend/services/llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py). The engine dynamically renders a tenant-scoped system prompt incorporating studio rules, business hours, and upcoming holidays, invoking registered platform tools in a two-step function-calling loop.
- **Responsible Modules**: [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py), [`backend/prompts/agent_prompt.py`](file:///d:/fitfront/backend/prompts/agent_prompt.py), [`backend/tools/platform.py`](file:///d:/fitfront/backend/tools/platform.py).

### 2. Trainer-Aware Dual Scheduling Engine
- **Interaction**: Callers request bookings for specific coaches, class categories (HIIT, Boxing, Yoga), or trial sessions.
- **Internal Mechanism**: Checks [`backend/services/calendar_service.py`](file:///d:/fitfront/backend/services/calendar_service.py). If the gym connected a Google Calendar, it queries Google Free/Busy APIs; otherwise, it delegates to [`backend/services/native_scheduling.py`](file:///d:/fitfront/backend/services/native_scheduling.py), verifying coach working hours overrides, 7-day trial windows, and parallel slot capacity limits.
- **Responsible Modules**: [`calendar_service.py`](file:///d:/fitfront/backend/services/calendar_service.py), [`native_scheduling.py`](file:///d:/fitfront/backend/services/native_scheduling.py), [`provider_service.py`](file:///d:/fitfront/backend/services/provider_service.py).

### 3. Dynamic Waitlist Queue & Auto-Promotion
- **Interaction**: When a class is fully booked, the agent offers to place the caller on the waitlist. If an opening occurs (via cancellation or capacity expansion), the system resolves the queue.
- **Internal Mechanism**: Tracks queue priority and time preferences in `waitlist_entries`. When [`cancel_appointment()`](file:///d:/fitfront/backend/routes/appointments.py) is triggered, [`check_waitlist_for_opening()`](file:///d:/fitfront/backend/services/waitlist_service.py) automatically identifies the highest-priority eligible candidate and sends an SMS notification or auto-promotes the booking.
- **Responsible Modules**: [`backend/services/waitlist_service.py`](file:///d:/fitfront/backend/services/waitlist_service.py), [`backend/routes/waitlist.py`](file:///d:/fitfront/backend/routes/waitlist.py).

### 4. Automated SMS Lifecycle & Review Solicitation
- **Interaction**: Automated text messages sent to callers without human intervention.
- **Internal Mechanism**: Outbound SMS messages are resolved per-tenant via [`_resolve_twilio()`](file:///d:/fitfront/backend/services/sms_service.py) to guarantee that Studio A's credentials and caller-ID never intermingle with Studio B. A background task in [`backend/services/reminder_service.py`](file:///d:/fitfront/backend/services/reminder_service.py) wakes every 60 seconds to scan for upcoming sessions (sending 2h reminders) and attended visits (sending Google Review links after 24h).
- **Responsible Modules**: [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py), [`reminder_service.py`](file:///d:/fitfront/backend/services/reminder_service.py).

### 5. Multi-Tenant Studio Administration & CRM
- **Interaction**: Studio managers log into the React dashboard (`/login`) to manage trainer schedules, view call audio transcripts, monitor appointment status history, and update studio knowledge base entries.
- **Internal Mechanism**: All API calls validate JWT bearer claims. Database queries enforce strict `where(Tenant.id == current_user.id)` boundaries.
- **Responsible Modules**: [`backend/routes/`](file:///d:/fitfront/backend/routes/), [`frontend/src/components/`](file:///d:/fitfront/frontend/src/components/).

---

## C. Technology Stack

| Layer | Technology | Role in FitFront | Justification |
| :--- | :--- | :--- | :--- |
| **Backend Framework** | Python 3.12–3.14 + FastAPI | Core REST API & Webhook Service | Native asynchronous request handling (`async`/`await`), automatic OpenAPI generation, and sub-millisecond response latency. |
| **Database** | PostgreSQL 15 | Relational & JSONB Data Store | ACID compliance, JSONB document storage for flexible per-tenant knowledge bases, and partial unique indexes for multi-tenant isolation. |
| **Database Driver & ORM** | `asyncpg` + SQLAlchemy 2.0 | Async Database Interaction | High-performance non-blocking connection pooling and type-safe query construction. |
| **Local LLM Engine** | Ollama (`qwen3:8b`, `qwen2.5:3b`) | Conversational Brain (Self-Hosted) | Zero-cost, privacy-preserving on-premise execution with OpenAI-compatible tool-calling endpoints. |
| **Cloud LLM Engine** | Google Gemini 2.0 Flash (`google-genai`) | Low-Latency Cloud LLM | Low-latency inference (<400ms token-to-speech turnaround) required for live voice calls over public phone networks. |
| **Voice Telephony** | Bolna | Voice AI Orchestration | Handles SIP trunking, WebRTC audio streaming, Deepgram transcription, and ElevenLabs/Cartesia voice synthesis. |
| **Messaging** | Twilio REST API (via HTTPX) | Outbound & Inbound SMS | Reliable SMS delivery without the overhead of heavy blocking SDKs. |
| **Frontend Framework** | React 19 + Vite | Studio Owner & Admin Dashboard | Fast component rendering, instant HMR development, and reactive state management. |
| **Styling** | Tailwind CSS + Lucide Icons | Dashboard UI Design System | Clean, responsive dark/light mode interface tailored for desktop and mobile tablets. |
| **Testing** | Pytest + Pytest-Asyncio | Evaluation & Isolation Testing | Deterministic workflow benchmarks and automated cross-tenant security verification. |
| **Deployment** | Docker & Docker Compose | Containerized Services | Single-command PostgreSQL provisioning with persistent named volumes and healthcheck probes. |

---

## D. System Architecture

### Component Architecture Diagram

```mermaid
flowchart TD
    subgraph Clients["Inbound Channels"]
        Call[Inbound Phone Call]
        BrowserChat[Browser Chat / Test Front Desk]
        StudioAdmin[Gym Owner Dashboard]
    end

    subgraph Edge["Gateway & Ingestion"]
        Bolna[Bolna Voice AI / Webhook]
        FastAPI[FastAPI Gateway :8000]
    end

    subgraph Security["Auth & Tenant Guard"]
        JWT[JWT Authentication]
        TenantResolver[Tenant Context Resolver]
        PrivacyGate[Caller Privacy Filter]
    end

    subgraph Engine["Conversational Core"]
        LLMService[LLM Service Router]
        PromptBuilder[System Prompt Engine]
        ToolRegistry[Platform Tool Dispatcher]
        Ollama[Local Ollama: qwen3:8b]
        Gemini[Google Gemini 2.0 Flash]
    end

    subgraph Services["Domain Services"]
        Sched[Calendar & Scheduling Engine]
        WaitlistSvc[Waitlist Queue Engine]
        CallerCRM[Caller Recognition & CRM]
        SMSSvc[Twilio SMS Service]
        ReminderSvc[Background Reminder Daemon]
    end

    subgraph Storage["Persistence Layer"]
        Postgres[(PostgreSQL 15 :5433)]
    end

    subgraph External["External Services"]
        GCal[Google Calendar API]
        TwilioAPI[Twilio Messaging API]
    end

    Call --> Bolna
    Bolna -->|POST /api/llm| FastAPI
    BrowserChat -->|POST /api/chat| FastAPI
    StudioAdmin -->|REST API Requests| FastAPI

    FastAPI --> JWT
    FastAPI --> TenantResolver
    TenantResolver --> PrivacyGate
    PrivacyGate --> LLMService

    LLMService <-->|Prompt & History| Ollama
    LLMService <-->|Prompt & History| Gemini
    PromptBuilder --> LLMService

    LLMService -->|Execute Tools| ToolRegistry
    ToolRegistry --> Sched
    ToolRegistry --> WaitlistSvc
    ToolRegistry --> CallerCRM
    ToolRegistry --> SMSSvc

    Sched <--> GCal
    SMSSvc <--> TwilioAPI
    ReminderSvc --> SMSSvc

    Sched --> Postgres
    WaitlistSvc --> Postgres
    CallerCRM --> Postgres
    ReminderSvc --> Postgres
    TenantResolver --> Postgres
```

### Architecture Component Responsibilities
1. **Gateway & Route Controllers** ([`backend/routes/`](file:///d:/fitfront/backend/routes/)): Receives incoming HTTP requests, terminates SSL, enforces CORS, validates schemas using Pydantic, and routes requests to appropriate services.
2. **Tenant Context Resolver** ([`backend/services/tenant_service.py`](file:///d:/fitfront/backend/services/tenant_service.py)): Inspects request subdomains, query parameters, or JWT claims to build an immutable [`TenantContext`](file:///d:/fitfront/backend/services/tenant_service.py) populated with timezone, business hours, and credentials.
3. **Conversational LLM Service** ([`backend/services/llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py)): Manages active conversation sessions in memory (`sessions` dict keyed by `call_id`), formats system prompts with dynamic time/holiday injection, and drives the function-calling loop.
4. **Platform Tool Registry** ([`backend/tools/registry.py`](file:///d:/fitfront/backend/tools/registry.py) & [`backend/tools/platform.py`](file:///d:/fitfront/backend/tools/platform.py)): Translates LLM tool invocations (`book_appointment`, `get_available_slots`, `add_to_waitlist`) into internal service operations with strict parameter verification.
5. **Persistence Engine** ([`backend/database.py`](file:///d:/fitfront/backend/database.py)): Manages async connection pools, model schemas, automatic column migrations, and transactional rollbacks.

---

## E. Project Structure

```
fitfront/
├── .github/
│   └── workflows/
│       └── ci.yml                 # GitHub Actions CI workflow (Docker, pytest, benchmarks)
├── backend/
│   ├── config.py                  # Pydantic BaseSettings environment configuration
│   ├── database.py                # Async SQLAlchemy engine, sessionmaker, table auto-migrations
│   ├── defaults.py                # Platform default constants, standard timezones, type slugifiers
│   ├── main.py                    # FastAPI application entrypoint, lifespan startup/shutdown
│   ├── models/                    # SQLAlchemy ORM declarative models
│   │   ├── appointment.py         # Appointment schema, status enum, caller relationships
│   │   ├── call.py                # Call interaction logs, transcript JSONB, outcomes
│   │   ├── caller.py              # Multi-tenant caller/client records with composite unique keys
│   │   ├── provider.py            # Gym trainer/coach profiles, specialties, capacity overrides
│   │   ├── tenant.py              # Tenant accounts, approval status, hours & knowledge base
│   │   └── waitlist.py            # Waitlist queue entries, time preference windows, statuses
│   ├── prompts/
│   │   └── agent_prompt.py        # Dynamic tenant-scoped system prompt constructor
│   ├── routes/                    # FastAPI route modules grouped by business resource
│   │   ├── appointments.py        # /api/appointments: List, patch, cancel, reschedule
│   │   ├── auth.py                # /api/auth: Login, registration, token refresh
│   │   ├── chat.py                # /api/chat: Local browser-based chat streaming endpoint
│   │   ├── llm_proxy.py           # /api/llm: Bolna/voice phone integration webhook
│   │   ├── providers.py           # /api/providers: Trainer CRUD and availability configuration
│   │   └── waitlist.py            # /api/waitlist: Queue management, promote, conflict check
│   ├── services/                  # Business logic services
│   │   ├── calendar_service.py    # High-level scheduling dispatcher (Native vs. Google)
│   │   ├── caller_service.py      # E.164 phone normalizer and CRM profile upsert
│   │   ├── google_calendar.py     # Google Calendar OAuth2 and Free/Busy client
│   │   ├── llm_service.py         # Stateful conversation manager (Ollama & Gemini)
│   │   ├── native_scheduling.py   # Internal SQL slot calculation and capacity engine
│   │   ├── provider_service.py    # Trainer lookup and update operations (tenant-scoped)
│   │   ├── reminder_service.py    # Background 60s daemon for 2h reminders and review requests
│   │   ├── sms_service.py         # Outbound Twilio SMS dispatcher and message logger
│   │   └── waitlist_service.py    # Waitlist addition, opening detection, and promotion
│   ├── tests/
│   │   └── evaluation/            # Dedicated evaluation and security benchmark suite
│   │       ├── scenarios.json     # 54 version-controlled benchmark evaluation test cases
│   │       ├── test_agent_evaluation.py # 54-scenario deterministic agent benchmark
│   │       └── test_tenant_isolation.py  # 22-scenario multi-tenant adversarial security suite
│   └── tools/                     # Agent tool definitions and dispatchers
│       ├── platform.py            # Core tools: book, cancel, reschedule, waitlist, office info
│       └── registry.py            # Tool provider registration and session context injection
├── docs/                          # Architecture guides and integration specifications
│   ├── FULL_STACK_SETUP.md        # Production phone, SMS, and calendar onboarding guide
│   └── explainer/                 # Comprehensive High-Level and Low-Level Design deep dives
├── frontend/                      # React 19 + Vite dashboard
│   ├── src/
│   │   ├── components/            # UI components and management pages
│   │   │   ├── AgentConfig.jsx    # Voice persona, greeting, and escalation rules editor
│   │   │   ├── AppointmentManager.jsx # Class schedule, calendar filters, and cancellations
│   │   │   ├── CallerCRM.jsx      # Client profile inspection, booking history, tags
│   │   │   ├── Dashboard.jsx      # Key metrics, real-time call logs, occupancy charts
│   │   │   ├── LocalChat.jsx      # "Test Front Desk" in-browser voice/chat testing tool
│   │   │   ├── TrainerManager.jsx # Coach roster, specialties, and slot capacities
│   │   │   └── WaitlistView.jsx   # Waitlist queue overview and one-click promotion
│   │   ├── App.jsx                # Router configuration and protected route layout
│   │   └── main.jsx               # React DOM entrypoint and global CSS imports
│   ├── package.json               # Frontend dependencies and npm scripts
│   └── vite.config.js             # Vite build configuration and API reverse-proxy rules
├── reports/
│   └── fitfront-evaluation/       # Benchmark reports and metrics
│       ├── environment.md         # Host OS, Python, and Ollama execution environment
│       ├── evaluation_results.json # Machine-readable 54-scenario benchmark results
│       ├── evaluation_summary.md  # Full technical evaluation summary with KaTeX formulas
│       ├── tenant_isolation_results.json # Machine-readable 22-scenario security results
│       └── test_execution_summary.txt # Raw execution logs and reproduction commands
├── docker-compose.yml             # PostgreSQL 15 container definition on port 5433
├── requirements.txt               # Backend Python dependencies
├── seed_data.py                   # Demo gym generator (Iron & Ivy Fitness Studio)
└── start.sh                       # Unified startup script with automatic localtunnel capture
```

---

## F. Complete Setup and Installation Guide

### Prerequisites
- **Operating System**: Linux, macOS, or Windows 10/11
- **Python**: Version `3.12`, `3.13`, or `3.14`
- **Node.js**: Version `18.x` or higher (with `npm`)
- **Docker**: Docker Desktop or Docker Engine (for PostgreSQL)
- **Ollama** (Optional for local LLM mode): Installed from [ollama.com](https://ollama.com)

---

### Step-by-Step Installation

#### 1. Clone the Repository
```bash
git clone https://github.com/atulrana0209/FitDesk.git
cd FitDesk
```

#### 2. Configure Environment Variables
Copy the example environment file:
```bash
# Linux / macOS / Windows PowerShell
cp .env.example .env
```
*Note: The default `.env.example` is preconfigured for the local demo (Ollama LLM, simulated SMS, and Docker PostgreSQL on port 5433).*

#### 3. Start the PostgreSQL Database
FitFront uses PostgreSQL 15. The provided Docker Compose maps container port `5432` to host port **`5433`** to avoid colliding with any pre-existing local PostgreSQL instances:
```bash
docker compose up -d postgres
```
Verify the container is healthy:
```bash
docker ps --filter "name=scheduler_ai_db"
```

#### 4. Setup Python Virtual Environment & Dependencies

**Linux / macOS:**
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

**Windows PowerShell:**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
# Ensure timezone database is available on Windows Python:
pip install tzdata
```

#### 5. Seed the Demo Database
Run the seed script to create **Iron & Ivy Fitness Studio**, 3 trainers, 5 clients, past call logs, and sample appointments:
```bash
# Linux / macOS
python3 seed_data.py

# Windows PowerShell
python seed_data.py
```

#### 6. Start the Backend API Server
```bash
# Linux / macOS
python3 -m uvicorn backend.main:app --reload --port 8000

# Windows PowerShell
python -m uvicorn backend.main:app --reload --port 8000
```
Verify the backend is live by opening [http://localhost:8000/docs](http://localhost:8000/docs).

#### 7. Setup and Start the Frontend Dashboard
Open a separate terminal window:
```bash
cd frontend
npm install
npm run dev
```
Open [http://localhost:5173](http://localhost:5173) in your web browser.

#### 8. Verify the Installation
1. Navigate to [http://localhost:5173/login](http://localhost:5173/login).
2. Log in using the default platform admin credentials:
   - **Email**: `admin@fitfront.local`
   - **Password**: `changeme`
3. Click **"Test Front Desk"** in the sidebar to open the in-browser agent (`/chat`).
4. Type: *"What are your hours on Monday and do you offer trial sessions?"*
5. The agent will retrieve hours and trial session details from the database and answer in real-time.

---

## G. Environment Variables and Configuration

| Variable Name | Required | Default / Safe Placeholder | Consumed In | Purpose |
| :--- | :---: | :--- | :--- | :--- |
| `DATABASE_URL` | **Yes** | `postgresql+asyncpg://scheduler_user:scheduler_pass@localhost:5433/scheduler_ai` | [`database.py`](file:///d:/fitfront/backend/database.py) | Async PostgreSQL connection string. |
| `ADMIN_EMAIL` | **Yes** | `admin@fitfront.local` | [`main.py`](file:///d:/fitfront/backend/main.py) | Initial superadmin account created on startup. |
| `ADMIN_PASSWORD` | **Yes** | `changeme` | [`main.py`](file:///d:/fitfront/backend/main.py) | Initial superadmin password. |
| `DEMO_MODE` | No | `true` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py), [`calendar_service.py`](file:///d:/fitfront/backend/services/calendar_service.py) | When `true`, SMS and calendar mutations are simulated in logs. |
| `LOCAL_CHAT_MODE` | No | `true` | [`main.py`](file:///d:/fitfront/backend/main.py), [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py) | When `true`, enables browser `/chat` mode without public webhooks. |
| `LLM_PROVIDER` | **Yes** | `ollama` (or `gemini`) | [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py) | Selects the active LLM engine. |
| `OLLAMA_BASE_URL` | If Ollama | `http://localhost:11434` | [`config.py`](file:///d:/fitfront/backend/config.py), [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py) | Local Ollama REST endpoint. |
| `OLLAMA_MODEL` | If Ollama | `qwen3:8b` (or `qwen2.5:3b`) | [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py) | Ollama model tag. |
| `GEMINI_API_KEY` | If Gemini | `AIzaSy...` | [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py) | Google AI Studio Gemini API key. |
| `GEMINI_MODEL` | If Gemini | `gemini-2.0-flash` | [`llm_service.py`](file:///d:/fitfront/backend/services/llm_service.py) | Cloud Gemini model identifier. |
| `SERVER_BASE_URL` | No | `http://localhost:8000` | [`main.py`](file:///d:/fitfront/backend/main.py) | Public server URL used for webhook callbacks. |
| `TWILIO_ACCOUNT_SID` | If SMS | `ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py) | Platform Twilio Account SID. |
| `TWILIO_AUTH_TOKEN` | If SMS | `auth_token_here` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py) | Platform Twilio Auth Token. |
| `TWILIO_PHONE_NUMBER`| If SMS | `+15551234567` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py) | Default outbound SMS phone number. |
| `FEATURE_TWILIO_ENABLED`| No | `true` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py) | Global feature flag for SMS dispatch. |
| `BOLNA_API_KEY` | If Voice | `bolna_api_key` | [`bolna_service.py`](file:///d:/fitfront/backend/services/bolna_service.py) | Bolna voice telephony API key. |
| `BOLNA_AGENT_ID` | If Voice | `agent_uuid` | [`bolna_service.py`](file:///d:/fitfront/backend/services/bolna_service.py) | Pre-configured Bolna Agent UUID. |
| `ESCALATION_PHONE_NUMBER`| No | `+15559990000` | [`sms_service.py`](file:///d:/fitfront/backend/services/sms_service.py), [`platform.py`](file:///d:/fitfront/backend/tools/platform.py) | Staff phone number receiving escalation SMS alerts. |
| `ESCALATION_TRANSFER_NUMBER`| No| `+15559990001` | [`llm_proxy.py`](file:///d:/fitfront/backend/routes/llm_proxy.py) | Phone number for live voice transfer handoff. |
| `JWT_SECRET` | No | `fitfront-jwt-insecure-secret-key...` | [`auth_service.py`](file:///d:/fitfront/backend/services/auth_service.py) | Secret signing key for JWT tokens. |

---

## H. How to Run and Use the Project

### Service Management Commands

```bash
# Start PostgreSQL container
docker compose up -d postgres

# Stop PostgreSQL container
docker compose down

# Start FastAPI backend
uvicorn backend.main:app --reload --port 8000

# Start Frontend dev server
cd frontend && npm run dev

# Run full evaluation benchmark suite
python backend/tests/evaluation/test_agent_evaluation.py
python backend/tests/evaluation/test_tenant_isolation.py

# Run all automated tests via pytest
pytest -v
```

### Health Check & API Verification
Verify backend health and database connectivity:
```bash
curl -X GET http://localhost:8000/health
```
**Expected Response:**
```json
{
  "status": "healthy",
  "app": "FitFront",
  "version": "1.0.0",
  "database": "connected"
}
```

### Interactive User Workflows

1. **Studio Registration**:
   - Visit [http://localhost:5173/register](http://localhost:5173/register).
   - Enter business name, owner email, phone, and timezone.
   - Status defaults to `PENDING_APPROVAL`.
2. **Admin Approval**:
   - Log into `admin@fitfront.local`.
   - Navigate to **"Manage Tenants"** (`/admin/tenants`).
   - Click **"Approve"** on the pending gym.
3. **Studio Front Desk Testing**:
   - Log in as the studio owner.
   - Click **"Test Front Desk"** (`/chat`).
   - Interact with the voice/text agent to book a workout, inquire about gym parking, or test cancellation logic.

---

## I. Internal Workflow: Step-by-Step

### 1. Inbound Call & Message Booking Lifecycle

```mermaid
sequenceDiagram
    autonumber
    actor Caller as Caller / User
    participant Gateway as FastAPI (/api/llm or /api/chat)
    participant Auth as Tenant Context Resolver
    participant LLM as LLM Service (Ollama / Gemini)
    participant Tools as PlatformToolProvider
    participant Sched as Scheduling Engine
    participant DB as PostgreSQL 15
    participant SMS as Twilio SMS Service

    Caller->>Gateway: "Hi, I'd like to book a Trial Session next Monday at 10 AM"
    Gateway->>Auth: resolve_by_slug() or resolve_by_phone()
    Auth->>DB: Query Tenant, BusinessHours, Holidays, Trainers
    DB-->>Auth: TenantContext
    Auth-->>Gateway: Injected TenantContext

    Gateway->>LLM: process_message(call_id, message, tenant_ctx)
    LLM->>LLM: Render System Prompt with Tenant Data
    LLM-->>Gateway: Tool Call: get_available_slots(date="2026-11-02", type="trial_session")
    
    Gateway->>Tools: call_tool("get_available_slots", args)
    Tools->>Sched: get_provider_aware_slots()
    Sched->>DB: Check Trainer working hours, capacity, existing bookings
    DB-->>Sched: Open slot: 2026-11-02T10:00:00
    Sched-->>Tools: Available
    Tools-->>LLM: {"ok": true, "available_slots": ["2026-11-02T10:00:00"]}

    LLM-->>Gateway: Tool Call: book_appointment(client_name="Alex", client_phone="+15551234567", slot_time="2026-11-02T10:00:00")
    Gateway->>Tools: call_tool("book_appointment", args)
    Tools->>DB: Insert Appointment(status="CONFIRMED")
    DB-->>Tools: Appointment ID a1a1a1a1-...
    Tools->>SMS: send_confirmation(client_phone, scheduled_at, tenant_ctx)
    SMS-->>Caller: SMS Confirmation: "You're booked for Trial Session on Mon, Nov 2 at 10:00 AM!"
    Tools-->>LLM: {"ok": true, "summary_for_assistant": "Booking confirmed"}

    LLM-->>Gateway: "You are all set, Alex! I've booked your Trial Session for Monday, November 2nd at 10:00 AM."
    Gateway-->>Caller: Spoken / Displayed Response
```

### Execution Trace:
1. **Trigger**: An inbound voice utterance reaches `/api/llm` or a chat message reaches `/api/chat`.
2. **Context Resolution**: [`tenant_service.resolve_by_id()`](file:///d:/fitfront/backend/services/tenant_service.py) loads the tenant's timezone, appointment types, business hours, and knowledge base.
3. **Privacy Gate**: [`_enforce_caller_phone()`](file:///d:/fitfront/backend/services/llm_service.py) verifies that the caller cannot access another client's appointments.
4. **Tool Selection**: The LLM outputs a function call (`book_appointment`).
5. **Tool Execution**: [`PlatformToolProvider.call_tool()`](file:///d:/fitfront/backend/tools/platform.py) validates arguments, computes slot availability, commits the `Appointment` record in PostgreSQL, and dispatches confirmation SMS via [`sms_service.send_confirmation()`](file:///d:/fitfront/backend/services/sms_service.py).
6. **Final Synthesis**: The LLM receives the tool result JSON and synthesizes a friendly, spoken voice response.

---

## J. API Documentation

### Authentication
All protected administrative routes require an HTTP Bearer JWT token in the `Authorization` header:
```http
Authorization: Bearer <your_jwt_token>
```
Tokens are acquired via `POST /api/auth/login`.

---

### Core Endpoint Catalog

| Method | Endpoint | Description | Auth Required | Request Body / Query Params |
| :--- | :--- | :--- | :---: | :--- |
| `POST` | `/api/auth/register` | Register a new fitness studio | No | `{ "business_name": "...", "owner_email": "...", "password": "..." }` |
| `POST` | `/api/auth/login` | Authenticate and obtain JWT | No | `{ "email": "...", "password": "..." }` |
| `GET` | `/api/appointments` | List studio appointments (filtered) | **Yes** | `status`, `date_from`, `date_to`, `include_test` |
| `POST` | `/api/appointments/{id}/cancel` | Manually cancel an appointment | **Yes** | Path: `id` (UUID) |
| `PATCH` | `/api/appointments/{id}` | Update appointment status/notes | **Yes** | `{ "status": "ATTENDED", "notes": "Great session" }` |
| `GET` | `/api/providers` | List gym trainers & capacities | **Yes** | `active_only=true` |
| `POST` | `/api/providers` | Create a new trainer | **Yes** | `{ "name": "Marcus", "slot_capacity": 1, "specialty": "HIIT" }` |
| `GET` | `/api/waitlist` | List waitlist entries | **Yes** | `status=WAITING` |
| `POST` | `/api/waitlist/{id}/promote` | Promote waitlisted caller to class | **Yes** | `{ "scheduled_at": "2026-11-02T10:00:00", "force": false }` |
| `POST` | `/api/chat` | Browser test agent chat stream | No | `{ "call_id": "...", "message": "...", "tenant_slug": "..." }` |
| `POST` | `/api/llm` | Bolna voice telephony webhook | No | OpenAI-compatible chat completion payload |

---

### Sample cURL Invocations

#### 1. Authenticate Studio Owner
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "owner@ironandivy.fit",
    "password": "Password123!"
  }'
```
**Response (200 OK):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
  "token_type": "bearer",
  "tenant": {
    "id": "11111111-1111-1111-1111-111111111111",
    "slug": "iron-and-ivy-fitness",
    "business_name": "Iron & Ivy Fitness Studio",
    "is_admin": false
  }
}
```

#### 2. Query Studio Appointments
```bash
curl -X GET "http://localhost:8000/api/appointments?include_test=false" \
  -H "Authorization: Bearer <your_jwt_token>"
```

#### 3. Cancel an Appointment
```bash
curl -X POST "http://localhost:8000/api/appointments/a1a1a1a1-a1a1-a1a1-a1a1-a1a1a1a1a1a1/cancel" \
  -H "Authorization: Bearer <your_jwt_token>"
```

---

## K. Database Design and Data Models

FitFront utilizes PostgreSQL with relational foreign-key integrity and JSONB extensibility.

### Entity Relationship Diagram

```mermaid
erDiagram
    TENANTS ||--o{ TRAINERS : "employs"
    TENANTS ||--o{ CALLERS : "manages"
    TENANTS ||--o{ CALLS : "handles"
    TENANTS ||--o{ WAITLIST_ENTRIES : "queues"
    CALLERS ||--o{ APPOINTMENTS : "books"
    TRAINERS ||--o{ APPOINTMENTS : "trains"
    CALLS ||--o{ APPOINTMENTS : "originates"
    APPOINTMENTS ||--o{ APPOINTMENT_STATUS_HISTORY : "tracks"
    CALLERS ||--o{ SMS_MESSAGES : "receives"

    TENANTS {
        uuid id PK
        string slug UK
        string business_name
        string business_phone
        string timezone
        jsonb appointment_types
        jsonb business_hours
        jsonb holidays
        jsonb knowledge_base
        string status
        boolean demo_mode
    }

    TRAINERS {
        uuid id PK
        uuid tenant_id FK
        string name
        string title
        string specialty
        int slot_capacity
        jsonb business_hours_override
        boolean is_active
    }

    CALLERS {
        uuid id PK
        uuid tenant_id FK
        string name
        string phone
        string email
        boolean is_test
    }

    APPOINTMENTS {
        uuid id PK
        uuid caller_id FK
        uuid provider_id FK
        uuid call_id FK
        string client_name
        string client_phone
        string appointment_type
        datetime scheduled_at
        string status
        int duration_minutes
    }

    WAITLIST_ENTRIES {
        uuid id PK
        uuid tenant_id FK
        uuid caller_id FK
        string appointment_type
        date preferred_date
        string preferred_time_start
        string preferred_time_end
        string status
        int priority
    }

    CALLS {
        uuid id PK
        uuid tenant_id FK
        uuid caller_id FK
        string vapi_call_id UK
        string outcome
        jsonb transcript
        datetime started_at
        datetime ended_at
    }

    APPOINTMENT_STATUS_HISTORY {
        uuid id PK
        uuid appointment_id FK
        string old_status
        string new_status
        string changed_by
        string note
        datetime created_at
    }

    SMS_MESSAGES {
        uuid id PK
        uuid caller_id FK
        string direction
        string from_number
        string to_number
        text body
        datetime created_at
    }
```

### Key Data Integrity Constraints
1. **Multi-Tenant Composite Uniqueness on Callers**: `UniqueConstraint("tenant_id", "phone", name="uq_callers_tenant_phone")` allows a client to use the same mobile phone number across different fitness studios without data bleeding.
2. **Double-Booking Partial Unique Index**: A custom partial index prevents overlapping confirmed bookings for single-capacity personal trainers while permitting multi-capacity group classes.
3. **Database Scoping**: All mutations and queries on `Trainer`, `WaitlistEntry`, `Appointment`, and `Caller` verify `tenant_id` to enforce partition security.

---

## L. Important Implementation Details

### 1. Multi-Tenant Caller Privacy Gate
When an agent is handling an active phone call, callers may attempt to access or modify reservations for friends or family. The [`_enforce_caller_phone()`](file:///d:/fitfront/backend/services/llm_service.py) guard checks incoming tool arguments:
- If a caller asks to cancel, lookup, or modify an appointment using a phone number that does not match the active caller-ID, the tool invocation is immediately intercepted and rejected before hitting the database.

### 2. Timezone Normalization Architecture
All database timestamps (`scheduled_at`, `created_at`, `started_at`) are strictly stored in **UTC**. Before formatting SMS notifications or feeding slot availability to the voice assistant, timestamps are dynamically localized using the gym's specific IANA timezone (e.g. `America/Chicago`, `America/New_York`) to ensure the caller hears *"Monday at 9:00 AM CST"* rather than confusing UTC offsets.

### 3. Agent Tool Looping Guardrail
Small local LLMs (e.g., 3B parameters) can occasionally get trapped in repetitive tool invocation loops when processing multi-faceted inquiries. To guarantee stability:
- FitFront implements a hard `MAX_TOOL_ROUNDS = 5` execution threshold.
- Greeting heuristics automatically suppress tool calls on casual greetings (*"Hi"*, *"Good morning"*), preventing hallucinated transfers.

---

## M. Error Handling and Troubleshooting

| Error Symptom | Root Cause | Diagnosis Steps | Practical Resolution |
| :--- | :--- | :--- | :--- |
| `ZoneInfoNotFoundError: 'No time zone found with key America/Chicago'` | Missing OS timezone database on Windows Python. | Occurs when calling `sms_service._to_local_time()`. | Run `pip install tzdata` inside the `.venv`. |
| `ConnectionRefusedError: [Errno 111 / WinError 1225]` | PostgreSQL container is stopped or port mismatch. | Run `docker ps` to verify container status. | Run `docker compose up -d postgres`. Verify `.env` uses port **`5433`** (`localhost:5433`). |
| `AttributeError: 'NoneType' object has no attribute 'replace'` | Missing or uninitialized `TENANT_ID`. | Check FastAPI application startup logs. | Run `python seed_data.py` to populate the initial tenant and admin record. |
| `Ollama dial tcp 127.0.0.1:11434: connectex: forbidden` | Ollama daemon is not running. | Run `curl http://127.0.0.1:11434/api/tags`. | Start Ollama desktop application or run `ollama serve`. |
| `TypeError: 'Query' object has no attribute 'replace'` | Calling FastAPI endpoints directly without default parameter bindings in unit tests. | Inspect test method invocation signatures. | Provide explicit arguments (e.g., `include_test=False`) when invoking route handlers in tests. |

---

## N. Testing and Quality Assurance

FitFront incorporates a deterministic evaluation and security benchmark suite located in [`backend/tests/evaluation/`](file:///d:/fitfront/backend/tests/evaluation/):

```
backend/tests/evaluation/
├── scenarios.json             # 54 version-controlled functional scenarios
├── test_agent_evaluation.py   # Deterministic workflow benchmark runner
└── test_tenant_isolation.py    # 22-scenario adversarial security benchmark
```

### Running the Evaluation Suite

```bash
# 1. Run deterministic workflow evaluation (54 scenarios)
python backend/tests/evaluation/test_agent_evaluation.py

# 2. Run multi-tenant isolation security tests (22 scenarios)
python backend/tests/evaluation/test_tenant_isolation.py

# 3. Run full suite via pytest
pytest backend/tests/evaluation/ -v
```

### Verified Test Results
- **Deterministic Agent Benchmark**: **54 / 54 Passed (100.0%)** across 17 categories (trial bookings, slot capacity enforcement, holiday closures, waitlist queueing, ambiguous input resolution, and tool failure recovery).
- **Tenant Isolation Security**: **22 / 22 Passed (100.0%)** confirming complete defense against cross-tenant read, update, delete, and IDOR vectors.
- **Full Pytest Execution**: **24 passed in 17.18s**.

---

## O. Security Considerations

- **Strict Multi-Tenant Scoping**: All database queries for trainers, appointments, callers, and waitlist entries explicitly enforce `WHERE tenant_id == current_user.id`.
- **JWT Signature & Expiration**: Authentication tokens are signed using HMAC-SHA256 with user UUID claims, expiring after 7 days.
- **IDOR Protections**: Route handlers for `PATCH /api/appointments/{id}`, `POST /api/appointments/{id}/cancel`, `PATCH /api/providers/{id}`, and `POST /api/waitlist/{id}/promote` verify that the target object's `tenant_id` matches the authenticated caller before executing mutations.
- **Credential Masking**: Application startup routines log integration readiness while masking sensitive tokens (`ACxxxxxxxx...`).

---

## P. Deployment and Production Considerations

### Production Architecture Checklist
1. **Database**: Managed PostgreSQL (AWS RDS, Neon, Supabase) with connection pooling (`pgbouncer`).
2. **Reverse Proxy**: Nginx or Cloudflare terminating SSL and forwarding traffic to Uvicorn.
3. **Process Management**: Run Uvicorn via Gunicorn with multiple worker processes:
   ```bash
   gunicorn backend.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
   ```
4. **Cloud LLM Selection**: Set `LLM_PROVIDER=gemini` and configure `GEMINI_API_KEY` for live telephony to achieve sub-second voice response turnarounds.
5. **Twilio Webhook Configuration**:
   - Inbound SMS webhook URL: `https://your-domain.com/api/sms/webhook`
6. **Bolna Telephony Configuration**:
   - Custom LLM endpoint: `https://your-domain.com/api/llm`
   - Caller lookup endpoint: `https://your-domain.com/api/bolna/caller-lookup`

---

## Q. Future Improvements

- [ ] **Stripe Embedded Checkout**: Direct automated SMS links for membership deposit collection.
- [ ] **ClassPass / Mindbody Integration**: Bi-directional sync with legacy boutique gym studio management software.
- [ ] **WhatsApp Business API Channel**: Expand conversational booking agent to WhatsApp voice and text.
- [ ] **WebRTC Voice Widget**: Embed conversational voice agent directly on studio marketing websites.
- [ ] **Multi-Location Gym Clusters**: Support for regional gym chains with unified brand voice and localized studio routing.

---

## R. Contributing

Contributions are welcomed! Follow these steps to contribute:

1. **Fork the Repository** on GitHub.
2. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/trainer-skills-filter
   ```
3. **Implement Changes & Add Tests**:
   - Add new test scenarios in [`backend/tests/evaluation/scenarios.json`](file:///d:/fitfront/backend/tests/evaluation/scenarios.json).
   - Ensure all tests pass: `pytest -v`.
4. **Commit Your Changes**:
   ```bash
   git commit -m "feat: add skill-level filtering to trainer slot matching"
   ```
5. **Push to Your Fork & Open a Pull Request**.

---

## S. License and Acknowledgements

### License
This project is licensed under the **MIT License**. See the `LICENSE` file for details.

### Acknowledgements
- [FastAPI](https://fastapi.tiangolo.com/) by Sebastián Ramírez.
- [Ollama](https://ollama.com/) for local LLM inference tooling.
- [SQLAlchemy](https://www.sqlalchemy.org/) & [asyncpg](https://github.com/MagicStack/asyncpg) for asynchronous database management.
- [Bolna](https://www.bolna.dev/) for voice AI orchestration.
- [Tailwind CSS](https://tailwindcss.com/) & [Lucide React](https://lucide.dev/) for dashboard aesthetics.
