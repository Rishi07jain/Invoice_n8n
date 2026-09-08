# Enterprise Invoice Automation Pipeline

An automated pipeline that turns a folder of incoming invoices into a live, self-updating accounts payable/receivable ledger — no manual data entry required.

## The problem

I noticed this problem firsthand in my father's business — every invoice that came in had to be manually opened, read, and typed into a tracker: vendor name, amount, date, all by hand. It worked, but it didn't scale, and manual entry meant it was only a matter of time before a number got mistyped or an invoice slipped through untracked.

## What it does

That's the gap this pipeline closes. Drop an invoice (PDF or image) into a watched Google Drive folder, and the pipeline takes it from there — it reads the document with the Gemini API, pulls out the fields that actually matter (creditor, debtor, amount, date), and logs them straight into a Google Sheet acting as the accounts payable/receivable ledger. The finance team gets an email the moment it's processed, so there's no need to go check manually or wait for someone to update a tracker at the end of the day.

Because the extraction is AI-based rather than traditional OCR, it can read handwritten invoices, not just typed ones — a common failure point for rule-based OCR systems.

## Pipeline flow

```
Google Drive (watched folder)
        │  new file detected
        ▼
Download invoice (Drive API)
        │
        ▼
Encode to base64
        │
        ▼
Gemini API — multimodal extraction
        │  (vendor, invoice #, dates, total, line items)
        ▼
Parse & structure response
        │
        ├──────────────► Email notification (Gmail API)
        │
        └──────────────► Append row to ledger (Google Sheets API)
```

## Tech stack

- **[n8n](https://n8n.io)** — self-hosted (Docker) workflow orchestration engine running the entire pipeline
- **Google Drive API** — folder polling trigger + file download
- **Gemini API** — multimodal document understanding for field extraction, including handwritten invoices
- **Gmail API** — automated finance-team notifications
- **Google Sheets API** — ledger storage, append-only

Runs entirely on free-tier services — no paid infrastructure required.

## Why this approach

- **AI extraction over OCR**: traditional OCR struggles with handwriting and inconsistent invoice layouts; a multimodal LLM reads the document semantically instead of pattern-matching characters.
- **Event-driven, not scheduled batch**: processing happens the moment a file lands, so the ledger is always current rather than updated in end-of-day batches.
- **No infrastructure to maintain**: the whole pipeline is a single n8n workflow definition, portable and version-controllable as JSON.
