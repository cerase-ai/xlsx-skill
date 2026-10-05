---
name: xlsx
description: "Creates an Excel workbook (.xlsx) with data and formulas, or an OpenDocument (.ods) or Google Sheet version of it. Delegated to by `source-to-artifact` when the target is a table: a report, a data summary, a financial model, a list with totals."
---
# XLSX — Excel workbooks

You hand the office converter the rows, and it builds the workbook. Your container runs no Python, so you never build the file yourself: the converter does.

## Inputs

From the caller, or from the person:
- **data**: a CSV, TSV or JSON file in the workspace, a table pasted in the chat, or figures taken from a document
- **format**: `xlsx` (Excel), `ods` (LibreOffice) or `gsheet` (Google Sheet)
- **file name**: such as `q3-sales.xlsx`
- **sheets**: one, or one per logical group (a month, a region)

## Build the workbook

```
call_recipe("cerase-office-converter.create_xlsx", {
  "output_filename": "q3-sales.xlsx",
  "sheets": [
    {
      "name": "Sales",
      "rows": [
        ["Region", "Q1", "Q2", "Total"],
        ["North", 1200.5, 1350, "=SUM(B2:C2)"],
        ["South", 980, 1010, "=SUM(B3:C3)"],
        ["Total", "=SUM(B2:B3)", "=SUM(C2:C3)", "=SUM(D2:D3)"]
      ],
      "number_formats": {"B": "#,##0.00 \"€\"", "C": "#,##0.00 \"€\"", "D": "#,##0.00 \"€\""},
      "column_widths": {"A": 18}
    }
  ]
})
```

It answers `{path, filename, size_bytes}`: the workbook is in your workspace at `path`, which is `outputs/<file name>`.

What each sheet takes:
- `name`: at most 31 characters, none of `[ ] : * ? / \`, different from every other sheet's.
- `rows`: top to bottom. Numbers go in as numbers, never as strings, so a formula can add them up. A string starting with `=` is a formula. A `YYYY-MM-DD` string is stored as a date. `null` leaves a cell empty.
- `header_rows`: how many top rows are the header, bold, shaded and frozen at the top. Default 1; 0 for none.
- `number_formats`: an Excel format per column letter, applied below the header: `#,##0.00 "€"` for money, `0.0%` for a share, `yyyy-mm-dd` for a date.
- `column_widths`: width per column letter. A column not named is sized to its content.
- `freeze`: the cell to freeze panes at, when it is not the first cell under the header.

## Other formats

- **ods**: `call_recipe("cerase-office-converter.convert_xlsx_to_ods", {"path": "outputs/<name>.xlsx", "output_filename": "<name>.ods"})`
- **PDF** of the table: `call_recipe("cerase-office-converter.convert_xlsx_to_pdf", {"path": "outputs/<name>.xlsx", "output_filename": "<name>.pdf"})`
- **gsheet** (Google Sheet): build the .xlsx, then upload it converted, with the upload call below, and only when the person asked for a Google file: the upload puts the content in their Drive. The answer carries the new file's `Link:`; give the person that link. If the Google Workspace connector is not among your connectors, say in their language that a Google Sheet needs that connector, which the organisation's admin assigns, and send the .xlsx instead.

The upload that makes the Google Sheet:

```
call_recipe("google-workspace.uploadFile", {"localPath": "outputs/<name>.xlsx", "name": "<title>", "mimeType": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", "convertToGoogleFormat": true})
```

These calls are the complete set. Do not invent others.

## Deliver

Attach the file: `[[attach: outputs/<name>.xlsx]]`. Never paste its content or any base64 in the chat.

## Rules

- **Formulas, not computed values**: a total, an average or a difference is a formula, so the person can change a figure and see the result follow.
- **One header row**, then data. No title rows above the header: the sheet name is the title.
- **Several sheets** when the data has natural groups; never one sheet a hundred columns wide.
- **No chart**: the workbook carries the numbers.

## Don't

- Don't write Python, or call `libreoffice` from bash: neither is in your container.
- Don't invent data: a value you do not know stays empty, with a `Note` column saying what is missing.
