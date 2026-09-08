# Petition Data Exporter

A single-file web app that fetches a UK Parliament petition's data and exports it as a two-sheet Excel `.xls` file.

## What it does

1. You enter a petition ID (e.g. `700896`) or paste a full petition URL into the input field.
2. The app fetches the petition's data from the official UK Parliament petitions API:
   `https://petition.parliament.uk/petitions/{id}.json`
3. It shows a preview of the petition — title, status, signature count, a snippet of the main text, and a table of signatures by constituency.
4. Clicking **Download .xls** generates and downloads a legacy Excel `.xls` file (not `.xlsx`) containing two sheets built from that data.

## File structure of the generated .xls

**Sheet 1 — `Petition data - {id}`**

A two-column, four-row summary (no header row):

| | |
|---|---|
| Title | the petition's `action` text |
| Main text | the petition's `background` text (HTML stripped to plain text) |
| Status | the petition's `state` (e.g. `open`, `closed`) |
| Signatures | the petition's `signature_count` |

**Sheet 2 — `Signature data - {id}`**

A table with one row per constituency, built from the `signatures_by_constituency` field:

| Constituency | MP | Signatures |
|---|---|---|
| `name` | `mp` | `signature_count` |

## How it's built

- A single self-contained `petition-xls-exporter.html` file — no build step, no server, no installation.
- Petition data is fetched client-side, directly from the Parliament petitions API.
- The `.xls` file is generated in the browser using [SheetJS](https://sheetjs.com/) (`xlsx` library), loaded from a CDN, writing in the `biff8` format to produce a genuine legacy `.xls` binary rather than a renamed `.xlsx`.
- Styling is plain CSS with no framework dependency.

## Usage

1. Open `petition-xls-exporter.html` in any modern web browser (double-click it, or host it anywhere).
2. Enter a petition ID or URL and click **Fetch petition**.
3. Review the preview, then click **Download .xls** to save the file.

## Limitations

- Requires an internet connection to reach `petition.parliament.uk`.
- If the Parliament site blocks the browser request (CORS) or the petition ID doesn't exist, the app shows an on-screen error explaining what went wrong rather than downloading a file.
- Only petitions that exist on `petition.parliament.uk` (UK Government and Parliament petitions site) are supported — this does not work with petitions from other platforms.
