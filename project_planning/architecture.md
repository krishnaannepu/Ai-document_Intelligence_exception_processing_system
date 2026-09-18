# System Architecture

This explains how one supplier invoice moves through the system, from upload to final resolution, and why each step exists.

See `architecture_diagram.md` in this same folder for a visual flow chart of this process.

## 1. Upload

The Finance Admin uploads a supplier invoice (PDF, JPG, or PNG) through the Lovable dashboard. The file is stored in Google Drive.

## 2. Detection

An n8n workflow watches that Google Drive folder and automatically picks up each new file. No one has to manually start the process.

## 3. Field extraction

n8n sends the document to Google Gemini, which reads the invoice image and returns structured data: supplier name, invoice number, invoice date, PO number, subtotal, VAT/tax, and total amount. This replaces manual data entry entirely.

## 4. Validation

The extracted data is written to Google Sheets, which acts as the system's database. Formula-driven business rules check:

- All required fields are present
- A PO number exists
- Subtotal + VAT/Tax equals the Total Amount
- The invoice number hasn't already been used by a different invoice

If every check passes, the invoice is marked **Approved**. If any check fails, it's marked **Needs Review**, with a specific Exception Reason recorded.

## 5. Exception handling (AWS extension)

Two specific kinds of "Needs Review" cases get a second, automated opinion from a separate AWS Lambda service, instead of going straight to a human:

- **Possible duplicate invoice.** The check in step 4 only compares invoice numbers, so two unrelated companies that happen to reuse the same invoice number would otherwise be wrongly flagged as duplicates of each other. The Lambda service uses Amazon Bedrock to compare the two supplier names by meaning, not just spelling, and only treats it as a real duplicate if the names actually match.
- **Low-confidence extraction.** If Gemini wasn't confident about what it read — for example, a blurry scan — the case is flagged for a second look instead of being silently accepted or silently rejected.

Inside the Lambda function, a small decision graph (built with LangGraph) decides the outcome: approve, retry, or escalate to a human. That decision is written back to Google Sheets as the invoice's Final Status.

This extension only runs for these two specific cases. Every other exception (missing field, total mismatch, and so on) goes straight to manual review, exactly as before — the AWS piece adds a new branch, it doesn't replace or change the base system.

## 6. Review and resolution

The dashboard reads directly from Google Sheets and shows every invoice's current status. For anything still needing review, the Finance Admin opens the record, compares it with the original document, corrects values if needed, adds review notes, and marks it resolved.

## 7. Notification

When an invoice is marked Needs Review, Gmail sends the Finance Admin an automatic notification. The sheet tracks whether that notification was already sent, so nobody gets emailed twice about the same invoice.
