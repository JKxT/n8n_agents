# AI Software Development Company (n8n Multi-Agent Workflow)

This directory contains the n8n workflow and setup guide for an **AI Software Development Company Agent**. It implements a complete autonomous team of agents (CEO, Product Manager, Developer, QA, and DevOps) to compile software from a single prompt.

---

## Team of Agents Overview

The workflow models a structured corporate pipeline where each agent has specialized instructions and roles:

1.  **CEO Agent (LLM)**:
    *   **Role**: Creates technical roadmaps.
    *   **Task**: Analyzes the user's idea, outlines the architecture, recommends a tech stack, and sets up high-level milestones.
2.  **Product Manager Agent (LLM)**:
    *   **Role**: Writes requirements.
    *   **Task**: Translates the CEO's roadmap into a detailed markdown Product Requirements Document (PRD) with user stories, functional requirements, and layout schemas.
3.  **Developer Agent (LLM)**:
    *   **Role**: Writes code.
    *   **Task**: Translates the PRD into clean, working, self-contained source code. For web applications, it outputs a single HTML file containing CSS and JavaScript.
4.  **QA Agent (LLM)**:
    *   **Role**: Tests and audits code.
    *   **Task**: Performs static analysis against the PM's PRD. Checks for bugs, security vulnerabilities, and missing features. Outputs a QA Report and sets a `PASS` or `FAIL` status.
5.  **Developer Fix Agent (LLM)**:
    *   **Role**: Recursive bug-fixer.
    *   **Task**: Active only if QA outputs `FAIL`. Rewrites the code incorporating the QA report's instructions.
6.  **DevOps Agent (Code & File Node)**:
    *   **Role**: Deploys.
    *   **Task**: Converts the final code into binary format so you can download it directly from the n8n UI, and writes it to your local filesystem.

---

## Prerequisites

1.  **n8n Instance**: Cloud or self-hosted.
2.  **OpenAI API Key**: Connected as an OpenAI Credential in n8n.
    *   *Note: We recommend using `gpt-4o` for Developer and QA nodes for advanced coding capabilities, though `gpt-4o-mini` can be substituted to reduce costs.*
3.  **Filesystem Access (Optional)**:
    *   If using self-hosted n8n, make sure n8n has write permissions to write files to the target workspace or output directory. You can set the environment variable:
        `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=false`
    *   If running inside Docker, ensure the volume `/out` is mounted.

---

## 1. n8n Import Instructions

1.  Log into your n8n dashboard.
2.  Click **Workflows** > **Add workflow** (or click the plus icon in the top right).
3.  Click the three dots menu in the top right corner and select **Import from File**.
4.  Upload the [`workflow_dev_company.json`](./workflow_dev_company.json) file.

---

## 2. Configuration & Credentials

Once imported, configure credentials for:
*   **OpenAI Agent Nodes**: Click on each of the OpenAI nodes (CEO, PM, Developer, QA, and Developer Fix) and link them to your **OpenAI API Key**.
*   **Save File to Disk Node**:
    *   If you do not want to save files locally on your machine and just want to download them via the browser, you can **Disable** or ignore the `Save File to Disk` node. The workflow will still run and present a download link on the `Prepare Binary File` node.
    *   If enabled, configure the output path in the `File Name` property (defaults to `/out/{{ $json.filename }}`).

---

## 3. Running the Agency

1.  Double-click the **Set Initial State** node (the 2nd node).
2.  Change the value of `software_idea` to your desired application. Example ideas:
    *   *"A retro Snake game with local high score storage, grid layouts, and sound alerts."*
    *   *"A custom dark-mode calculator with scientific functions and an audit history panel."*
    *   *"A simple task manager where tasks can be dragged, edited, and saved to localStorage."*
3.  Click **Execute workflow** in the bottom bar.
4.  Once completed, go to the **Prepare Binary File** node, click on the **Binary** tab in the output panel, and click **Download** to save the working code file (e.g., `index.html`) directly to your machine.

---

## 4. Customizing the Agency Tech Stack

By default, the **Developer Agent** is told to output single-file HTML apps to make the deployment and direct-download seamless. 

To create backend scripts (e.g. Python scripts or Node scripts), change the prompt inside the **Developer Agent (LLM)** node:
*   Change: *"If it is a frontend app, bundle everything into a single HTML file."*
*   To: *"Write a single self-contained Python script using standard libraries, saving it as app.py."*
*   Remember to update the `filename` key in the Developer's output JSON parsing code.
