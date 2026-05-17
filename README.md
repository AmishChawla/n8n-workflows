# 🤖 AI SDR Agent Suite (n8n + Gemini 2.5 + LangChain)

An enterprise-grade, automated Sales Development Representative (SDR) agent system built using **n8n**, **Google Gemini 2.5 Flash Lite**, **LangChain**, and **PostgreSQL**. This suite automates the entire outbound and inbound sales funnel: from scraping/adding new leads to generating hyper-personalized emails, extracting scheduling intent, checking calendar availability, auto-booking Google Meet sessions, and managing long-term conversation memory.

---

## 📸 Workflow Walkthrough (Screenshots)

Below is a visual overview of the agent system running in n8n:

### 1. Inbound Leads Outreach Flow
This workflow triggers automatically when a new lead is added to the Google Sheet leads database. It retrieves calendar availability, determines free slots, and writes a highly personalized introduction.

![Inbound Outreach Workflow](screenshots/Screenshot%202026-03-09%20at%2012.44.44%20PM.png)

---

### 2. Conversational Intent & Reply Intelligence Flow
The core decision engine of our inbound sales rep. It intercepts new Gmail replies, retrieves context from a PostgreSQL database, classifies intent, and branches accordingly.

![Reply Intelligence & Scheduling Flow](screenshots/Screenshot%202026-03-09%20at%2012.48.04%20PM.png)

---

### 3. Real-time Scheduling and Calendar Engine
Visual mapping of the Google Calendar node verifying calendar slots and booking discovery calls with a Google Meet link automatically attached.

![Calendar Node Mapping](screenshots/Screenshot%202026-03-09%20at%2012.45.24%20PM.png)

---

### 4. LangChain Chat Memory and PostgreSQL Store
Demonstrates how the agent stores, manages, and retrieves past interactions using memory managers to ensure it always remembers previous discussions with leads.

![Chat Memory Nodes](screenshots/Screenshot%202026-03-09%20at%2012.49.00%20PM.png)
![PostgreSQL Session Store](screenshots/Screenshot%202026-03-09%20at%2012.49.53%20PM.png)

---

## ⚙️ Core Workflows

### 1. AI Outreach Agent (Outbound Funnel)
- **Trigger**: Listens for new rows added to a Google Sheets database (Leads list).
- **Calendar Intel**: Pulls current events from Google Calendar and runs an optimized JavaScript node to calculate 2 non-conflicting, randomized 30-minute meeting slots over the next 5 days.
- **AI Composition**: Sends the lead information, message, and available slots to **Gemini 2.5 Flash Lite**. Gemini generates a concise, friendly, and natural outreach email following precise sales rules (pricing ranges, service retainers, B2B targets).
- **Email Out**: Sends the email directly via Gmail.
- **Memory Initialization**: Logs the email thread ID in a PostgreSQL-backed LangChain Chat Memory manager to start tracking the conversation history.

### 2. AI SDR Reply Intelligence (Inbound Funnel)
- **Trigger**: Triggered by incoming replies in Gmail.
- **Message Cleaning**: A JavaScript node extracts and cleans the email content, stripping out old quoted emails or signatures to keep the context clean.
- **Context Retrieval**: Pulls the thread's complete historical context from the PostgreSQL database using LangChain's memory module.
- **Intent Classification**: Uses Gemini to analyze the message and classify the lead's intent into one of three buckets:
  1. `BookMeeting`: The lead wants to schedule a call or mentions availability.
  2. `QuestionHandler`: The lead asks questions about services, packages, pricing, or scope.
  3. `ObjectionHandler`: The lead expresses doubts, budget issues, or declines.
- **Automated Routing & Actions**:
  - **If BookMeeting**: The agent automatically checks calendar availability for the date/time mentioned by the lead.
    - *If Free*: Schedules the meeting, attaches a Google Meet link, sends a confirmation email to the lead, and logs it.
    - *If Busy*: Calculates 2 alternative free slots, proposes them politely, and emails them back.
  - **If QuestionHandler**: Drafts a conversational response answering the specific queries (pulling details fromABC TimePass Agency rules) and prompts them to book a call.
  - **If ObjectionHandler**: Gracefully acknowledges concerns, explains retainers, and offers a Free DIY Content Framework Guide if there is no budget.
- **Memory Store**: Saves both user input and agent reply back to Postgres.

---

## 📂 Repository Structure

```directory
.
├── ai_agents/                   # Placeholder for additional agents
│   └── lead_generation_agent/   
├── customer_support/            # Placeholder for customer support pipelines
├── screenshots/                 # Workflow walkthough screenshots
│   ├── Screenshot...12.44.44.png
│   ├── Screenshot...12.45.24.png
│   ├── Screenshot...12.48.04.png
│   ├── Screenshot...12.49.00.png
│   └── Screenshot...12.49.53.png
├── sdr/
│   └── n8n/
│       ├── prompts to gemini/   # Raw prompt structures used by LLMs
│       │   ├── outreach(Gemini)/
│       │   │   └── GenerateEmailPrompt.json
│       │   └── reply handler/
│       │       ├── ClassifyIntent(Gemini).json
│       │       └── QuestionHandler(Gemini).json
│       └── workflows/           # n8n Workflow JSON Exports (Sanitized)
│           ├── AI Outreach Agent (Gemini) (1).json
│           └── AI SDR Reply Intelligence (Gemini) (1).json
├── .gitignore                   # Configured to ignore secrets/credentials
```


---

## 🚀 Setup & Installation

1. **Prerequisites**: 
   - An active **n8n** instance (Local, Cloud, or Docker-hosted).
   - A **PostgreSQL** database (to run LangChain chat memory).
   - Google Cloud Credentials configured in n8n for Gmail and Google Calendar.
   - A **Google Gemini API Key** (from Google AI Studio).

2. **Import Workflows**:
   - Go to your n8n workspace.
   - Click **Add Workflow** -> **Import from File**.
   - Select either of the JSON files in `sdr/n8n/workflows/`.

3. **Configure Node Variables**:
   - Replace the placeholders (`YOUR_GEMINI_API_KEY`, `YOUR_EMAIL@gmail.com`, etc.) in the HTTP Request nodes and Trigger nodes with your actual keys (available in your local [credentials_backup.md](credentials_backup.md)).
   - Link your local Google OAuth2 credentials to the Gmail and Google Calendar nodes inside n8n.

4. **Run**:
   - Toggle both workflows to **Active** inside n8n!
