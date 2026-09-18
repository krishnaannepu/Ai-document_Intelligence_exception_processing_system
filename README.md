# AI Document Intelligence & Exception Processing System

An end-to-end AI-powered invoice exception processing system. It takes a supplier invoice from upload to either automatic approval or a human review queue, with almost no manual work in between.

## Business Problem Solved

Finance and accounts payable teams get supplier invoices by email, portal, or shared folder, often with missing or incorrect information:

- No PO number
- No invoice number
- Missing supplier name
- Wrong totals
- Duplicate invoice numbers

Most companies still catch these by hand, using spreadsheets and email. That's slow, error-prone, and hard to track. This system replaces that manual check with an automated pipeline that reads each document, validates it, and decides whether to approve it or send it for review.

## What this system does

1. A finance admin uploads an invoice through a web dashboard.
2. The file is stored in Google Drive.
3. An n8n workflow detects the new file automatically.
4. Google's Gemini AI extracts the invoice fields (supplier, amounts, dates, PO number, etc.) straight from the document image — no manual typing.
5. The extracted data is validated against business rules and written to Google Sheets, which acts as the system's database.
6. Each invoice is classified as **Approved** or **Needs Review**.
7. A dashboard (built with Lovable) shows every invoice's status.
8. If an invoice needs review, an email notification goes out automatically, and the sheet tracks whether that notification was already sent (so no duplicate emails).
9. **Exception handling extension:** two categories of exceptions are escalated to a separate AWS-based decision service for a second opinion — see below.

```
Upload (Lovable UI)
   -> Google Drive
   -> n8n detects new file
   -> Gemini AI extracts invoice fields
   -> Validation rules run in Google Sheets
   -> Approved  ──────────────────────────────► Dashboard
   -> Needs Review
        -> AWS exception service (see below) decides: approve / retry / escalate
        -> Gmail notification to finance admin
        -> Dashboard
```

## AWS exception-handling extension

The base pipeline's duplicate check only compared invoice numbers, which meant two *different* companies that happened to reuse the same invoice number got wrongly flagged as duplicates. Low-confidence AI extractions also had no automatic second attempt.

This is fixed by a small, separate AWS Lambda service that the n8n workflow calls only for these two cases. It's kept in its own repository so it stays focused and easy to review:

**[invoice-exception-service](https://github.com/krishnaannepu/invoice-exception-service)** — uses Amazon Bedrock (Titan embeddings) to compare supplier names by meaning rather than exact text, LangGraph to route the decision logic, and GitHub Actions to automatically run its test suite on every change.

The main pipeline above is untouched by this extension — it's an additional branch, not a replacement.

## Tech stack

n8n (automation/orchestration), Google Gemini (document AI extraction), Google Sheets (data store + business rules), Google Drive (file storage), Lovable (dashboard UI), Gmail (notifications), AWS Lambda + Amazon Bedrock + LangGraph (exception decision service), GitHub Actions (CI for the exception service).

## Repository contents

| Folder | Contents |
|---|---|
| `01_project_planning` | Scope and architecture notes |
| `02_sample_invoices` | Sample PDF invoices used for testing |
| `03_google_sheets_schema` | The Google Sheet (data store + dashboard + validation rules) and setup screenshots |
| `04_prompts` | The exact prompts used for Lovable (UI generation) and Gemini (invoice extraction) |
| `05_validation_rules` | Business validation logic |
| `06_ui_notes` | Dashboard and review-modal requirements |
| `07_screenshots` | Screenshots of the working system (dashboard, review flow, notifications) |
| `08_final_submission` | Full project write-up |
| `10_test_invoices` | Test documents built specifically to exercise the AWS exception service (duplicate false positives, genuine duplicates, low-confidence scans) |

The AWS exception service itself (`09_lambda_exception_service`) lives in its own repository, linked above.

## Author

Krishnachaitanya Annepu
