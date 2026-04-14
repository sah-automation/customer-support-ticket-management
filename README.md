# AI-Human Hybrid Customer Support Management (n8n)

This project is an advanced, production-ready **n8n workflow** that manages customer support tickets from multiple channels (Gmail, WhatsApp, Typeform, Tally, and custom webhooks). It leverages **AI (Google Gemini)** with **Retrieval-Augmented Generation (RAG)** to classify issues, check order statuses in **WooCommerce**, and draft professional responses for human review.

## 🚀 Key Features

*   **Omnichannel Support:** Unified ticket handling for Email, WhatsApp, and Webhooks.
*   **AI-Driven RAG:** Uses a Pinecone vector store to retrieve product knowledge and support policies.
*   **Real-time E-commerce Integration:** Automated lookups for WooCommerce orders and products.
*   **Intelligent Classification:** Automatic categorization and urgency scoring (1-3).
*   **Human-in-the-Loop (HITL):** Final response review and editing via Slack.
*   **Automated Acknowledgements:** Instant confirmation messages sent back to customers.
*   **Duplicate Detection:** Prevents multiple AI processing for the same issue within 24 hours.
*   **Comprehensive Logging:** Full audit trail in Google Sheets (Tickets, AI Logs, Resolutions).
*   **Robust Error Handling:** Dedicated error workflow for critical failure notifications.

## 🛠️ Prerequisites

To run this workflow, you need:
*   [n8n](https://n8n.io/) (Desktop or Hosted)
*   **Google Cloud Account:** For Gmail and Google Sheets APIs.
*   **Pinecone API Key:** For the vector knowledge base.
*   **Google Gemini API Key:** For the AI agent.
*   **Slack Workspace:** For internal notifications and agent interactions.
*   **WooCommerce API Keys:** For order and product lookups.
*   **(Optional)** WhatsApp Business API/Meta Developer Account.

## 📦 Setup & Installation

1.  **Google Sheets:** Create a spreadsheet with the following sheets: `Tickets`, `AI Log`, `Resolutions`, `Error Log`.
2.  **Pinecone:** Initialize a Pinecone index (e.g., `ske-customer-support-n8n`) and upload your support documents (FAQs, policies).
3.  **n8n Import:**
    *   Download the `.json` files from the `workflow/` folder.
    *   Import `Customer Support Ticket Management.json` into n8n.
    *   Import `Customer Support Ticket Management - Error Workflow.json` and ensure it's set as the Error Workflow in the main workflow settings.
4.  **Credentials:** Set up and link your credentials in n8n for:
    *   Google Sheets & Gmail (OAuth2)
    *   Slack API
    *   Pinecone API
    *   Google Gemini (LangChain)
    *   WooCommerce REST API
5.  **Environment Variables:** Update any `N8N_BASE_URL` or Webhook paths in the Code nodes as needed.

## 📝 Workflow Stages

### Step 1: Intake & Processing
The workflow receives data from various sources, normalizes it, checks for duplicates, and logs it to Google Sheets. An automated acknowledgement is sent to the customer immediately.

### Step 2: AI Support Agent
The AI agent analyzes the ticket. It uses its tools to search the Knowledge Base (Pinecone) and check WooCommerce for relevant order/product data. It drafts a response and identifies if human escalation is required.

### Step 3: Human Review & Response
Human agents receive a notification in Slack. They can choose to:
*   **Send:** Approve the AI draft as-is.
*   **Edit:** Open a secure HTML form to modify the draft before sending.
*   **Escalate:** Mark the ticket for manual handling.

## 📸 Screenshots

The following screenshots illustrate the workflow logic:
*   **Step 1:** Intake & Normalization (`screenshot/Step1.png`)
*   **Step 2:** AI Agent Analysis (`screenshot/Step2.png`)
*   **Step 3:** Human-in-the-Loop Interaction (`screenshot/Step3.png`)
*   **Knowledge Base:** RAG Document Updater (`screenshot/RAG Doc Updater.png`)

## 🛡️ Error Handling

The system includes a dedicated error-handling workflow that monitors for critical failures (e.g., failed to send reply, failed to log ticket). It notifies the support team via Slack and Email, ensuring no customer request is lost due to technical issues.

---
*Created with ❤️ for efficient customer support.*
