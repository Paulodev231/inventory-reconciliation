Inventory Reconciliation & Replenishment Signal - Created by Paulo (x.com/TheAiGeng) 16/1/2025

One-line: Daily automated comparison of ERP, WMS and e-commerce inventory to surface exceptions and generate replenishment recommendations.

This repo contains an import-ready n8n workflow, sample inventory snapshots, and step-by-step instructions so a non-technical teammate can upload, import, and test the workflow safely.

What’s included in this repo
workflows/
  01-inventory-recon.json        ← n8n workflow (import file)
samples/
  sample_inventory_erp.json
  sample_inventory_wms.json
  sample_inventory_ecom.json
README.md                    ← this file
.gitignore

High-level overview (what this does)

Every day the workflow:

Pulls inventory snapshots from ERP, WMS and e-commerce (or uses sample files for demo).

Normalizes and compares counts per SKU.

Flags SKUs where systems disagree beyond a threshold.

Creates a replenishment recommendation row (Airtable or Google Sheet).

Notifies the buyer via Slack with a suggested quantity + confidence score.

Writes a CSV reconciliation report to SFTP/SharePoint for audit.

Before you start — what you need

Ask your IT/admin team for these items, or use the sample files for a demo:

Access to an n8n instance (cloud or local). If local, n8n can run in Docker.

API credentials / endpoints for:

ERP (or SFTP folder with snapshot files)

WMS

E-commerce (Shopify or other)

Airtable or Google Sheets (for recommendations)

Slack (bot token to post messages)

SFTP or SharePoint (for report uploads)

Optional: OpenAI key if you want LLM-assisted velocity notes (the workflow has this step optional).

Important safety note

Do not upload API keys or any file named .env to GitHub.
Keep credentials private. This repo only contains safe example files.

Quick start — non-technical step-by-step

Follow these exact steps in order. No coding needed.

1) Confirm the folder is ready

Make sure you have the folder with the files listed in What’s included. Do not include any .env or credential files.

2) Import the workflow into n8n

Open your n8n instance in a browser and sign in.

Top-right, click Import → Import from File.

Choose workflows/01-inventory-recon.json from this repo and import it.

The workflow appears in your n8n workspace. Do not activate it yet.

3) Add credentials in n8n (clicks)

In n8n, click Credentials in the left menu.

Add credentials for each system that you will connect:

HTTP Request / API credentials for ERP, WMS, E-commerce (or use separate credentials per system).

Airtable (or Google Sheets), if using Airtable create an API key and enter it here.

Slack bot token credential (so n8n can post messages).

SFTP credential (username/password or SSH key) for report upload.

OpenAI credential (optional).

If you prefer environment variables, you can set those instead in Settings → Environment (advanced). The workflow uses environment variables for sheet IDs and threshold values — see below.

Tip: For a quick demo you can skip the API credentials and use the sample JSON files instead (instructions below).

4) Set environment variables (optional but recommended)

In n8n Settings → Environment or via your n8n .env file, set the following (replace placeholders):

ERP_API_URL=https://erp.example.com/api
ERP_API_TOKEN=REPLACE_WITH_ERP_TOKEN

WMS_API_URL=https://wms.example.com/api
WMS_API_TOKEN=REPLACE_WITH_WMS_TOKEN

SHOPIFY_API_URL=https://your-shop.myshopify.com/admin
SHOPIFY_TOKEN=REPLACE_WITH_SHOPIFY_TOKEN

AIRTABLE_APP_ID=appXXXXXXXXXXXX
SLACK_CHANNEL=#inventory-alerts

RECON_DIFF_THRESHOLD=5
SFTP_HOST=sftp.example.com
SFTP_USER=youruser
SFTP_PATH=/reports/inventory


If you do not set environment variables, you can edit the workflow nodes directly and paste these values into the node bodies.

5) Test using the sample files (no external systems required)

This is the safest path for a first run.

Open the imported workflow in n8n.

Temporarily edit the Cron (daily) node:

Change it to Manual Trigger (or set it to run in the next minute) so you can run it immediately for testing.

Replace the three HTTP Request nodes (ERP, WMS, Ecom) with read-file nodes that read the sample JSON files:

Add a Read Binary File node or use an HTTP Request node pointed to a local file URL if supported by your n8n instance.

Point them to the samples/sample_inventory_*.json files in this repo.

Alternatively, open each HTTP Request node and temporarily paste the sample JSON into the Response area if your n8n allows local mocking.

Save and Execute Workflow (Run once / Manual execute).

Watch the execution results (n8n shows an "Executions" panel). Confirm the run completes.

6) Verify results (what to look for)

Airtable or Google Sheet: one or more new rows named “Replenishment” or similar (if you used Airtable).

Slack: a message should appear in the #inventory-alerts channel with SKU, ERP/WMS/Ecom counts and a suggested quantity.

SFTP / local file: a CSV report file inventory-recon-YYYY-MM-DD.csv should exist in the configured location.

n8n Execution log: no failed nodes — expand the nodes in the execution log to see inputs/outputs.

7) Switch to scheduled run

When the test passes:

Edit the Cron node and set it back to daily at your desired time (default used in the workflow is 03:00).

Activate the workflow (turn the toggle to ON).

How the reconciliation logic works (brief, plain)

The workflow collects quantities for each SKU from ERP, WMS and e-commerce.

It computes a simple diff and a reconciliation score for each SKU.

If the diff is greater than RECON_DIFF_THRESHOLD, it creates a replenishment row and notifies the buyer.

SuggestedQty uses a basic formula that can be tuned later to align with buying policies.

Acceptance criteria (how to know it’s working)

A manual test run with the sample files produces:

At least one replenishment row in Airtable / Google Sheet.

A Slack alert with SKU details and suggested quantity.

A CSV report uploaded to the chosen destination.

Buyer can view the Airtable row and mark it as acknowledged (this can be added as a follow-up step).

Troubleshooting — common issues & fixes

No data returned from API nodes: check the API URL and token. Test the API in Postman if unsure.

Airtable row failing: confirm API key and table names; check field names match exactly.

Slack message not posted: make sure the Slack bot token is valid and the bot is invited to the channel.

SFTP upload fails: verify hostname, port, username, and that the path exists and the user has write permission.

Workflow errors in n8n: open Executions → click the failed run → click the node with error → read the error and check the inputs used in that run.

Sample input format (use this to build test files)

Each source should return an array of objects. Example:

[
  {"sku":"SKU-001","qty":120},
  {"sku":"SKU-002","qty":45},
  {"sku":"SKU-003","qty":0}
]


The sample files in samples/ already follow this format.

Handover notes (what to give to the buyer or admin)

Repo URL (GitHub) and instructions to import the workflow.

API tokens (deliver these securely; do not commit them to the repo).

Slack channel name and Airtable base/table names.

Threshold value for exception alerts (RECON_DIFF_THRESHOLD).

Quick checklist to hand to a non-technical uploader

 I did not upload any .env or secret files.

 I imported workflows/01-inventory-recon.json into n8n.

 I added credentials for ERP / WMS / Ecom / Airtable / Slack / SFTP.

 I tested with sample files and confirmed Slack, Airtable and CSV outputs.

 I turned on the workflow schedule.

Where to go from here (recommended next steps)

Tune RECON_DIFF_THRESHOLD after 1–2 weeks of real data.

Improve SuggestedQty logic using historical sales velocity.

Add an acknowledgement step in Airtable (buyer can click "Acknowledge" and the workflow can then create a real PO).

Add retries and alerting for failed runs (n8n supports error workflows).

License

MIT — feel free to reuse the workflow and adapt it to your system