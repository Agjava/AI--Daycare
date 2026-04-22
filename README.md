### Overview
The Daycare AI Platform is an AI-native childcare operations system designed to replace manual administrative workflows (like tapping through apps or hiring virtual assistants) with a voice-first, high-accuracy pipeline. The platform allows daycare teachers to log daily events using voice memos and photos via WhatsApp. This unstructured data is processed, structured, validated, and pushed through a multi-tier review system before generating cohesive, AI-driven daily narrative reports for parents.

### The Problem It Solves
Independent daycare directors ($300K–$750K ARR) rely on consumer-grade tools (like Brightwheel) which require teachers to manually log events on tablets (disrupting the classroom). Alternatively, they hire offshore VAs to review photos and create logs, which is expensive, slow, and prone to context loss. 

This platform acts as an automated, highly-accurate operations layer. It eliminates the manual data entry bottleneck, guarantees structured data via LLM extraction, and uses a human-in-the-loop (HITL) model to prevent AI hallucinations from ever reaching parents.

## How It Works
Voice Intake: Teachers send voice memos via a WhatsApp Business bot.

AI Extraction: The backend processes the audio using Whisper and AssemblyAI, then uses GPT-4o to extract strictly-typed JSON that matches our database schemas.

### System Architecture & Pipeline

The system is a standalone platform with full stack ownership (no dependency on legacy tools). 

1. **Ingestion Layer (WhatsApp Business API via Twilio)**
   - Teachers interact exclusively via a WhatsApp Bot. They send voice memos (`.ogg`/`.mp4` up to 90s) and photo batches.
   - Natural language context (e.g., "Jason just ate all his mac and cheese") serves as the raw input.

2. **AI Extraction & Structuring Layer (FastAPI Backend)**
   - **Speech-to-Text (STT):** OpenAI Whisper (V1) is used for rapid transcription. (Future iterations plan to use AssemblyAI for better proper noun resolution).
   - **Entity Extraction (LLM):** GPT-4o processes the transcript at `temperature=0` to ensure deterministic outputs. It extracts data strictly matching predefined Pydantic schemas (JSON structure).
   - **Strict Validation:** The extracted JSON maps to strictly-typed entities (e.g., `MEAL`, `NAP`, `DIAPER`, `BILLING_LATE_PICKUP`). Parsing failures or ambiguous events default to `needs_review: true`.

3. **Three-Tier Human-in-the-Loop (HITL) Review System**
   - **Teacher Review:** High-confidence parsed events are sent back for a single-tap approval.
   - **Director Review:** Low-confidence, incident-related, or billing-related events are automatically flagged for the Director Console.
   - **Parent View:** Parents *only* see data after explicit human approval.

4. **Presentation & Narrative Generation**
   - At the end of the day, an automated pipeline groups all approved events per child.
   - GPT-4o synthesizes these discrete events into a warm, personalized 120-250 word narrative paragraph.
   - Parents access these reports via a Next.js-based web portal using magic links (no app install required).

5. **Billing & Monetization Module**
   - Voice memos containing billing intents (e.g., "Marcus was picked up 45 mins late") trigger a specific billing schema extraction.
   - Admins approve the extraction, generating a PDF invoice processed via Stripe Connect.

## Tech Stack
Backend: Python 3.11+, FastAPI, PostgreSQL

Frontend: React PWA (Admin/Teacher Console), Next.js/Vercel (Parent Portal)

AI/ML Pipelines: OpenAI Whisper API (V1), AssemblyAI Universal-2 (V1.5), GPT-4o (structured data extraction)

Integrations: Twilio (WhatsApp API), Stripe Connect (Billing module)

Infrastructure: Designed for Railway / Fly.io deployments

## Core Architectural Guardrails
Strict Validation: All LLM extraction calls run at temperature=0 and are validated against rigorous Pydantic models before database insertion. Parsing failures or ambiguous events automatically default to needs_review: true.

Multi-tenant Isolation: Every table and transactional query is strictly scoped by center_id at the database level.

Privacy First: COPPA compliant by design. Parent and admin access is scoped strictly via short-lived magic links and OTPs (no passwords). Facial recognition is strictly prohibited to avoid biometric compliance triggers.

Human Approval: NEVER auto-send any event to parents without explicit human approval (teacher OR director).

## Local Development Setup

1. **Backend Environment**:
   ```bash
   python -m venv venv
   source venv/bin/activate
   pip install -r backend/requirements.txt
   ```

2. **Database Migrations**:
   ```bash
   alembic upgrade head
   ```

3. **Run FastAPI Server**:
   ```bash
   cd backend
   uvicorn main:app --reload
   ```

4. **Run Frontend Clients**:
   ```bash
   # Admin & Teacher Console
   cd frontend/console
   npm install && npm run dev
   
   # Parent Portal
   cd frontend/parent
   npm install && npm run dev
   ```

## Documentation

Full product requirements and structural overviews are detailed in `docs/PRD.md` and `docs/legal_PRD.md`.
