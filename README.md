# Delikat-HR-Bot-n8n-Telegram-

#  DELIKAT HR Bot — AI-Powered Human Resources Assistant

> An intelligent, Telegram-based HR automation system built with n8n for DELIKAT restaurant group. Designed for management and area managers to handle HR tasks through natural language — no technical knowledge required.

---

##  Workflow Overview

<img width="1235" height="693" alt="image" src="https://github.com/user-attachments/assets/2091d89a-ec62-4f7a-ad69-556634c3e0a3" />

### Ingestion Workflow (PDF → Qdrant)

<img width="711" height="395" alt="image" src="https://github.com/user-attachments/assets/4eeb5e12-b4e9-4f7d-a3df-c90bc8e131c2" />

### RAG Query Workflow (Policy Q&A)

<img width="318" height="296" alt="image" src="https://github.com/user-attachments/assets/c3ecfe19-1e64-4a5c-b3ce-1617836bd0f0" />

---

##  Features

###  Employee Management
- Track employee records stored in Google Sheets
- Add absences (faltas) and warnings (amonestaciones) via Telegram
- Count records by type (Absence / Warning / Sick leave)

###  Document Generation
- Generate warning letters and contracts automatically
- Documents created in Google Docs and sent as PDF via Telegram

###  Role-Based Permissions
- **Management** — Full access to all areas
- **Area Managers** — Restricted to their own area only
- Bot blocks cross-area data access automatically

###  AI Knowledge Base (RAG)
- Answers HR policy questions in Spanish
- Based on internal documents:
  - Reglamento Interno de Trabajo (DELIKAT)
  - Código de Trabajo de El Salvador
- Cites specific articles as references

###  Natural Language Interface
- No commands needed — just type naturally in Telegram
- Supports Spanish and English
- Friendly fallback messages for unrecognized inputs

---

##  Architecture

```
Telegram Message
      │
      ▼
┌─────────────────────────────────┐
│   Auth Check                    │
│   Manager Lookup — Google Sheets│
└─────────────────────────────────┘
      │ Authorized          │ Unauthorized
      ▼                     ▼
Intent Parser (JS)     ❌ Access Denied
      │
      ▼
┌─────────────────────────────────────────────────┐
│                  Switch Router                   │
└──┬─────┬──────┬──────┬───────┬──────┬───────────┘
   │     │      │      │       │      │
   ▼     ▼      ▼      ▼       ▼      ▼
Count  Add    Add   Generate Generate Policy
       Absence Warning Warning Contract Q&A
         │      │      Doc     Doc      │
         │      │       │       │       ▼
         │      │       ▼       ▼   Qdrant Vector DB
         │      │   Google   Google  (1,252 points)
         │      │   Docs     Docs        │
         ▼      ▼       │       │        ▼
    Permission Permission ▼      ▼   GPT-4o (OpenAI)
    Check      Check   PDF     PDF       │
    (Area)     (Area)  →Drive  →Drive    ▼
         │      │       │       │   Telegram Reply
         ▼      ▼       ▼       ▼   (Spanish Answer
    Google  Google  Telegram Telegram  + Article Ref)
    Sheets  Sheets  Send    Send
    Append  Append  PDF     PDF
         │      │
         ▼      ▼
    Telegram Telegram
    Confirm  Confirm
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Bot Interface | Telegram |
| Workflow Engine | n8n (self-hosted) |
| Employee Data | Google Sheets |
| Document Storage | Google Drive |
| Document Generation | Google Docs |
| Vector Database | Qdrant Cloud |
| AI Model | GPT-4o (OpenAI) |
| Embeddings | text-embedding-3-small |
| Language | Spanish / English |

---

##  Project Structure

```
DELIKAT HR Bot
├── Workflows
│   ├── Main HR Bot (Telegram Trigger → Auth → Intent → Actions)
│   ├── PDF Ingestion (Google Drive → Qdrant)
│   └── RAG Query (Telegram → Qdrant → GPT-4o → Telegram)
│
├── Google Sheets
│   ├── Manager List (Auth + Role data)
│   ├── Employees (Employee records)
│   └── Absence (Absences + Warnings log)
│
├── Google Drive
│   └── HR Policies (PDF documents)
│       ├── Reglamento Interno de Trabajo.pdf
│       └── Codigo de trabajo de el salvador.pdf
│
└── Qdrant Cloud
    └── delikat-hr (1,252 vector points)
```

---

##  Example Commands

```
# Absence & Warning Tracking
"Add absence for Rahim"
"Add warning for Maria"
"How many absences does Rahim have?"
"How many warnings does Juan have?"

# Document Generation
"Generate warning for Rahim"
"Generate contract for Maria"

# Policy Questions (Spanish)
"¿Cuántos días de vacaciones tiene derecho un trabajador?"
"¿Qué pasa si un trabajador falta sin justificación?"
"¿Cuál es el horario de trabajo en DELIKAT?"
"¿Cómo se calcula el pago de horas extras?"
```

---

##  Security & Permissions

| Action | Management | Area Manager |
|---|---|---|
| View own area employees | ✅ | ✅ |
| View other area employees | ✅ | 
| Add absence/warning | ✅ | Own area only |
| Generate documents | ✅ | Own area only |
| Policy Q&A | ✅ | ✅ |

---

##  Current Status

| Feature | Status |
|---|---|
| Auth system | ✅ Complete |
| Add absence | ✅ Complete |
| Add warning | ✅ Complete |
| Count absence/warning | ✅ Complete |
| Permission check | ✅ Complete |
| RAG / Policy Q&A | ✅ Complete |
| Document generation | ✅ Complete |
| Drive folder structure | ✅ Complete |
| Add/update employee | ✅ Complete |
| Client n8n migration | ✅ Complete |

---

##  Setup Guide

### Prerequisites
- n8n instance (self-hosted or cloud)
- Google Workspace account
- OpenAI API key
- Qdrant Cloud account
- Telegram Bot (via BotFather)

### Step 1 — Google Sheets Setup
Create 3 sheets:
1. **Manager List** — columns: `Name`, `Telegram ID`, `Area`, `Identificador`, `Role`
2. **Employees** — columns: `Name`, `Area`, `Position`
3. **Absence** — columns: `Date`, `Manager Name`, `Area`, `Employee Name`, `Reason`

### Step 2 — Qdrant Setup
1. Create account at [cloud.qdrant.io](https://cloud.qdrant.io)
2. Create cluster: `delikat-hr`
3. Note your API key and cluster URL

### Step 3 — PDF Ingestion
1. Upload HR policy PDFs to Google Drive folder
2. Run the PDF Ingestion workflow once
3. Verify ~1,252 points in Qdrant dashboard

### Step 4 — Import Workflows
1. Import all 3 workflow JSONs into n8n
2. Set credentials (Google, OpenAI, Qdrant, Telegram)
3. Activate workflows

### Step 5 — Test
Send these messages to your Telegram bot:
```
Add absence for [employee name]
How many absences does [name] have?
¿Cuántos días de vacaciones tiene derecho un trabajador?
```

---

##  Built By

**Shuvo Biswas**
```
AI Engineer (Softvence Agency Group of Betopia)
```
---

##  Project Timeline

| Date | Milestone |
|---|---|
| May 2026 | Project started |
| May 10, 2026 | Core HR bot + auth system |
| May 22, 2026 | RAG / Knowledge base complete |
| June 3, 2026 | Absence/Warning tracking + permissions |
| June 10, 2026| Document generation + Drive integration |
| TBD | Client n8n migration + delivery |
