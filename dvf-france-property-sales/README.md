# DVF property sales → Google Sheets (n8n)

Ready-to-import n8n workflow on top of [France Property Sales & Prices (DVF)](https://apify.com/suitable_sunspot/dvf-france-property-sales) (`YKAlMwGiGWNYdpcHh`).

Every Monday it pulls the latest DVF transactions for the selected department(s) and appends them to Google Sheets — one clean row per sale.

## Files

- `workflow.json` — importable n8n workflow (no secrets, credentials are placeholders)
- `README.md` — this page

## What the workflow does

```
Schedule (Monday 07:00)
  → Apify: run dvf-france-property-sales (departments, year, filters)
    → Code: flatten to stable columns
      → Google Sheets: append
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
| `Apify - run DVF` | `departments` (e.g. `["75"]`, `["69"]`, `["2A"]`); optional `communeName`, `postalCode`, `propertyType`, price/surface ranges, `year`; `maxResults` |
| `Google Sheets - append sales` | `Document ID` — your spreadsheet ID; tab name (`Sheet1` by default) |
| `Schedule - weekly Monday 07:00` | cron expression (n8n instance timezone) |

The sheet header row is created automatically on the first run. Columns:

`mutation_id, date, department, commune, postal_code, address, property_types, price_eur, price_per_m2, surface_m2, rooms, land_m2, latitude, longitude`

`oneRowPerTransaction: true` merges multi-lot sales (apartment + cellar + parking) into one row.

## Actor input used

```json
{
  "departments": ["75"],
  "year": "2025",
  "salesOnly": true,
  "oneRowPerTransaction": true,
  "maxResults": 100
}
```

## Good to know

- DVF is published twice a year. A year that is not published yet returns no rows (`2026` becomes available after the publisher's release).
- The workflow has no incremental cursor: re-running appends the same transactions again. Keep `maxResults` modest, or use a fresh sheet per reporting period.
- `salesOnly` drops expropriations, adjudications and exchanges, keeping real sales and VEFA.

## Cost

- $0.003 per returned transaction row (Apify pricing)
- `maxTotalChargeUsd: 0.5` in the Apify node caps a single run

## Verified

- Actor run [`7BhPc2kddYXxRfakT`](https://console.apify.com/actors/YKAlMwGiGWNYdpcHh/runs/7BhPc2kddYXxRfakT) — 100 transactions for department `75`, year `2025`, dataset `sq45odPPyM9P0WEF8`, $0.0016
- `Code - normalize sales` replayed against the real 100-row dataset: 100 rows, all 14 columns populated

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `Unrecognized node type @apify/n8n-nodes-apify.apify` | Install the Apify community node and reload n8n |
| Empty result | Check `year` (must be published), department code format (`75`, `2A`, `971`) and price/surface filters |
| `Could not find column ...` | Clear the sheet so the header row is recreated, or rename columns to match the Code node keys |
