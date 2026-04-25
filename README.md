# SOFHealth — Multimodal Clinical Assistant

> One photo. Every answer. In under 5 seconds.

![Status](https://img.shields.io/badge/Status-Beta-yellow)
![Stack](https://img.shields.io/badge/Stack-n8n%20%7C%20Groq%20%7C%20Telegram-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## What Is This?

SOFHealth is an AI-powered clinical decision support tool built for health professionals. A pharmacist or physician sends a single photo of a prescription, a voice note, or a text query — and gets back a structured drug analysis in seconds.

No manual database searching. No switching between apps. No friction.

**What it returns:**
- Drug identification — brand name and generic equivalent
- Drug-Drug Interaction (DDI) check — with severity classification
- Dosage verification — against standard therapeutic windows
- Patient-specific contraindications — when patient context is provided
- Source citation — every clinical claim grounded in data

---

## The Problem It Solves

Health professionals in fast-paced clinical environments face a recurring friction point: the time required to verify drug information manually is inconsistent with the speed at which clinical decisions must be made.

Existing tools like Micromedex and Epocrates are reliable but slow, expensive, and not built for multimodal or hands-free input. SOFHealth is.

---

## Current State (v1.0 Beta)

v1.0 is a Telegram bot orchestrated on n8n. It is the foundation — not the final product.

```
User sends photo / voice / text
        ↓
Telegram Bot receives message
        ↓
n8n workflow routes by message type
        ↓
Groq AI processes the query
  ├── Text   → Llama 3.3 70B
  ├── Image  → Llama 4 Scout (Vision)
  └── Voice  → Whisper → Llama 3.3 70B
        ↓
Clinical response generated
        ↓
Airtable logs query + response + latency
        ↓
User receives structured drug analysis
```

**Beta sprint:** 7 days · 25 verified health professionals · February 2026

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Interface | Telegram Bot API | User-facing chat interface |
| Automation | n8n (self-hosted) | Workflow orchestration |
| AI — Text | Groq / Llama 3.3 70B | Drug query processing |
| AI — Vision | Groq / Llama 4 Scout | Prescription photo analysis |
| AI — Voice | Groq Whisper | Voice-to-text transcription |
| Database | Airtable | User registry + query logging |
| Email | Gmail API | Automated onboarding |

---

## Repository Structure

```
sofhealth-bot/
├── README.md
├── docs/
│   ├── PRD_v2.docx                     Product Requirements Document
│   ├── GTM_Strategy.docx               Go-to-Market Strategy
│   ├── Technical_Architecture.docx     System Architecture Document
│   ├── Technical_Specification.docx    Developer & API Specification
│   ├── Business_Model.docx             Unit Economics & Revenue Model
│   ├── Legal_Compliance.docx           NDPR & Regulatory Checklist
│   └── Sprint_Findings_Template.docx   Beta Sprint Framework
├── architecture/
│   └── SOFHealth_Architecture_Map.jsx  Interactive system map (React)
├── workflows/
│   └── clinpharm_bot.json              n8n workflow — import to run
└── assets/
    └── workflow_screenshot.png         n8n workflow screenshot
```

---

## Architecture Overview

The full interactive architecture map is in `/architecture/SOFHealth_Architecture_Map.jsx`.

It covers every layer of the system:

```
INPUT LAYER          Camera · Voice · Text
      ↓
CLIENT LAYER         React Native App (v2.0 target)
      ↓
API GATEWAY          Auth · Rate limiting · Audit logging
      ↓
PROCESSING PIPELINE  OCR (Google Vision) · STT (Whisper) · NLP (spaCy)
      ↓              [All three run in parallel]
CLINICAL ENGINE      OpenFDA · RxNorm · DDI Database · LLM Synthesis
      ↓
STORAGE LAYER        PostgreSQL · Redis Cache · Encrypted S3
      ↓
COMPLIANCE LAYER     NDPR · AES-256 · PII Detection · Audit Trail
      ↓
RESPONSE             Structured clinical analysis · Veracity score · Citation
```

Target response time: **< 5 seconds** on a photo query end-to-end.

---

## Documentation

All product documentation is in `/docs/`. It covers:

| Document | What It Contains |
|----------|-----------------|
| PRD v2.0 | Full product requirements, feature specs, risk register, sprint criteria |
| GTM Strategy | 4-phase go-to-market plan, acquisition channels, pricing model |
| Technical Architecture | Layer-by-layer system design, stack decisions, data flow |
| Technical Specification | API contracts, database schema SQL, build sequence, hiring brief |
| Business Model | Unit economics, break-even analysis, 3-year revenue projections |
| Legal & Compliance | CAC registration, NDPR checklist, NAFDAC positioning, ToS requirements |
| Sprint Template | Fillable 7-day beta findings framework and Go/No-Go decision criteria |

---

## Roadmap

### v1.0 — Current (Telegram Bot)
- [x] Multimodal input: text, voice, image
- [x] Groq AI integration (Llama + Whisper)
- [x] User registration and whitelisting
- [x] Query logging with latency tracking
- [x] Automated onboarding email
- [x] Waitlist logic (25-user cap)

### v2.0 — Native App (In Specification)
- [ ] React Native iOS + Android app
- [ ] OpenFDA + RxNorm API integration
- [ ] Drug-Drug Interaction engine (DrugBank/Lexicomp)
- [ ] Patient context contraindication engine
- [ ] Veracity scoring with source citations
- [ ] NDPR-compliant data pipeline
- [ ] Biometric authentication
- [ ] Freemium subscription model

---

## Important Notice

> SOFHealth is a **clinical reference tool** for licensed health professionals.
> It is not a diagnostic system, a prescribing engine, or a replacement for clinical judgment.
> All clinical decisions remain the responsibility of the licensed professional.
>
> **Do not input identifiable patient data during beta.**

---

## Builder

**Shedrach Obafemi Rotimi**
Pharmacist · AI Builder · Health Tech

- 🔗 [LinkedIn](https://linkedin.com/in/shedrachobafemi)
- 💼 [Upwork](https://www.upwork.com/freelancers/shedrachobafemi)
- ✍️ [Substack](https://substack.com/@mrobafemi)

---

*Built in Nigeria. For health professionals everywhere.*
