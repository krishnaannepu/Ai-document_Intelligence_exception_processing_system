# AI Document Intelligence & Exception Processing System

## 1. Project Identity

Project Name: AI Document Intelligence & Exception Processing System

Current Use Case: Supplier Invoices

Core Reusable Architecture:

Document → Field Extraction → Business Rules → Pass/Exception → Human Review → Structured Record

## 2. Business Problem

Many organisations already use ERP, accounting, procurement, or e-invoicing systems to manage supplier invoices. However, existing workflows can still encounter incomplete, inconsistent, duplicated, incorrectly calculated, or unstructured documents that require manual investigation.

This system is designed as an intelligent layer around an existing business workflow rather than a replacement for ERP or finance platforms. It extracts key information from incoming documents, applies deterministic business rules, automatically clears straightforward cases, and routes only exceptions to a human for review.

## 3. Primary User

The primary user is a Finance Admin or Accounts Payable employee.

This user reviews supplier invoices, resolves exceptions, corrects extracted data if needed, and approves invoices after issues are resolved.

## 4. MVP Input

The MVP accepts supplier invoices as PDF, JPG, or PNG files.

For this prototype, invoices may be uploaded manually or stored in Google Drive.

## 5. AI Extraction Fields

The system extracts:

1. Supplier Name
2. Invoice Number
3. Invoice Date
4. PO Number
5. Subtotal
6. VAT / Tax
7. Total Amount

## 6. Storage

Extracted invoice data is saved into Google Sheets as structured rows.

Google Sheets acts as the MVP database.

## 7. Business Rules

The system applies the following rules:

1. All required fields must be present.
2. PO number must be present.
3. Subtotal + VAT / Tax must equal Total Amount.
4. Invoice number must not already exist in Google Sheets.

## 8. Status Logic

The MVP uses two automated statuses:

- Approved
- Needs Review

If all rules pass, the invoice is marked Approved.

If any rule fails, the invoice is marked Needs Review.

## 9. Exception Reasons

The system records the following exception reasons:

- Missing PO number
- Missing invoice number
- Missing required field
- Total mismatch
- Duplicate invoice detected

If multiple issues exist, all exception reasons are recorded.

## 10. Review Summary

The system writes a plain-English review summary explaining why the invoice was approved or why it needs review.

## 11. Human Review

The Finance Admin only reviews invoices marked as Needs Review.

During review, the Finance Admin can inspect extracted fields, compare them with the original invoice, correct values, add review notes, and mark the invoice as approved after resolution.

## 12. Tech Stack

Frontend / UI:
Lovable or simple web app

AI Extraction:
Google Gemini

File Source:
Google Drive

Storage / Database:
Google Sheets

Validation Logic:
App logic / Google Apps Script

Notification:
Gmail through Google Apps Script

Dashboard:
UI reading data from Google Sheets

Documentation:
GitHub