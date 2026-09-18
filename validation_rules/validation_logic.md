# Validation Logic



After invoice fields are extracted and stored, the system applies deterministic validation rules.



## Rule 1: Required Fields



Required fields:



- Supplier Name

- Invoice Number

- Invoice Date

- PO Number

- Subtotal

- VAT / Tax

- Total Amount



If any required field is missing, mark:



Status: Needs Review  

Exception Reason: Missing required field



If PO Number is missing, mark:



Status: Needs Review  

Exception Reason: Missing PO number



If Invoice Number is missing, mark:



Status: Needs Review  

Exception Reason: Missing invoice number



## Rule 2: Total Calculation



Check:



Subtotal + VAT / Tax = Total Amount



If not equal:



Status: Needs Review  

Exception Reason: Total mismatch



## Rule 3: Duplicate Invoice



Check Google Sheets for existing Invoice Number.



If invoice number already exists:



Status: Needs Review  

Exception Reason: Duplicate invoice detected



## Status Decision



If all rules pass:



Status: Approved  

Exception Reason: None  

Final Status: Approved



If any rule fails:



Status: Needs Review  

Exception Reason: all failed rule reasons  

Final Status: Pending Review



## Review Summary



Generate a plain-English summary explaining the status decision.

