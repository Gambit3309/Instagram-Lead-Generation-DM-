[README.md](https://github.com/user-attachments/files/34510366/README.md)
# 🚀 Instagram Lead Gen & AI Outreach Pipeline (n8n)

An automated, modular pipeline built in [n8n](https://n8n.io/) for targeted B2B Instagram outreach. This project handles the entire lead generation lifecycle: scraping targeted Instagram accounts, cleaning and scoring the data, utilizing AI to determine prospect fit, drafting highly personalized outreach messages, and queuing them for human review.

This was developed as a final internship project, demonstrating advanced workflow architecture, API integrations, and prompt engineering.

---

## ✨ Features

- **Automated Data Acquisition:** Scrapes Instagram profiles based on dynamic hashtags using Apify.
- **Fail-Safe Mechanisms:** Global error handling with Slack alerts and local CSV fallbacks if scraping APIs fail.
- **Intelligent Scoring:** Combines rule-based Regex scoring with an LLM (GPT-4o) fit-check to qualify leads.
- **AI-Drafted Outreach:** Uses AI Agents (LangChain / GPT-4o-mini) to draft personalized, context-aware DMs based on the prospect's bio and niche.
- **Human-in-the-Loop (HITL):** Queues messages in Google Sheets and alerts the team via Slack for final approval before sending.
- **Prompt-Injection Safe:** Sanitizes inputs to prevent malicious Instagram bios from hijacking the AI instructions.

## 🏗 Architecture

The system is broken down into modular sub-workflows orchestrated by a master controller:

*   **`MASTER.json`**: The orchestrator. Runs sub-workflows in a strict sequence (S1 -> S2 -> S3 -> S4).
*   **`S0_ErrorHandler.json`**: Universal error catcher. Formats error data and posts to Slack and Google Sheets. Set this as the "Error Workflow" in n8n settings for all other workflows.
*   **`S1_Scraper.json`**: Sources top-of-funnel leads via Apify (Hashtag & Profile scrapers) with a hard rate limit and CSV fallback.
*   **`S2_CleanAndScore.json`**: Dedupes, extracts emails/URLs, applies rule-based scoring, and uses AI to determine final prospect tier (Hot/Warm/Cold).
*   **`S3_Sequence.json`**: The AI Agent reads qualified leads and templates to draft personalized messages with mandatory opt-out compliance.
*   **`S4_QueueAndAlerts.json`**: Pushes Slack notifications that queues are ready for human review.

## 📋 Prerequisites

To run this pipeline, you will need:
1.  **[n8n](https://n8n.io/)** (Self-hosted or Cloud)
2.  **Google Cloud Platform Account** (For Google Sheets API credentials)
3.  **Apify Account** (For Instagram scraping actors)
4.  **OpenRouter Account** (For OpenAI GPT-4o and GPT-4o-mini API access)
5.  **Slack Workspace** (For alerts)

## 🚀 Setup & Installation

### 1. Database Setup
1. Create a new Google Sheet.
2. Create the following tabs: `Config`, `Leads`, `Templates`, `Queue`, `Events`.
3. Set up your `Config` tab with variables like `SCRAPER_MODE` (csv/apify), `HASHTAGS`, `POSTS_LIMIT`, and `MAX_PROFILES`.

### 2. Import Workflows
1. Clone this repository.
2. Open your n8n workspace.
3. Import the workflows in this order: `S0`, `S1`, `S2`, `S3`, `S4`, and finally `MASTER`.
4. *Important:* Open the `MASTER` workflow and ensure the `Execute Workflow` nodes are pointing to the correctly imported S1-S4 workflows. Do the same for the `S0` error handler in the workflow settings of S1-S4.

### 3. Configure Credentials
1. Connect your Google Sheets OAuth2 API.
2. Connect your Slack OAuth2 API (or Webhook).
3. Connect your OpenRouter API key.
4. Add your Apify API token inside the HTTP Request nodes in `S1_Scraper`.

### 4. Update Node References
1. Go through the Google Sheets nodes in workflows S2, S3, and S4.
2. Update the Document ID and Sheet names to point to your newly created Google Sheet.

## 💻 Usage

To run the pipeline, simply click **Execute Workflow** on the `MASTER` workflow, or enable the built-in Schedule Trigger to run it daily at your preferred time. 

Watch the Slack channel for the final output alert indicating your queue is ready for review!

---
*Created as part of an engineering internship project.*
