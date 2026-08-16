# Agentic Purchase Order Processing Workflow

An AI-powered Purchase Order (PO) processing workflow built with **n8n**, **Azure OpenAI**, **ChromaDB**, and **Google Workspace** integrations.

The workflow receives a Purchase Order through email, extracts structured information using an LLM, validates the extracted information against supplier/policy knowledge, and routes the result for automatic processing or human review.

## Architecture

```text
Incoming Email
      |
      v
  n8n Email Trigger
      |
      v
  Get Attachment
      |
      v
  Extract PO Information
      |
      v
  Azure OpenAI(GPT 5.4)
      |
      v
 Structured PO JSON
      |
      +--------RAG-----------+
      |                      |
      v                      v
 Supplier / Policy       Vector Search
 Data                     (ChromaDB)
      |                      |
      +----------+-----------+
                 |
                 v
          Validation / Decision
                 |
          +------+------+
          |             |
          v             v
       Approved     Human Review
          |             |
          |             |
          |             v
          |       Email Notification
          |             |
          v             v 
          +------+-------
                 |
                 v
             Google Sheets   
                        
```

## Workflow Overview

The workflow is designed around the following steps:

1. **Email Intake**
   - Monitors an email account for incoming Purchase Orders.
   - Retrieves the attached PO document.

2. **Document Processing**
   - Extracts relevant information from the Purchase Order.
   - Sends the extracted content to an LLM for structured data extraction.

3. **Structured Extraction**
   - Produces structured JSON containing fields such as:
     - PO number
     - Supplier
     - Customer
     - Currency
     - Items
     - Total amount
     - Payment terms
     - Delivery date
     - Approval status
     - Approval reason

4. **Knowledge Retrieval**
   - Uses a vector database to retrieve relevant supplier/policy information.
   - The retrieved information is used to validate the Purchase Order.

5. **Decision**
   - If the PO satisfies the required conditions, it can be processed automatically.
   - If information is missing, ambiguous, or conflicts with policy, it is routed for human review.

6. **Output**
   - Approved results can be recorded in Google Sheets.
   - Human-review cases trigger an email notification.

## Example Structured Output

```json
{
  "po_number": "PO-2026-00123",
  "supplier": {
    "name": "ABC Technologies Ltd",
    "country": "Singapore"
  },
  "customer": {
    "name": "XYZ Corporation",
    "country": "Sri Lanka"
  },
  "currency": "USD",
  "items": [
    {
      "description": "Laptop",
      "quantity": 10,
      "unit_price": 1500.0
    }
  ],
  "total_amount": 15000.0,
  "payment_terms": "Net 30",
  "delivery_date": "2026-09-15",
  "approval_status": "approved",
  "approval_reason": "Supplier is approved and payment terms comply with company policy."
}
```

## Technology Stack

| Component | Technology |
|---|---|
| Workflow orchestration | n8n |
| LLM | Azure OpenAI |
| Embeddings | Azure OpenAI Embeddings |
| Vector database | ChromaDB |
| Email | Gmail |
| Structured output | JSON |
| Result storage | Google Sheets |
| Document storage | Google Drive |

## Repository Structure

```text
.
├── n8n/
│   └── workflow.json
├── knowledge_base/
│   └── supplier_data.csv
├── sample_documents/
│   ├── clean_po.pdf
│   ├── policy_mismatch_po.pdf
│   └── ambiguous_po.pdf
├── .env.example
├── .gitignore
└── README.md
```

## Setup

### 1. Install n8n

Run n8n using your preferred installation method.

For a local installation:

```bash
npx n8n
```

Then open the n8n editor in your browser.

### 2. Import the workflow

1. Open n8n.
2. Select **Import from File**.
3. Select:

```text
n8n/workflow.json
```

4. Configure the required credentials.

> The workflow committed to this repository should be a sanitized export. Real API keys, OAuth tokens, passwords, and other secrets must never be committed to GitHub.

### 3. Configure credentials

The workflow requires credentials for the services used by the workflow, such as:

- Azure OpenAI
- Gmail
- ChromaDB
- Google Sheets
- Google Drive

Configure these credentials directly inside your n8n instance.

Do **not** place API keys or OAuth secrets directly inside the workflow JSON.

### 4. Configure the knowledge base

The knowledge base can contain supplier and procurement information.

Example:

```csv
supplier,approved,payment_terms,max_po_value,country
ABC Technologies,Yes,Net 30,50000,Singapore
XYZ Supplies,Yes,Net 60,100000,Sri Lanka
Test Corp,No,Net 30,25000,India
```

The data can be converted into embeddings and stored in the vector database for retrieval.

## RAG Flow

The RAG component is used to provide additional business context when validating a Purchase Order.

```text
Knowledge Base
      |
      v
   Chunk / Prepare Data
      |
      v
   Generate Embeddings
      |
      v
    ChromaDB
      |
      v
Retrieve Relevant Information
      |
      v
LLM / Validation
```

For example, if a Purchase Order contains:

```text
Payment Terms: Net 90
Supplier: ABC Technologies
```

and the retrieved supplier policy states:

```text
ABC Technologies
Allowed Payment Terms: Net 30
```

the workflow can identify the mismatch and route the PO for human review.

## Approval Logic

The workflow supports three possible outcomes:

### Approved

The PO contains sufficient information and complies with the available supplier/policy information.

```json
{
  "approval_status": "approved"
}
```

### Human Review

The PO contains ambiguous information, missing information, or a policy mismatch.

```json
{
  "approval_status": "human_review"
}
```

### Rejected

The PO does not satisfy the required business rules.

```json
{
  "approval_status": "rejected"
}
```

## Test Scenarios

The workflow can be demonstrated using three different Purchase Orders:

### 1. Clean Purchase Order

Expected result:

```text
Approved
→ Automatically processed
```

### 2. Policy Mismatch

Example:

```text
PO Payment Terms: Net 90
Supplier Policy: Net 30
```

Expected result:

```text
Human Review
→ Email notification
```

### 3. Ambiguous / Incomplete PO

Example:

```text
Missing supplier information
or
Unclear total amount
```

Expected result:

```text
Human Review
```

## Security

**Never commit secrets to this repository.**

Do not commit:

- Azure OpenAI API keys
- ChromaDB API keys
- Gmail OAuth tokens
- Google OAuth credentials
- Passwords
- Access tokens
- Refresh tokens
- Private certificates
- `.env` files containing real secrets

Use `.env.example` for configuration documentation and keep the actual `.env` file local.

Also sanitize n8n workflow exports before publishing them if they contain:

- Personal email addresses
- Credential IDs
- Google Drive IDs
- Google Sheet IDs
- Internal resource URLs
- n8n instance identifiers

## Disclaimer

This repository is a demonstration of an agentic AI workflow for Purchase Order processing. It is intended for development, testing, and demonstration purposes and should be reviewed and hardened before use in a production environment.

## Author

AI Engineer — Agentic AI / RAG / Automation
