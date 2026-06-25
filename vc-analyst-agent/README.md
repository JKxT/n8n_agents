# AI Venture Capital Analyst Agent (n8n Workflow)

This directory contains the n8n workflow and setup guide for an **AI Venture Capital Analyst Agent**. It automates the deal-flow screening process for early-stage venture capital firms, performing competitor checks, founder audits, traction assessments, pipeline logging, and producing comprehensive investment memos.

---

## Agent Capabilities

The workflow runs a structured early-stage VC evaluation pipeline:

1.  **Lead Sourcing (Trigger)**: Receives startup details from webhook submissions (e.g., Typeform/Fillout pitch forms) or manual inputs.
2.  **Market & Competitor Check**: Calls the **Google Serper API** to search for competitor listings based on the startup's product description.
3.  **Team & Traction Audit (LLM)**: Evaluates the founders' backgrounds (pedigree, ex-employers, technical capability) and submitted metrics (MRR, growth rates, churn, design pilots). Outputs a score out of 10 for both.
4.  **Market Opportunity Evaluation (LLM)**: Analyzes search results to build competitive matrixes, estimating TAM/SAM/SOM and grading market opportunity and competitive advantages out of 10.
5.  **Weighted VC Scoring Engine (Code node)**: Calculates a final deal rating (1.0 to 10.0) based on customizable investment thesis weights.
6.  **Investment Memo Compilation (LLM)**: Generates a full Venture Capital Investment Memo in Markdown (including Recommendation, Exec Summary, Team Audit, Financial Traction, Market and Risks).
7.  **Pipeline Logging**: Appends all evaluation metrics, verdicts, and summaries to your CRM Deal Ledger (Google Sheets).
8.  **Priority Alerts**: Instantly emails partners when a startup scores **>= 8.0/10 (DEEP_DIVE verdict)**.

---

## 1. CRM Deal Pipeline Setup (Google Sheets)

Create a Google Sheet named **VC Deal Flow** with a sheet tab named `Deals`. Set the exact column headers in row 1:

| Column A | Column B | Column C | Column D | Column E | Column F | Column G | Column H | Column I | Column J |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Date** | **Startup** | **Description** | **Founders** | **Traction** | **Ask** | **Score** | **Verdict** | **Memo Summary** | **Status** |

---

## 2. API Key Configuration

1.  **Google Serper API Key**:
    *   Register for a free account at [Serper.dev](https://serper.dev/) (includes 2,500 free search queries).
    *   If no key is configured, search checks will fail gracefully and the workflow will default to standard LLM market assumptions.
2.  **OpenAI API Key**:
    *   Used for evaluation and memo drafting. Recommended model: `gpt-4o`.

---

## 3. n8n Import & Setup

1.  Log into your n8n dashboard.
2.  Click **Workflows** > **Add workflow**.
3.  Click the top-right menu and select **Import from File**.
4.  Upload the [`workflow_vc_analyst.json`](./workflow_vc_analyst.json) file.
5.  Configure credentials on flagged nodes:
    *   **OpenAI nodes**: Connect your OpenAI API key.
    *   **Google Sheets nodes**: Connect your Google Sheets OAuth2.
    *   **Gmail node**: Connect the account to send out deal alerts.
6.  Open the **Set Configuration** node (the 3rd node) and input:
    *   `serper_api_key`: Paste your Serper API token.
    *   `partner_email`: Set to your investment team's distribution list.
    *   *Adjust weights if desired* (`weight_team`, `weight_traction`, etc.).
7.  Open the Google Sheets nodes and link your **Document ID**.

---

## 4. Customizing the Investment Thesis

You can easily adapt the agent's screening rules to fit your specific investment thesis:

*   **Adjust Weighting**: In the **Set Configuration** node, change the decimals (e.g. increase `weight_traction` to `0.5` and decrease `weight_team` to `0.15` if you are a growth-stage investor prioritizing numbers over founding pedigree).
*   **Sector Prompts**: If you invest primarily in Biotech or Web3, edit the system prompts inside the **Evaluate Team & Traction** and **Evaluate Market & Competition** nodes to specify sector-specific milestones (e.g., FDA clearance progress or tokenomics utility).
*   **Verdict Thresholds**: In the **Calculate Deal Score** code node, you can adjust the numerical limits for `DEEP_DIVE` (default `8.0`) and `SCREENING_CALL` (default `5.5`).
