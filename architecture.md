# Architecture: AI-Human Hybrid Customer Support Management

This document outlines the architectural design of the AI-Human Hybrid Customer Support Management system, built using **n8n**.

## 1. System Overview

The system is designed to automate the initial stages of customer support while keeping a human expert in the loop for final approval. It integrates multiple communication channels, uses AI for ticket classification and drafting responses (using RAG), and provides a seamless interface for human agents to review and send replies.

### High-Level System Map
```mermaid
graph TD
    A[Customer Channels] --> B{n8n Orchestrator}
    B --> C[Google Sheets - Data Logging]
    B --> D[AI Engine - Gemini/RAG]
    D --> E[Knowledge Base - Pinecone]
    D --> F[Live Data - WooCommerce]
    B --> G[Human Review - Slack]
    G --> H[Final Response]
    
    style B fill:#f96,stroke:#333,stroke-width:2px
    style D fill:#69f,stroke:#333,stroke-width:2px
    style G fill:#6f9,stroke:#333,stroke-width:2px
```

## 2. Detailed Workflow

The workflow is divided into three main stages, ensuring data integrity, AI accuracy, and human oversight.

### Full Operational Flow
```mermaid
graph LR
    subgraph Stage1[Stage 1: Intake & Processing]
    G1[Gmail] --> N[Normalize]
    W1[WhatsApp] --> N
    T1[Typeform] --> N
    N --> V[Validate & De-duplicate]
    V --> L1[Log to Sheets]
    V --> AK[Auto-Ack to Customer]
    end

    subgraph Stage2[Stage 2: AI & RAG Analysis]
    L1 --> AI[AI Support Agent]
    AI <--> P1[Pinecone Knowledge Base]
    AI <--> WC1[WooCommerce API]
    AI --> AL[Log AI Reasoning]
    end

    subgraph Stage3[Stage 3: Human-in-the-Loop]
    AI --> SL[Slack Notification]
    SL -- Approve --> S1[Send Reply]
    SL -- Edit --> EF[HTML Edit Form]
    EF --> S1
    S1 --> RS[Resolve & Log Resolution]
    end
    
    style Stage1 fill:#f5f5f5,stroke:#333,stroke-dasharray: 5 5
    style Stage2 fill:#e1f5fe,stroke:#01579b
    style Stage3 fill:#e8f5e9,stroke:#2e7d32
```

### Stage 1: Ticket Intake and Pre-processing
*   **Triggers:** The system listens for new support requests from:
    *   **Gmail:** Support inbox polling.
    *   **Typeform:** Form submissions.
    *   **WhatsApp:** Business API messages.
    *   **Generic Webhook:** For Tally forms or other custom integrations.
*   **Normalization:** A unified schema is created regardless of the source.
*   **Validation:** Basic checks for message length and content.
*   **Duplicate Detection:** Checks Google Sheets for recent open tickets from the same customer to avoid redundant processing.
*   **Logging:** New tickets are logged into the **Google Sheets ("Tickets" sheet)**.
*   **Acknowledgement:** An automated "We received your request" message is sent back to the customer via the original channel.

### Stage 2: AI Analysis & RAG (Retrieval-Augmented Generation)
*   **AI Agent:** Uses **LangChain** with **Google Gemini** as the LLM.
*   **Knowledge Retrieval (RAG):** The agent queries a **Pinecone** vector database (Knowledge Base) for relevant policies and FAQs.
*   **Tool Integration:** The agent can check real-time data:
    *   **WooCommerce Orders:** To check order status and history.
    *   **WooCommerce Products:** To get product details.
*   **Analysis:** The AI classifies the ticket category, assigns an urgency level (1-3), and drafts a professional response based on retrieved facts.
*   **Escalation Logic:** If the urgency is "Critical" (Urgency 1), a high-priority alert is sent to Slack.

### Stage 3: Human-in-the-Loop Review
*   **Slack Interaction:** The AI's analysis and draft reply are sent to a Slack channel with interactive buttons (**Send**, **Edit**, **Escalate**).
*   **Edit Form:** If the agent chooses "Edit", a custom HTML form (served via n8n webhook) allows them to modify the reply.
*   **Final Action:** Once approved or edited, the reply is sent to the customer via Gmail or WhatsApp.
*   **Resolution:** The ticket status is updated to "Resolved" in Google Sheets, and a final log is created in the "Resolutions" sheet.

## 3. Tech Stack

*   **Automation Engine:** [n8n](https://n8n.io/)
*   **Large Language Model (LLM):** Google Gemini (via LangChain)
*   **Vector Database:** Pinecone (for RAG)
*   **Data Storage:** Google Sheets
*   **Communication:** Slack (Internal), Gmail & WhatsApp (External)
*   **E-commerce Integration:** WooCommerce (REST API)

## 4. Data Model (Google Sheets)

The system uses a Google Spreadsheet with the following sheets:
*   **Tickets:** Main log of all incoming requests and their current status.
*   **AI Log:** Detailed logs of AI reasoning, tools used, and confidence scores.
*   **Resolutions:** Tracking when and how tickets were resolved.
*   **Error Log:** Logs from the dedicated Error Workflow.

## 5. Error Handling & Observability

A dedicated **Error Workflow** handles any failures within the main process, ensuring high availability and notification of critical issues.

### Error Resolution Flow
```mermaid
graph TD
    E1[Node Failure] --> ET[Error Trigger]
    ET --> FE[Format Error Context]
    FE --> SC{Is Critical?}
    SC -- Yes --> SL[Slack #support-errors]
    SC -- Yes --> EM[Email Admin]
    SC -- No --> SL
    FE --> GS[Log to Error Log Sheet]
    
    style ET fill:#ffcdd2,stroke:#b71c1c
    style SC fill:#fff9c4,stroke:#fbc02d
```

*   **Error Trigger:** Catches node failures.
*   **Severity Assessment:** Categorizes errors as "CRITICAL" or "WARNING" based on the failed node (e.g., failures in sending replies are critical).
*   **Multi-channel Alerts:** Sends notifications via Slack (#support-errors) and Email to the administrator.
*   **Logging:** All errors are persisted in the "Error Log" sheet for auditing.
