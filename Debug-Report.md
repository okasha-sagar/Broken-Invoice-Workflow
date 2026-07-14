# Debug Report – Broken Invoice Workflow

## Overview

I imported the provided workflow into n8n Cloud (v2.28.4), executed it using the webhook, and used the execution logs to identify and fix the issues preventing the workflow from running correctly. After resolving the problems, the workflow successfully validates the invoice, calculates a 20% tax, and returns the processed total.

---

## Bug 1 – Missing Node Connection

**Issue:**
The **Validate Invoice** node was not connected to the **Format Response** node, causing the workflow to stop before formatting the response.

**Fix:**
Connected the nodes in the correct order:

```
Invoice Webhook
    ↓
Validate Invoice
    ↓
Format Response
    ↓
Respond
```

---

## Bug 2 – Incorrect HTTP Method

**Issue:**
The **Validate Invoice** HTTP Request node used the **GET** method while attempting to send a JSON request body to an endpoint intended for POST requests.

**Fix:**
Changed the HTTP method from **GET** to **POST**.

---

## Bug 3 – Empty fields in HTTP Request node

**Issue:**
The JSON request body contained empty value and expression, preventing the HTTP Request node from executing successfully.

**Fix:**
Configured the request body correctly by sending:

* **Field:** `invoice_amount`
* **Value:** `{{$json.body.amount}}`

This allowed the invoice amount received by the webhook to be sent correctly to the validation API.

---

## Bug 4 – Logic Error in the Format Response Node

**Issue:**
The original **Function** node was unavailable in my n8n Cloud version (v2.28.4), so it was replaced with a **Code** node. The original logic also referenced the invoice amount incorrectly.

**Fix:**
Replaced the unavailable Function node with a Code node and updated the JavaScript to:

* Read the invoice amount from the HTTP response.
* Calculate a 20% tax.
* Calculate the final total.
* Return the data in the correct n8n array format.

The workflow now correctly returns:

```json
{
  "invoiceAmount": 150,
  "tax": 30,
  "total": 180
}
```

---

## Bug 5 – Misconfigured Error Workflow

**Issue:**
The imported workflow referenced a non-existent Error Workflow. In addition, the main workflow was initially inactive, preventing the Error Workflow from executing during production tests.

**Fix:**

* Created a dedicated **Invoice Error Workflow** using the **Error Trigger** node.
* Activated the Error Workflow.
* Configured the main workflow to use it as its **Error Workflow**.
* Activated the main workflow and tested using the **production webhook**, confirming that errors correctly triggered the Error Workflow.

---

## Additional Debugging Performed

During testing, I encountered a temporary **503 Service Unavailable** response from the external validation endpoint. Since this was caused by the external service rather than the workflow itself, I replaced it with a **Postman Mock Server** to provide reliable and repeatable API responses for testing.

The mock server was also configured to simulate both successful and failed validation responses, allowing verification of the workflow's error handling.

---

## Final Result

The corrected workflow now:

* Receives invoice data via a webhook.
* Sends the invoice amount to the validation API.
* Calculates a 20% tax.
* Returns the processed total.
* Correctly triggers the configured Error Workflow when a workflow execution fails.

The workflow executes successfully and meets all assignment requirements.
