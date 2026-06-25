# Global Supply Chain Optimization Agent (n8n Workflow)

This directory contains the n8n workflow and setup guide for an **Autonomous Global Supply Chain Optimization Agent**. It audits logistics inventory levels, correlates shipping transit delays, evaluates alternative supplier pricing/speed catalog, and automatically drafts reorder alerts and cost-optimized mitigation plans.

---

## Triage & Optimization Logic

The agent operates in four distinct phases:

1.  **Ingestion (ERP simulation)**: Ingests data from a Google Sheet acting as the corporate ERP ledger, fetching three separate tables: `Inventory`, `Shipments` (active transit), and `Suppliers` (alternative catalog).
2.  **Shortage Predictor (Local JS Code)**:
    *   **Days of Cover (DoC)**: Computes `Current Stock / Daily Consumption` to forecast how many days of supply remain.
    *   **Reorder Check**: Triggers if `Days of Cover <= Lead Time + Safety Stock`.
    *   **Delay Correlation**: Scans active transits; if a shipment for a low-stock item is marked as `DELAYED`, it flags a high risk of stockout.
3.  **Alternative Sourcing Evaluation (LLM)**: Takes the at-risk item profile, details of the delayed route, and the alternative supplier list. Weighs unit cost, reorder minimums, delivery times, and geo-political/transit risk to select the optimal backup supplier.
4.  **Mitigation Plan & Alerting**: Logs reorder recommendations to the sheet and emails an emergency **Supply Chain Mitigation Report** to the operations team to execute the reorder.

---

## 1. ERP Ledger Setup (Google Sheets)

Create a Google Sheet named **Supply Chain Ledger** with four worksheet tabs. Create the following exact column headers in row 1 of each tab:

### Worksheet 1: `Inventory`
| Column A | Column B | Column C | Column D | Column E | Column F |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | **Current Stock** | **Daily Consumption** | **Safety Stock** | **Current Supplier** | **Supplier Lead Time** |

### Worksheet 2: `Shipments`
| Column A | Column B | Column C | Column D | Column E | Column F |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | **Shipment ID** | **Quantity** | **Status** | **Location** | **ETA** |

### Worksheet 3: `Suppliers`
| Column A | Column B | Column C | Column D | Column E | Column F |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | **Supplier Name** | **Unit Cost** | **Lead Time** | **Min Order Qty** | **Reliability Rating** |

### Worksheet 4: `Mitigation Log`
| Column A | Column B | Column C | Column D | Column E | Column F | Column G | Column H | Column I |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Date** | **Item Name** | **Current Supplier** | **Delayed Shipment** | **Suggested Supplier** | **Recommended Qty** | **Cost Impact** | **Action Strategy** | **Status** |

---

## 2. n8n Import & Setup

1.  Log into your n8n dashboard.
2.  Click **Workflows** > **Add workflow**.
3.  Click the top-right menu and select **Import from File**.
4.  Upload the [`workflow_supply_chain.json`](./workflow_supply_chain.json) file.
5.  Configure credentials on flagged nodes:
    *   **OpenAI nodes**: Connect your OpenAI API key.
    *   **Google Sheets nodes**: Connect your Google Sheets OAuth2.
    *   **Gmail node**: Connect the account to send out alerts.
6.  Open the **Set Configuration** node (the 2nd node) and input:
    *   `operations_email`: The email address where logistics alerts will be sent.
7.  Open the Google Sheets nodes and link your **Document ID** (found in the URL of your Google Sheet).

---

## 3. Run & Customization

*   **Testing**: Add a test item like "Steel Rods" in your sheet. Set current stock low (e.g. 100), daily consumption high (e.g. 15), and active shipment status to `DELAYED`. Run the workflow and confirm the agent logs the switch to "EuroSteel Ltd" and sends the email report.
*   **Safety Stock Buffer**: You can adjust safety stock multiplier variables in the **Set Configuration** node (defaults to `1.2` or +20%) to adapt to high-volatility markets.
*   **Disruption Feeds**: In production, connect the **Transit Disruption Lookups** HTTP node to live APIs (e.g. Flexport API or marine traffic trackers) instead of the default mock endpoint.
