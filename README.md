# Apify → n8n workflow templates

Importable [n8n](https://n8n.io) workflows for Apify Actors. Each workflow runs an Actor on a schedule, normalizes its dataset and appends clean rows to Google Sheets. The ATS template can also post one digest per run to Slack or Telegram.

## Templates

| Folder | Actor | What it does |
| --- | --- | --- |
| [`keyless-ats-jobs-api/`](keyless-ats-jobs-api/) | [ATS Jobs API & Hiring Signals](https://apify.com/suitable_sunspot/keyless-ats-jobs-api) | Daily at 08:00, returns only job postings that appeared since the previous run (`onlyNewJobs` + `stateKey`), appends them to Google Sheets, optional Slack/Telegram digest |
| [`dvf-france-property-sales/`](dvf-france-property-sales/) | [France Property Sales & Prices (DVF)](https://apify.com/suitable_sunspot/dvf-france-property-sales) | Weekly on Monday, pulls DVF transactions for the selected department(s) and appends one row per sale to Google Sheets |

Each folder contains:

- `workflow.json` — ready to import into n8n
- `README.md` — setup steps, Actor input, sheet columns, cost and troubleshooting

## Requirements

- n8n 1.60 or newer (self-hosted with Node.js 22+, or n8n Cloud)
- Apify community node: `@apify/n8n-nodes-apify` (Settings → Community nodes → Install)
- Apify API token
- Google Sheets account connected via OAuth2 in n8n
- Optional: Slack and/or Telegram credentials for digest notifications

## Import a workflow

1. Download `workflow.json` from the template folder.
2. In n8n open **Workflows → ⋯ → Import from File** and select the file.
3. If n8n reports an unknown node type, install the Apify community node and reload the page.
4. Open each node marked with a credential warning and select (or create) your credential.
5. Set your spreadsheet ID, target companies or departments, and the schedule.

The first run creates the header row in the sheet. Column names come from the workflow's Code node and are documented in each template README.

## Credentials and secrets

This repository contains **no tokens, emails or account data**. Every credential reference is a placeholder (`REPLACE_APIFY_CREDENTIAL_ID`, `REPLACE_GOOGLE_SHEETS_CREDENTIAL_ID`, `REPLACE_SLACK_CREDENTIAL_ID`, `REPLACE_TELEGRAM_CREDENTIAL_ID`, `YOUR_GOOGLE_SHEET_ID`). Your Apify token and Google/Slack/Telegram credentials are stored only inside n8n's credential store.

## Cost

Runs are billed by Apify according to each Actor's pricing (see the Actor pages). Both workflows set `maxTotalChargeUsd: 0.5` on the Apify node, so a single run cannot exceed that cap.

- ATS Jobs: $0.002 per delivered job listing
- DVF: $0.003 per returned transaction row

## License

MIT — see [LICENSE](LICENSE).
