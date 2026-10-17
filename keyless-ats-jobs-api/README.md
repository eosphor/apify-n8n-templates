# ATS jobs monitor → Google Sheets (n8n)

Ready-to-import n8n workflow on top of [ATS Jobs API & Hiring Signals](https://apify.com/suitable_sunspot/keyless-ats-jobs-api) (`N6UZgOS3pRkYpPZwQ`).

Every day it runs the Actor for your target companies, keeps **only job postings that appeared since the previous run**, appends them to Google Sheets, and optionally posts one digest to Slack and/or Telegram.

## Files

- `workflow.json` — importable n8n workflow (no secrets, credentials are placeholders)
- `README.md` — this page

## What the workflow does

```
Schedule (08:00 daily)
  → Apify: run keyless-ats-jobs-api (onlyNewJobs = true, stateKey)
    → Code: normalize + de-duplicate rows
      → Google Sheets: append
        → Code: one digest message
          → Slack (optional, disabled)
          → Telegram (optional, disabled)
```

## Requirements

- n8n 1.60+ (self-hosted with Node.js 22+, or n8n Cloud)
- Community node **Apify**: Settings → Community nodes → Install → `@apify/n8n-nodes-apify`
- Apify account with an API key ([console → Integrations](https://console.apify.com/settings/integrations))
- Google Sheets account connected via OAuth2 in n8n

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and pick `workflow.json`.
2. Install the Apify community node if n8n asks for it, then reload the page.
3. Open each node with a red credential badge and select (or create) the credential.

## Configure

| Node | What to change |
| --- | --- |
| `Apify - run ATS jobs` | `companyWebsites` — your target companies (website or careers URL, e.g. `acme.com`); `stateKey` — one name per watch-list; `maxJobsPerCompany`, `maxItems` |
| `Google Sheets - append jobs` | `Document ID` — your spreadsheet ID; tab name (`Sheet1` by default) |
| `Schedule - daily 08:00` | cron expression (n8n instance timezone) |
| `Slack - digest (optional)` | enable the node, pick credential + channel |
| `Telegram - digest (optional)` | enable the node, pick credential + chat ID |

The first run returns the current baseline (up to `maxItems`). Later runs return only new postings, because the Actor stores its memory in a named key-value store in **your** Apify account under `stateKey`. Use a different `stateKey` per schedule.

The sheet header row is created automatically on the first run. Columns:

`job_id, company, title, location, department, employment_type, remote, posted_at, salary, url, ats, first_seen`

## Actor input used

```json
{
  "companyWebsites": ["stripe.com", "notion.so"],
  "onlyNewJobs": true,
  "stateKey": "n8n-ats-watchlist",
  "maxJobsPerCompany": 50,
  "maxItems": 100,
  "includeDescriptions": false
}
```

Companies that cannot be matched to a supported ATS are listed in the Actor's coverage output as `not_detected` and cost nothing. Hitting `maxJobsPerCompany` is reported as `capped`, never as a silent truncation.

## Cost

- $0.002 per delivered job listing (Apify pricing)
- `maxTotalChargeUsd: 0.5` in the Apify node caps a single run
- Typical daily run for 5–10 companies: a few cents

## Verified

- Actor run [`WdIr8xaLdhZoi8IXv`](https://console.apify.com/actors/N6UZgOS3pRkYpPZwQ/runs/WdIr8xaLdhZoi8IXv) — 100 jobs from `stripe.com` + `notion.so`, dataset `ap3HCV1gOYtLy7CeX`, $0.0019
- Follow-up run on the same `stateKey` returned 0 rows (`already_seen: 100`), confirming only-new behaviour
- `Code` nodes replayed against the real 100-row dataset: 100 normalized rows, 1 digest message

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `Unrecognized node type @apify/n8n-nodes-apify.apify` | Install the Apify community node and reload n8n |
| `Could not find column ...` | The sheet has a header row that does not match the Code node keys — clear the sheet so headers are recreated, or rename columns |
| Empty result every day | `stateKey` already has a baseline; the Actor only returns new postings. Change `stateKey` to re-baseline |
| `Run ... did not finish with status SUCCEEDED` | Open the run link from the node error; usually an invalid company input or a timeout |
