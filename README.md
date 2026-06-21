# Agent 01 — Autonomous B2B Client Acquisition Agent

> Built by [Elixr Co](https://github.com/elixr-co) · Islamabad, Pakistan  
> CEO & Co-Founder: [Sumair Ul Hassan](https://www.linkedin.com/in/sumair-ul-hassan-065544260)

---

## What This Agent Does

This is a fully autonomous B2B lead generation and outreach pipeline. Once configured and triggered, it runs end-to-end without human input:

1. **Finds businesses** matching your target niche via Google Maps API
2. **Fetches business details** — website, phone, description
3. **Scrapes the target website** for context
4. **Sends content to an LLM** which identifies pain points, infers the decision maker, and writes a personalised cold email
5. **Saves the lead** to a Google Sheet with full metadata
6. **Sends the email** via SMTP automatically
7. **Logs completion** with timestamp and status

---

## Architecture

```
⏰ Daily Trigger
    └── ⚙️ Search Config Generator
            └── 🗺️ Google Maps Search (Places API)
                    └── 📋 Extract Business List
                            └── 🔍 Fetch Place Details
                                    └── 🔗 Merge Business Details
                                            └── 🌐 Scrape Website Content
                                                    └── 🧹 Clean Website Text
                                                            └── 🤖 LLM: Analyse & Write Outreach
                                                                    └── 📊 Parse LLM Output
                                                                            ├── 📑 Save to Google Sheets
                                                                            └── 📧 Send via SMTP
                                                                                    └── ✅ Log & Complete
```

---

## Tech Stack

| Component | Tool |
|---|---|
| Workflow Engine | n8n |
| Business Discovery | Google Maps Places API |
| Website Scraping | n8n HTTP Request node |
| AI Analysis & Copywriting | OpenAI GPT-4o-mini (swappable) |
| CRM / Lead Storage | Google Sheets |
| Email Delivery | SMTP (any provider) |

---

## Prerequisites

Before importing the workflow, you need:

- [ ] **n8n** instance running (self-hosted or n8n.cloud)
- [ ] **Google Maps API key** with Places API enabled
- [ ] **Google Sheets** credential set up in n8n (OAuth2)
- [ ] **OpenAI API key** set up in n8n credentials
- [ ] **SMTP credentials** (Gmail App Password, Brevo, Mailgun, etc.)
- [ ] A **Google Sheet** created with the columns listed below

---

## Setup Instructions

### Step 1 — Import the Workflow

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Upload `workflow/agent_01_b2b_acquisition.json`

### Step 2 — Configure Credentials

Set up the following in **n8n → Settings → Credentials**:

| Credential Name | Type | Notes |
|---|---|---|
| `Google Maps API` | HTTP Header Auth | Header: `key`, Value: your API key |
| `Google Sheets OAuth2` | Google OAuth2 | Enable Sheets + Drive scope |
| `OpenAI` | OpenAI API | Your OpenAI API key |
| `SMTP` | SMTP | Host, port, user, password |

### Step 3 — Configure the Workflow

Open the **⚙️ Search Config Generator** node and edit:

```js
const CONFIG = {
  searchQueries: [
    "e-commerce store",
    "online clothing store",
    "Shopify store"
  ],
  locations: [
    "London, UK",
    "Manchester, UK",
    "New York, USA"
  ],
  leadsPerRun: 10
};
```

Change `searchQueries` and `locations` to match your target niche and market.

### Step 4 — Set Your Google Sheet ID

1. Create a new Google Sheet
2. Name the first tab `Leads`
3. Add these column headers in row 1:

| Business Name | Website | Address | Phone | Rating | Pain Points | Decision Maker | Email Subject | Email Body | Confidence Score | Status | Created At | Follow Up Count |
|---|---|---|---|---|---|---|---|---|---|---|---|---|

4. Copy the Sheet ID from the URL:  
   `https://docs.google.com/spreadsheets/d/`**`THIS_IS_YOUR_SHEET_ID`**`/edit`
5. Paste it into the **📑 Save to Google Sheets** node → `documentId` field

### Step 5 — Add Real Email Addresses

> ⚠️ The SMTP node currently has a placeholder `toEmail`. You need a way to find actual decision-maker emails.

**Recommended approach — Hunter.io API:**

Add an HTTP Request node between **📊 Parse LLM Output** and **📧 Send via SMTP**:

```
GET https://api.hunter.io/v2/domain-search
  ?domain={extracted domain from website URL}
  &api_key={your hunter.io key}
```

Extract the first email from the response and pass it to the SMTP node.

**Free alternative:** Use the website's contact page URL as a fallback — add a scrape step targeting `/contact` or look for `mailto:` links in the scraped HTML.

### Step 6 — Activate

1. Toggle the workflow to **Active**
2. It will run automatically on the daily schedule
3. To test immediately: click **Execute Workflow** manually

---

## Google Sheet Columns Reference

| Column | Description |
|---|---|
| Business Name | Company name from Google Maps |
| Website | Homepage URL |
| Address | Full business address |
| Phone | Phone number if available |
| Rating | Google Maps star rating |
| Pain Points | LLM-identified operational pain points |
| Decision Maker | Inferred title (Founder, CEO, etc.) |
| Email Subject | AI-generated subject line |
| Email Body | Full personalised cold email |
| Confidence Score | LLM's confidence 0–100 |
| Status | Pending Send / Sent / Replied / No Email |
| Created At | ISO timestamp |
| Follow Up Count | Number of follow-up emails sent |

---

## Customisation

### Swap the LLM

The analysis node uses GPT-4o-mini by default. To use Claude (Anthropic):

1. Add an Anthropic credential in n8n
2. Replace the OpenAI node with an HTTP Request node calling:
   ```
   POST https://api.anthropic.com/v1/messages
   ```
3. Keep the same system prompt — it works with any capable LLM

### Change the Schedule

Open the **⏰ Daily Trigger** node and change the interval. Options:
- Every 12 hours: `hoursInterval: 12`
- Every Monday at 9am: switch to Cron mode → `0 9 * * 1`

### Follow-up Automation

To add automatic follow-ups:

1. Add a second scheduled workflow
2. Query Google Sheets for rows where `Status = Sent` and `Follow Up Count = 0` and `Created At` is older than 3 days
3. Use the LLM to write a follow-up email referencing the original
4. Send via SMTP and update `Follow Up Count` to 1

---

## Ethical Usage

- Only target **publicly listed businesses** discoverable via Google Maps
- Include an **unsubscribe option** in every email
- Comply with **CAN-SPAM** (US) and **GDPR** (UK/EU) regulations
- Do not send more than **1 email per business per week**
- Recommended daily send limit: **50 emails/day max** when starting out

---

## File Structure

```
agent-01-b2b-acquisition/
├── workflow/
│   └── agent_01_b2b_acquisition.json   ← Import this into n8n
├── docs/
│   └── architecture.md                 ← Detailed node-by-node breakdown
├── assets/
│   └── workflow_preview.png            ← Screenshot of the workflow canvas
└── README.md                           ← You are here
```

---

## Built By

**Elixr Co** — AI Chatbot Development & Workflow Automation  
📍 Islamabad, Pakistan · Serving US/UK e-commerce companies  
🌐 [Portfolio](https://sumair-portfolio-wbo9.vercel.app) · [LinkedIn](https://www.linkedin.com/in/sumair-ul-hassan-065544260)
