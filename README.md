# Daycare AI Platform

An AI-native childcare operations platform that replaces manual admin work with a voice-first pipeline. Teachers send voice memos via WhatsApp, the system structures them into logged events, admins perform the final review, and parents receive daily narrative reports. 

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

Local Development Setup
Backend Environment:

Bash
python -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt
Database Migrations:

Bash
alembic upgrade head
Run FastAPI Server:

Bash
cd backend
uvicorn main:app --reload
Run Frontend Clients:

Bash
# Admin & Teacher Console
cd frontend/console
npm install && npm run dev

# Parent Portal
cd frontend/parent
npm install && npm run dev
