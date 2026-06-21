# Agent 01 — Architecture & Node Reference

> Detailed breakdown of every node in the workflow.

---

## Node-by-Node Reference

### 1. ⏰ Daily Trigger
- **Type:** Schedule Trigger
- **Runs:** Every 24 hours
- **Purpose:** Kicks off the entire pipeline automatically
- **Config:** Change `hoursInterval` to adjust frequency

---

### 2. ⚙️ Search Config Generator
- **Type:** Code (JavaScript)
- **Purpose:** Generates search query + location combinations from your config
- **Output:** Array of `{ query, niche, location }` objects
- **Edit here:** Target niches, cities, and leads per run

---

### 3. 🗺️ Google Maps Search
- **Type:** HTTP Request → Google Places Text Search API
- **Endpoint:** `https://maps.googleapis.com/maps/api/place/textsearch/json`
- **Purpose:** Finds real businesses matching the search query
- **Returns:** List of places with basic info and `place_id`
- **Requires:** Google Maps API key with Places API enabled

---

### 4. 📋 Extract Business List
- **Type:** Code (JavaScript)
- **Purpose:** Flattens and normalises the Places API response
- **Drops:** Businesses without a `place_id`
- **Output:** Clean business objects ready for detail fetching

---

### 5. 🔍 Fetch Place Details
- **Type:** HTTP Request → Google Places Details API
- **Endpoint:** `https://maps.googleapis.com/maps/api/place/details/json`
- **Fields fetched:** `website`, `formatted_phone_number`, `editorial_summary`
- **Purpose:** Enriches each business with website URL and phone number

---

### 6. 🔗 Merge Business Details
- **Type:** Code (JavaScript)
- **Purpose:** Combines Place Details response with business object
- **Filter:** Drops any business with no website URL
- **Reason:** No website = can't scrape = LLM has no context = low quality email

---

### 7. 🌐 Scrape Website Content
- **Type:** HTTP Request
- **Purpose:** Fetches raw HTML from the business's homepage
- **continueOnFail:** `true` — workflow continues even if site is down or blocks the request
- **Timeout:** 10 seconds

---

### 8. 🧹 Clean Website Text
- **Type:** Code (JavaScript)
- **Purpose:** Strips HTML tags, scripts, and styles — extracts readable text
- **Limit:** First 3,000 characters (enough context for the LLM, keeps cost low)
- **Fallback:** If scraping failed, passes `"Could not extract website content."`

---

### 9. 🤖 LLM: Analyse & Write Outreach
- **Type:** OpenAI node (swappable)
- **Model:** `gpt-4o-mini`
- **System Prompt:** Instructs the LLM to act as a B2B sales analyst
- **Task:** Identify pain points, infer decision maker, write personalised cold email
- **Output format:** Strict JSON with keys: `pain_points`, `decision_maker_title`, `email_subject`, `email_body`, `confidence_score`

---

### 10. 📊 Parse LLM Output
- **Type:** Code (JavaScript)
- **Purpose:** Parses the LLM's JSON response safely
- **Fallback:** If JSON parsing fails, creates a safe fallback object for manual review
- **Output:** Final enriched lead object ready for storage and sending

---

### 11. 📑 Save to Google Sheets *(runs in parallel)*
- **Type:** Google Sheets → Append Row
- **Sheet tab:** `Leads`
- **Purpose:** Saves every lead with full metadata before sending email
- **Why first:** Ensures lead is logged even if email sending fails

---

### 12. 📧 Send via SMTP *(runs in parallel)*
- **Type:** Email Send (SMTP)
- **Purpose:** Sends the AI-written personalised email
- **From:** Your configured SMTP user
- **Reply-To:** `sumair@elixrco.ai`
- **Note:** You must supply a real `toEmail` — see README for Hunter.io integration

---

### 13. ✅ Log & Complete
- **Type:** Code (JavaScript)
- **Purpose:** Console logs the completed lead for debugging
- **Output:** Final object with `status: Sent` and `sent_at` timestamp

---

## Data Flow Summary

```
CONFIG input
    ↓
Google Maps → N businesses found
    ↓
Place Details → Website URLs fetched
    ↓ (drop no-website leads)
Website Scrape → Raw text extracted
    ↓
LLM Analysis → Pain points + personalised email generated
    ↓
         ┌────────────────────────────────┐
         ↓                                ↓
  Google Sheets row saved          Email sent via SMTP
         └────────────────────────────────┘
                        ↓
                  Lead logged ✅
```

---

## Cost Estimate (per run)

| Service | Usage | Estimated Cost |
|---|---|---|
| Google Maps Places API | 10 searches + 10 detail lookups | ~$0.05 |
| Website scraping | 10 HTTP requests | Free |
| GPT-4o-mini | ~1,500 tokens per lead × 10 leads | ~$0.03 |
| SMTP | 10 emails | Free (most providers) |
| **Total per run** | | **~$0.08** |

At daily runs: **~$2.40/month**

---

## Known Limitations

1. **Email finding** — The workflow does not auto-find decision-maker email addresses. Hunter.io integration is needed (see README).
2. **Anti-scraping** — Some websites block automated requests. The node continues on fail but those leads get weaker LLM context.
3. **Rate limits** — Google Maps Places API has per-second limits. Add a `Wait` node (1–2s delay) between Place Details calls for large batches.
4. **LLM hallucination** — Low confidence scores (below 60) should be reviewed manually before sending.
