# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single self-contained static page (`index.html`) that fetches a UK Parliament petition from `https://petition.parliament.uk/petitions/{id}.json` and exports it as a legacy Excel `.xls` (BIFF8) file with two sheets. Everything — markup, CSS, and JS — lives in `index.html`.

## Development

There is no build step, package manager, linter, or test suite. To run it, open `index.html` in a browser, or serve the directory (e.g. `python3 -m http.server`) and visit it. Verifying changes means fetching a real petition (e.g. ID `700896`) and opening the downloaded file in Excel/LibreOffice.

External dependencies are loaded from CDNs only: SheetJS (`xlsx@0.18.5` from jsDelivr, exposing the global `XLSX`) and the Inter font from Google Fonts. Keep it dependency-free beyond these.

## How the script fits together

The inline script is a single IIFE with module-level state `currentData` (the API's `data.attributes` object) and `currentId`:

- `extractId` accepts either a bare numeric ID or any URL containing `petitions/<digits>`.
- `fetchPetition` resets state, calls the API client-side, handles 404 / non-OK / malformed-shape / network (often CORS) errors with on-screen messages via `showError`, then stores the attributes and calls `renderResult`.
- `renderResult` shows a preview, capped at the first 50 constituencies (the download always includes all of them).
- `downloadXls` builds both sheets from `currentData` with `XLSX.utils.aoa_to_sheet` and writes with `{ bookType: 'biff8' }` — this must stay `biff8` so the output is a genuine `.xls`, not a renamed `.xlsx`.
- `htmlToPlainText` converts the petition's HTML `background` field to plain text; it's used by both the preview and the export.

## Export format (contract described in README)

- Sheet 1 `Petition data - {id}`: four label/value rows, no header — Title (`action`), Main text (`background`, HTML stripped), Status (`state`), Signatures (`signature_count`).
- Sheet 2 `Signature data - {id}`: header row `Constituency | MP | Signatures`, then one row per entry in `signatures_by_constituency` (`name`, `mp`, `signature_count`).
- Sheet names are truncated to 31 characters (Excel/BIFF limit). Output filename is `petition-{id}.xls`.

If you change the export shape, update the README's "File structure of the generated .xls" section to match.
