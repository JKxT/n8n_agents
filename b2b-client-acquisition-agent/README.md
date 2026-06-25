# Autonomous B2B Client Acquisition Agent (n8n Workflows)

This directory contains the production-ready n8n workflows and setup documentation for an **Autonomous B2B Client Acquisition Agent**. This agent automates the entire sales pipeline: finding leads, analyzing their website, identifying pain points, searching for decision-maker contacts, logging data in a CRM, sending highly personalized cold outreach, and managing follow-ups automatically.

---

## Workflows Included

1. **`workflow_main.json` (Main Acquisition Flow)**:
   - Sourced from a target niche and city (e.g., "Dental Clinics in Miami, FL") using Google Maps.
   - Extracts website URL and cleans up the domain.
   - Inspects the CRM (Google Sheets) to check if the lead was already contacted to prevent duplicates.
   - Scrapes the website homepage and cleans HTML code to save token context.
   - Uses an LLM (GPT-4o-mini) to analyze the site and identify 2-3 specific business/technical pain points.
   - Searches Hunter.io Domain Search API to find decision makers (email, first name, title).
   - Generates a bespoke, highly personalized cold email emphasizing the extracted pain points.
   - Sends the email using Gmail/SMTP.
   - Appends/updates the lead record in Google Sheets.

2. **`workflow_followup.json` (Follow-up & Reply-Tracking Flow)**:
   - Triggers daily at 9:00 AM.
   - Retrieves active outreach records (status: `OUTREACH_SENT`, `FOLLOWUP_1_SENT`) that have had at least 3 days of silence.
   - Queries Gmail to check if the lead has replied.
   - **If Replied**: Stops the sequence, marks the CRM status as `REPLIED`, and emails a notification to the owner to take over manually.
   - **If No Reply**: Increments the follow-up count, drafts a custom bump or break-up email based on the sequence level, delivers it, and updates the CRM. Closes the lead after 2 follow-ups.

---

## Prerequisites

Before importing, ensure you have credentials/accounts for:
1. **n8n Instance**: Cloud or self-hosted.
2. **OpenAI API Key**: For website analysis and personalized copywriting.
3. **Google Workspace Account**: For Google Sheets (CRM) and Gmail (Outreach Sender).
4. **Google Maps API Key**: To source leads.
5. **Hunter.io API Key**: (Optional but highly recommended) To find decision makers.

---

## 1. CRM Setup (Google Sheets)

Create a new Google Sheet named **B2B Leads CRM** and create a worksheet tab named `Leads`. Put the following exact column headers in row 1:

| Column A | Column B | Column C | Column D | Column E | Column F | Column G | Column H | Column I | Column J | Column K | Column L | Column M |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Name** | **Website** | **Domain** | **Email** | **First Name** | **Title** | **Summary** | **Pain Points** | **Status** | **Follow Up Count** | **Date Added** | **Date Contacted** | **Date Replied** |

*Note: The workflows refer to these columns exactly. Ensure spelling matches.*

---

## 2. n8n Import Instructions

For both `workflow_main.json` and `workflow_followup.json`:
1. Log into your n8n dashboard.
2. Click **Workflows** > **Add workflow** (or click the plus icon in the top right).
3. Click the three dots menu in the top right corner and select **Import from File**.
4. Upload the corresponding JSON file.

---

## 3. Configuration of Credentials

Once imported, you will see yellow caution triangles on nodes requiring credentials. Configure them as follows:

*   **Google Maps Search Node**: Setup a Google Maps credential using your Google Cloud API key.
*   **Google Sheets Nodes**: Setup a Google Sheets OAuth2 credential connected to your Google Workspace account. Make sure to paste the Google Sheet ID (found in the sheet URL: `https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEETS_ID/edit`) into the **Document ID** parameter.
*   **Gmail Nodes**: Setup a Gmail OAuth2 credential connected to the email inbox you want to use for outreach.
*   **OpenAI Nodes**: Setup an OpenAI API credential with your API key.

---

## 4. Parameter Setup & Customization

### In the Main Workflow:
Double-click the **Set Lead Params** node (the 3rd node in the flow) and configure the default values:
*   `niche`: Target business type (e.g. `Dental Clinics`, `Boutique Gyms`, `Law Firms`).
*   `city`: Target geographic location (e.g. `Miami, FL`, `Austin, TX`).
*   `limit`: Number of new leads to extract and process per run (recommend starting with `5` to test).
*   `sender_name`: Your name (e.g., `Alex Mercer`).
*   `sender_company`: Your agency/business name (e.g., `Apex Dental Marketing`).
*   `hunter_api_key`: Paste your Hunter.io API key here (if you do not have one, the flow will fail gracefully and default to guessing `info@domain.com`).

### In the Follow-up Workflow:
Double-click the **Set Settings** node:
*   `sender_name`: Must match the sender name in the main workflow.
*   `sender_company`: Must match the sender company.
*   Double-click the **Notify Replied** node and set `Send To` to your personal notification inbox so you get notified immediately when a lead replies.

---

## 5. Adjusting LLM Prompts (Niche Specialization)

The copywriting style is controlled by the prompts inside the **Draft Outreach (LLM)** and **Draft Follow-up (LLM)** nodes. 

If you are offering a specific service (like SEO audits, Web Development, or Facebook Ads), edit the **Draft Outreach (LLM)** prompt. For example:
*   *Default*: Offers a custom mockup of their landing page.
*   *SEO Alteration*: "Pivot naturally to one SEO-related pain point (e.g., missing meta descriptions or slow page speed) and offer a quick 2-page SEO report."

---

## Compliance & Warm-up Recommendations

*   **CAN-SPAM / GDPR**: Ensure your outreach email includes a clear opt-out statement (e.g., *"If you'd prefer I don't reach out again, just let me know."*).
*   **Email Warmup**: Do not send cold emails immediately from a brand new domain or account. Warm up the inbox for 14-30 days prior to launching the agent at scale.
*   **Daily Limits**: Keep daily cold emails under 30-50 per inbox to avoid damaging your sender domain reputation. If you want to scale, connect multiple Gmail inboxes and use n8n to rotate credentials.
