### Overview
The Daycare AI Platform is an AI-native childcare operations system designed to replace manual administrative workflows (like tapping through apps or hiring virtual assistants) with a voice-first, high-accuracy pipeline. The platform allows daycare teachers to log daily events using voice memos and photos via WhatsApp. This unstructured data is processed, structured, validated, and pushed through a multi-tier review system before generating cohesive, AI-driven daily narrative reports for parents.

### The Problem It Solves
Independent daycare directors ($300K–$750K ARR) rely on consumer-grade tools (like Brightwheel) which require teachers to manually log events on tablets (disrupting the classroom). Alternatively, they hire offshore VAs to review photos and create logs, which is expensive, slow, and prone to context loss. 

This platform acts as an automated, highly-accurate operations layer. It eliminates the manual data entry bottleneck, guarantees structured data via LLM extraction, and uses a human-in-the-loop (HITL) model to prevent AI hallucinations from ever reaching parents.

## How It Works
Voice Intake: Teachers send voice memos via a WhatsApp Business bot.

AI Extraction: The backend processes the audio using Whisper and AssemblyAI, then uses GPT-4o to extract strictly-typed JSON that matches our database schemas.

## Three-Tier Review:

High-confidence events are surfaced for a single-tap teacher approval.

Low-confidence, billing, or incident events (needs_review: true) automatically flag for Director review.

No data reaches parents without a human in the loop.

Parent Narratives: Real-time event updates are published to a live feed, and at the end of the day, parents receive a cohesive AI-generated summary of their child's day via a magic-link web portal.

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
