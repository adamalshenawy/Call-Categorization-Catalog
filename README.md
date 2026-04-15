# 🎙️ Call Categorization Catalog
### AI-Powered Sales Call Analysis & CRM Stage Classification Engine

> **Automatically transcribes recorded sales calls, detects buying signals, and classifies the correct CRM stage — eliminating human bias from data entry entirely.**

---

## 📌 Short Intro

A production-ready AI automation engine built with **n8n**, **OpenAI GPT-4o**, **ElevenLabs**, and **AssemblyAI**.  
It takes a raw audio recording of a real estate sales call, understands what was said in **Egyptian Arabic**, separates the agent's speech from the client's, detects key signals, and outputs a structured CRM stage classification with full reasoning — all without a single human click.

---

## 🚨 The Problem

Sales agents were manually updating CRM stages after every call. This created serious data quality issues:

- 🧑‍💼 Agents manually decide the CRM stage after each call — introducing personal bias
- ⚠️ Human judgment is inconsistent — the same conversation could be classified differently by different agents
- 🧠 Agents forget to update the system, or misread the client's actual interest level
- 📉 Incorrect CRM data destroys visibility into the real sales pipeline
- 📊 Managers cannot reliably track lead progression, conversion rates, or agent performance
- ❗ Weak data = weak forecasting = poor business decisions

---

## ✅ The Solution

An end-to-end AI pipeline that processes every recorded call automatically:

1. **Audio received via Webhook** — file name + Google Drive link sent via POST request
2. **File metadata extracted** — AI reads the filename to extract client name, phone number, and campaign name
3. **Format detection** — system checks if the file is `.amr` or standard audio (`.m4a`, `.mp4`, etc.)
4. **Transcription** — two paths depending on format:
   - Standard audio → **ElevenLabs** (Arabic, 2-speaker diarization)
   - AMR format → **AssemblyAI** (upload → start job → wait 60s → retrieve transcript)
5. **Speaker separation** — AI Agent separates the transcript into what the **sales agent** said vs what the **client** said
6. **CRM classification** — AI Agent analyzes **only the client's speech** for key signals:
   - 💰 Numeric budget statements
   - ❓ Objections and hesitations
   - 📅 Meeting commitments with specific dates/times
   - 💬 General interest without commitment
7. **Stage assigned** from predefined real estate milestones
8. **Results logged** to Google Sheets — client info, agent info, AI stage, agent stage, mismatch flag, and full AI reasoning

---

## 📈 Business Impact

- 🎯 **Zero human bias** — classification is based purely on what the client said, not what the agent thinks
- 📊 **Consistent, standardized** lead qualification across all agents and campaigns
- 🔎 **Real pipeline visibility** — managers see the actual stage of every lead, not a subjective one
- 🧬 **Structured data for analytics** — every call produces a labeled, reasoned data point
- 📉 **Better forecasting** — clean CRM data enables accurate conversion prediction
- 🚀 **Long-term ROI** — sales conversations become valuable, reusable business intelligence

---

## 🛠️ Technologies Used

| Tool | Role |
|------|------|
| **n8n** | Workflow automation engine — orchestrates the entire pipeline |
| **OpenAI GPT-4o** | AI reasoning — speaker separation + CRM stage classification |
| **ElevenLabs** | Transcription of standard audio files (Arabic, 2-speaker diarization) |
| **AssemblyAI** | Transcription of AMR audio format |
| **Google Drive** | Storage for raw audio recordings |
| **Google Sheets** | Output destination — Categorization Comparison log |
| **Webhook (n8n)** | Entry point — receives file name + Drive link via POST |
| **JavaScript (Code nodes)** | File size calculation, JSON parsing, data transformation |

---

## ⚙️ Features

- ✅ **Dual transcription engine** — handles both standard (m4a, mp4) and AMR audio formats automatically
- ✅ **Arabic language support** — built specifically for Egyptian Arabic real estate calls
- ✅ **2-speaker diarization** — intelligently separates agent speech from client speech
- ✅ **Filename intelligence** — extracts client name, phone number (Egyptian format: 010/011/012/015), and campaign name from the recording filename alone
- ✅ **Client-only analysis** — CRM stage is based exclusively on what the client said, not the agent
- ✅ **Structured reasoning** — every classification includes a quoted justification from the client transcript
- ✅ **Mismatch detection** — compares AI classification vs agent classification and flags discrepancies
- ✅ **Production-ready** — live webhook endpoint, error handling, format detection, async polling for AssemblyAI

---

## 👤 What Users Can Do

**Team Leaders / Managers can:**
- Open the Google Sheets log and see every processed call with AI stage + reasoning
- Filter by agent name, campaign, or mismatch flag
- Identify which agents are consistently miscategorizing leads
- Use the data directly in Power BI for pipeline analytics

**The System Does Automatically:**
- Transcribe the audio
- Extract client and agent info from the filename
- Separate speaker roles
- Classify the CRM stage with reasoning
- Log everything to the sheet

---

## 🏗️ How I Built It

### Architecture Overview
```
Webhook (POST)
    │
    ├── AI Agent → Extract: client name, phone, campaign from filename
    │
    ├── Google Drive → Download audio file
    │
    ├── Code Node → Calculate file size
    │
    ├── IF: Is file AMR?
    │     ├── YES → AssemblyAI: Upload → Start → Wait(60s) → Get transcript
    │     └── NO  → ElevenLabs: Transcribe (Arabic, 2-speaker diarization)
    │
    ├── AI Agent (GPT-4o) → Speaker Diarization
    │     └── Separates: sales_agent[] | client[] | unknown[]
    │
    ├── AI Agent (GPT-4o) → CRM Classification
    │     └── Analyzes CLIENT speech only
    │     └── Maps to: interested / potential to close / meeting scheduled
    │     └── Returns: stage + exact client quote as evidence
    │
    ├── Code Node → Parse JSON output + save stage & reasoning
    │
    └── Google Sheets → Append row:
          Client Name | Campaign | Agent Name | AI Stage | Agent Stage | Mismatch? | AI Reasoning
```

### Key Engineering Decisions

**Why two transcription engines?**  
AMR is a compressed mobile audio format common in Egypt (WhatsApp voice notes). ElevenLabs doesn't support it, so I built a fallback path to AssemblyAI which does — with async polling to handle the delay.

**Why separate the speakers before classifying?**  
A budget number in the agent's mouth ("we offer plans from 500k") is meaningless. The same number in the client's mouth ("my budget is 500k") is a buying signal. Diarization before classification was the only way to make this reliable.

**Why analyze only the client transcript?**  
The agent always sounds interested — that's their job. The client's actual words are the only truthful signal of where they are in the buying journey.

**Why extract info from the filename?**  
The audio files are named with client info (phone number, name, campaign) by the business workflow. Parsing the filename with an AI agent was faster and more reliable than requiring structured input.

---

## 🧠 What I Learned

- **Prompt engineering for structured output** — getting GPT-4o to return clean, parseable JSON consistently required strict prompt design and output validation
- **Handling real-world audio formats** — AMR support was an unexpected challenge that taught me to always design for format variability
- **Speaker diarization without labels** — building a prompt that correctly attributes speech roles without ground-truth labels was the hardest part of this project
- **Async API patterns in n8n** — AssemblyAI's transcription is asynchronous; I learned to implement polling with a Wait node + retry logic
- **Data quality as a product** — the real value isn't the classification itself, it's the structured, consistent dataset it produces over hundreds of calls

---

## 📁 Project Structure

```
call-categorization/
│
├── Categorization_Engine_unified_Production_Ready.json   # Full n8n workflow
├── README.md
└── docs/
    ├── business_problem.pdf
    └── solution_overview.pdf
```

---

## 🚀 How to Run

1. Import the `.json` workflow into your n8n instance
2. Set up credentials:
   - OpenAI API key
   - ElevenLabs API key
   - AssemblyAI API key
   - Google Drive OAuth
   - Google Sheets OAuth
3. Update the Google Sheets document ID to your own sheet
4. Activate the workflow
5. Send a POST request to the webhook with:
```json
{
  "fileName": "Ahmed Mohamed - Obour City 01012345678_20260101.m4a",
  "fileId": "YOUR_GOOGLE_DRIVE_FILE_ID"
}
```
6. Check your Google Sheet for the output

---

## 📬 Contact

**Adam Alshenawy** — Data Analyst & AI Automation Specialist

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/adam-alshenawy-600b41267/)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://wa.me/201096322218?text=Hello%20I%20am%20interested%20in%20your%20AI%20automation%20and%20data%20analysis%20services)
[![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/Adam_Alshenawy)
[![Email](https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:adamalshenawy0@gmail.com)
