# Autonomous Cyber Defense Agent (n8n SOAR Workflow)

This directory contains the n8n workflow and setup guide for an **Autonomous Cyber Defense Agent**. It acts as an automated SOAR (Security Orchestration, Automation, and Response) playbook, triaging security events, checking IP whitelists, querying threat intelligence databases, executing active containment rules, and generating full forensic incident reports.

---

## Capabilities & Architecture

The workflow implements an automated security response cycle:

1.  **Ingestion (Trigger)**: Listens for webhook posts from SIEM engines (e.g. Splunk, Datadog, AWS CloudWatch) or takes manual log entries.
2.  **Safety Guard (Whitelist)**: Runs incoming IPs against a local whitelist array (excluding internal subnets like `10.0.0.0/8`, `192.168.0.0/16`, and local loopbacks) to prevent accidental denial of service on your own systems.
3.  **Threat Intelligence Lookup (AbuseIPDB)**: Queries AbuseIPDB API to check the reputation, abuse score (0-100%), country, and usage type of the source IP.
4.  **Triage (LLM SOC Analyst)**: A `gpt-4o-mini` node reviews the log message, alert name, and threat intelligence metadata to grade severity (Critical, High, Medium, Low) and determine if it is a false alarm.
5.  **Active Containment (Response)**:
    *   *If Severity is Critical/High*:
        *   If action is `block_ip`: Calls the **Cloudflare WAF API** to block the IP in your firewall rules.
        *   If action is `suspend_user`: Calls the **Okta Identity API** to suspend the user account.
    *   *If Severity is Medium/Low*: Logs the event and flags it for audit without taking autonomous actions.
6.  **Incident Reporting**: An LLM compiles the logs, threat intel, and containment logs into a detailed markdown Cyber Security Incident Report (CSIR).
7.  **Alerting & Ticketing**: Dispatches an email notification to the SecOps team with the report and appends a record to a central Google Sheets incident ticket queue.

---

## 1. Ticket Database Setup (Google Sheets)

Create a Google Sheet named **SOC Security Tickets** with a sheet tab named `Incidents`. Set the exact column headers in row 1:

| Column A | Column B | Column C | Column D | Column E | Column F | Column G | Column H | Column I |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Timestamp** | **Incident ID** | **Alert Name** | **Source IP** | **User Account** | **Severity** | **Containment Action** | **Status** | **Incident Report** |

*Note: Ensure spelling matches. Google Sheets OAuth2 needs permission to edit this sheet.*

---

## 2. API Key Setup

1.  **AbuseIPDB API Key**:
    *   Register for a free account at [AbuseIPDB](https://www.abuseipdb.com/) and generate an API key.
    *   If no key is configured, the workflow will continue on fail and default to general LLM reasoning.
2.  **OpenAI API Key**:
    *   Used for SOC triage and report drafting.
3.  **Active Containment Credentials**:
    *   **Cloudflare**: Needs your Cloudflare Zone ID, API Key, and email account.
    *   **Okta**: Needs your Okta Org URL domain.

---

## 3. n8n Import & Configuration

1.  Log into your n8n dashboard.
2.  Click **Workflows** > **Add workflow**.
3.  Click the top-right menu and select **Import from File**.
4.  Upload the [`workflow_cyber_defense.json`](./workflow_cyber_defense.json) file.
5.  Configure credentials on flagged nodes:
    *   **OpenAI nodes**: Connect your OpenAI API key.
    *   **Google Sheets nodes**: Connect your Google Sheets OAuth2.
    *   **Gmail node**: Connect the email sender credentials.
6.  Open the **Set Settings** node (the 3rd node) and input:
    *   `abuseipdb_api_key`: Paste your AbuseIPDB key.
    *   `admin_email`: Set to your SOC team email address.
    *   `cloudflare_zone_id` and `cloudflare_api_key`: Configure for firewall actions.
7.  Open the Google Sheets nodes and link your **Document ID**.

---

## 4. Testing the Agent (Dry-Run Mode)

To verify the agent without triggering actual network blocks:
1.  **Disable** the `Cloudflare WAF Block IP` and `Okta Suspend User` HTTP Request nodes (right-click -> select *Disable*), or keep the URLs pointed to the default mock endpoints.
2.  Double-click the **Manual Trigger Mock** node and review the mock event parameters:
    *   `source_ip`: Set to a known malicious IP (e.g. `118.25.10.15`) or use the default test IP (`198.51.100.42`).
    *   `alert_name`: Set to `"Multiple Failed SSH Logins"`.
3.  Click **Execute workflow** in n8n.
4.  Review the execution history:
    *   See the AbuseIPDB score.
    *   Inspect the LLM SOC Analyst's triage decisions.
    *   Confirm it routes to the correct containment channel.
    *   Verify that a Markdown Incident Report is compiled and logged in Google Sheets.
