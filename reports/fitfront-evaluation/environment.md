# FitFront Evaluation Benchmark Environment & Setup

## 1. Host Operating System & Hardware
- **OS Platform**: Microsoft Windows 11
- **Shell**: PowerShell
- **Architecture**: x86_64

## 2. Python Runtime & Virtual Environment
- **Python Version**: Python 3.14.6
- **Virtual Environment Path**: `.venv/`
- **Key Python Packages**:
  - `fastapi` == 0.141.1
  - `sqlalchemy` == 2.0.51
  - `asyncpg` == 0.31.0
  - `pydantic` == 2.13.4
  - `openai` == 2.52.1
  - `google-genai` == 2.16.0
  - `pyjwt` == 2.13.0
  - `httpx` == 0.28.1
  - `pytest` == 9.1.1
  - `pytest-asyncio` == 1.4.0
  - `pytest-timeout` == 2.4.0
  - `tzdata` == 2026.5 (installed to support `America/Chicago` on Windows Python 3.14)

## 3. LLM Configuration & Local Infrastructure
- **LLM Engine**: Ollama CLI `0.40.6`
- **Ollama Endpoint**: `http://localhost:11434`
- **Installed Models**:
  - `qwen3:8b` (5.2 GB, Q4_K_M quantization) — configured in `.env` as default
  - `qwen2.5:3b` (1.9 GB) — verified for low-latency CPU inference
- **Cloud Provider Support**: Google Gemini 2.0 Flash (`google-genai` SDK, fallback)
- **External Integration Guards**:
  - External SMS calls (`Twilio API`) guarded via `is_test=True` and `demo_mode=True` mocks.
  - External Google Calendar mutations guarded via mock adapters.
  - No live outbound telephony or external billing invocations during evaluation.

## 4. Multi-Tenant Database Architecture
- **DBMS**: PostgreSQL with `asyncpg` driver
- **Target URL**: `postgresql+asyncpg://scheduler_user:scheduler_pass@localhost:5433/scheduler_ai`
- **Isolation Harness**:
  - Deterministic evaluation uses mock async sessions with transaction rollbacks to eliminate execution order dependence and ensure 100% offline reproducible CI runs.
  - Continuous integration spins up a clean PostgreSQL 15 container with auto-migrated schema.

## 5. Dataset Versioning
- **Dataset**: `backend/tests/evaluation/scenarios.json`
- **Version**: `v1.0`
- **Scenario Count**: 54 distinct agent workflow scenarios
- **Security Scenarios**: 22 cross-tenant authorization and isolation attack vectors
