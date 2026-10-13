# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single self-contained static page (`index.html`) that fetches a UK Parliament petition from `https://petition.parliament.uk/petitions/{id}.json` and exports it as an Excel `.xlsx` file with two sheets. (The repo name says "xls"; the output was switched from legacy `.xls` to `.xlsx`.) Everything — markup, CSS, and JS — lives in `index.html`.

## Development

There is no build step, package manager, linter, or test suite. To run it, open `index.html` in a browser, or serve the directory (e.g. `python3 -m http.server`) and visit it. Verifying changes means fetching a real petition (e.g. ID `700896`) and opening the downloaded file in Excel/LibreOffice.

External dependencies are loaded from CDNs only: ExcelJS (`exceljs@4.4.0` from jsDelivr, exposing the global `ExcelJS`) and the Inter font from Google Fonts. Keep it dependency-free beyond these.

## How the script fits together

The inline script is a single IIFE with module-level state `currentData` (the API's `data.attributes` object) and `currentId`:

- `extractId` accepts either a bare numeric ID or any URL containing `petitions/<digits>`.
- `fetchPetition` resets state, calls the API client-side, handles 404 / non-OK / malformed-shape / network (often CORS) errors with on-screen messages via `showError`, then stores the attributes and calls `renderResult`.
- `renderResult` shows a preview, capped at the first 50 constituencies (the download always includes all of them).
- `downloadXlsx` builds both sheets from `currentData` via `addSheet`, writes with `wb.xlsx.writeBuffer()`, and triggers the download through a Blob URL.
- `addSheet` applies `EXPORT_FONT` (Aptos Narrow 11pt, black) and a per-sheet alignment (sheet 1 left/top with wrapped text so rows grow to fit, sheet 2 centred) to each column's default style and to every written cell. The workbook's built-in default font remains Calibri, so cells outside the exported columns use it.
- Sheet 2 has an `autoFilter` spanning the header and all data rows, giving filter/sort dropdowns in Excel.
- `styleAsLabels` gives sheet 1 column A and the sheet 2 header row white text on `#196B24` (Excel's "Dark Green, Accent 3" in the Office 2023 theme). It's an explicit RGB, not a theme colour, because ExcelJS embeds an older Office theme where accent 3 is olive `#9BBB59`.
- `htmlToPlainText` converts the petition's HTML `background` field to plain text; it's used by both the preview and the export.

## Export format (contract described in README)

- Sheet 1 `Petition data - {id}`: four label/value rows, no header — Title (`action`), Main text (`background`, HTML stripped), Status (`state`), Signatures (`signature_count`).
- Sheet 2 `Signature data - {id}`: header row `Constituency | MP | Signatures`, then one row per entry in `signatures_by_constituency` (`name`, `mp`, `signature_count`).
- Sheet names are truncated to 31 characters (Excel limit). Output filename is `petition-{id}.xlsx`.

If you change the export shape, update the README's "File structure of the generated .xlsx" section to match.
