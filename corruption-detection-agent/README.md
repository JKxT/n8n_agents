# Government Corruption Detection Agent (n8n Workflow)

This directory contains the n8n workflow and setup guide for an **Autonomous Government Corruption Detection Agent**. It automates forensic auditing of public procurement contracts, evaluating transaction details and corporate registries to flag potential bid rigging, bid splitting, shell companies, and conflict of interest issues.

---

## Detection Capabilities

The agent performs three layers of analysis on every contract:

1.  **Quantitative Rules Engine (Local JS Code)**:
    *   **Bid Splitting**: Flags contracts whose total value is close to (90%–99.9%) a public procurement threshold (e.g., $100,000), which indicates an attempt to bypass open public bidding.
    *   **Suspicious Approval Speed**: Flags contracts awarded within 3 days of public advertisement (suggests a pre-selected winner).
    *   **Single-Bidder Anomaly**: Flags contracts in competitive categories that had only a single bidder.
2.  **Corporate Registry Audit (OpenCorporates API)**:
    *   **Newly Incorporated Suppliers**: Flags companies registered less than 90 days before the contract was awarded.
    *   **Inactive/Dissolved Status**: Flags winning bidders that are inactive, dissolved, or struck off.
    *   **Unregistered Shells**: Flags suppliers that cannot be found in national corporate registries.
3.  **Qualitative Forensic Audit (LLM Agent)**:
    *   Synthesizes transaction data and registry outputs to detect beneficial ownership risks, political exposure anomalies, and conflict of interest signals.

---

## 1. Procurement Ledger Setup (Google Sheets)

Create a Google Sheet named **Government Contract Ledger** with a sheet tab named `Contracts`. Create the following exact column headers in row 1:

| Column A | Column B | Column C | Column D | Column E | Column F | Column G | Column H | Column I | Column J | Column K | Column L | Column M |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Contract ID** | **Title** | **Department** | **Amount** | **Supplier Name** | **Published Date** | **Award Date** | **Bidders Count** | **Method** | **Risk Score** | **Risk Level** | **Risk Flags** | **Auditor Notes** |

*Note: Ensure spelling matches. Dates should be in `YYYY-MM-DD` format.*

---

## 2. API Integration Setup

1.  **OpenCorporates API Key**:
    *   Register for a free OpenCorporates developer API token at [OpenCorporates API](https://api.opencorporates.com).
    *   If you do not have an API key, the lookup node will default to a soft match search (though rate limits will apply).
2.  **OpenAI API Key**:
    *   Used for forensic audit reasoning and report drafting. Recommended model: `gpt-4o` or `gpt-4o-mini`.

---

## 3. n8n Import & Configuration

1.  Log into your n8n dashboard.
2.  Click **Workflows** > **Add workflow**.
3.  Click the top-right menu and select **Import from File**.
4.  Upload the [`workflow_corruption_detection.json`](./workflow_corruption_detection.json) file.
5.  Configure credentials on flagged nodes:
    *   **OpenAI nodes**: Connect your OpenAI API key.
    *   **Google Sheets nodes**: Connect your Google Sheets OAuth2 credential.
    *   **Gmail node**: Connect the account to send out alerts.
6.  Open the **Set Configuration** node (the 2nd node) and configure:
    *   `opencorporates_api_key`: Paste your OpenCorporates API token.
    *   `tender_threshold`: Set this to your local threshold requiring open public tenders (defaults to `100000` / $100k).
    *   `notification_email`: The email address where forensic alerts and reports will be sent.
7.  Open the Google Sheets nodes and paste your **Document ID** (extracted from the URL of your Google Sheet).

---

## 4. Run & Execution Flow

1.  Input contract records into your Google Sheet (e.g. some standard contracts, and a high-risk test: a $99,500 contract awarded 2 days after publication to a company registered last week).
2.  Click **Execute workflow** in n8n.
3.  The agent processes the sheet row-by-row.
4.  The risk score (0-100%) and risk flags are updated in the Google Sheet.
5.  If a contract scores **High Risk (>=75%)**:
    *   The agent drafts a complete Anti-Corruption Forensic Report in Markdown.
    *   An email notification is dispatched instantly to the `notification_email` address with the report.
