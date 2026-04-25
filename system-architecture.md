🏗️ System Architecture: SOFHealth Clinical AI
Project: @SOFHealth_bot (v1.1)

Lead Architect: Obafemi Shedrach

Date: April 25, 2026

1. High-Level Overview
The architecture follows a Decoupled Multimodal Pattern. It separates the user interface (Telegram/App) from the processing logic (n8n/AI Engine) to ensure that we can swap the backend or frontend without breaking the entire system.

2. Layered Architecture
A. Presentation Layer (Frontend)
Telegram Bot API: Current primary interface for voice, image, and text input.

Web Form (n8n): External entry point for user registration and whitelisting.

Future Native App: iOS/Android interface using high-resolution camera modules.

B. Orchestration & Logic Layer (Middleware)
n8n Workflow Engine: The "Brain" that manages the sequence of events.

Input Router: Detects if the incoming data is a .jpg (Image), .ogg (Voice), or string (Text).

Wait Controller: Manages the 1-hour delay for onboarding emails.

Auth Gate: Checks the incoming Telegram ID against the "Verified Professionals" list.

C. Cognitive Layer (AI & Analysis)
OCR Engine: Extracting text from prescription photos.

Speech-to-Text (STT): Converting clinical voice notes into structured data.

Clinical Reasoning Engine: Large Language Model (LLM) tuned for pharmacology to check interactions and veracity.

D. Data Layer (Persistence)
Airtable: Relational database for tester profiles, waitlist management, and activity logging.

Clinical Knowledge Base: External APIs (future) or internal reference files used for drug-drug interaction (DDI) checks.

3. Data Flow Diagram (The Lifecycle of a Query)
Ingestion: User sends a voice note to @SOFHealth_bot.

Authentication: System checks if the User ID exists in the Airtable: Verified table.

Processing:

If Voice: Routed to STT → Text Query.

If Photo: Routed to OCR → Drug Extraction.

Clinical Lookup: The extracted drug name is checked against the Interaction Engine.

Response: A structured analysis (Accuracy & Speed focus) is sent back to the Telegram UI.

Telemetry: The transaction (without patient data) is logged for the 7-day performance report.

4. Security & Compliance (Non-Negotiable)
Data Minimization: No storage of patient-identifiable information (PII) during the beta.

Encrypted Transport: All data between Telegram, n8n, and Airtable is transmitted via HTTPS/TLS 1.2+.

Access Control: Use of environment variables in n8n for API keys and credentials to prevent exposure.

5. Deployment Environment
Platform: n8n Cloud (Production Instance).

Integrations: Gmail API (SMTP), Telegram Bot API, Airtable API.
