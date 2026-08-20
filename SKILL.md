---
name: xlsx
description: "Creates an Excel workbook (.xlsx) with data, formulas and charts; ODS / Google Sheet variant. Delegated to by `source-to-artifact` when the target is tabular."
---
# XLSX — Excel / spreadsheet creation

Create `.xlsx` Excel spreadsheets (or .ods / Google Sheet variant) from tabular content. Invoked by `source-to-artifact` when the user wants a spreadsheet — typical asks: report builder, data summary, financial model, list with formulas.

## Inputs

From the caller:
- **data source**: workspace CSV / TSV / JSON / pasted table, OR structured data extracted from a document
- **target format**: `xlsx` (Excel native), `ods` (LibreOffice), `gsheet` (Google Sheet)
- **filename**: e.g. `q3-sales.xlsx`
- **sheets**: optional; if multiple sheets, structure per-sheet

## Backend per format

### `xlsx` (native, fast)

Use openpyxl in the workspace:

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
wb = Workbook()
ws = wb.active
ws.title = '<sheet name>'
ws.append([header1, header2, header3])
for h in ws[1]: h.font = Font(bold=True)  # bold header
for row in rows: ws.append(row)
# Formula example:
ws['D2'] = '=SUM(B2:C2)'
# Column widths:
ws.column_dimensions['A'].width = 25
wb.save('<filename>.xlsx')
```

For multiple sheets: `wb.create_sheet('<name>')`.

Write to workspace. Attach to reply.

### `ods` (via cerase-office-converter)

Create `.xlsx` natively, then:

```
call_recipe("cerase-office-converter.convert_xlsx_to_ods", {input_b64: <base64 of .xlsx>})
```

### `gsheet` (via google-workspace MCP)

```
call_recipe("google-workspace.sheets_create", {
  title: "<filename without ext>",
  data: [[<row 1 cells>], [<row 2 cells>], ...],
})
```

Returns `{sheet_id, sheet_url}`. Surface URL.

## Best practices

- **Header row always styled**: bold + background fill + frozen pane (openpyxl `ws.freeze_panes = 'A2'`).
- **Formulas, not hardcoded values**: when summing / averaging / comparing, use `=SUM(...)` etc. The user can extend.
- **Number formatting**: dates as `YYYY-MM-DD`, currency with explicit symbol + 2 decimals (`#,##0.00 "€"`), percentages with 1 decimal (`0.0%`).
- **Multiple sheets**: use when the data has logical groupings (e.g. per-month, per-region). Avoid stuffing everything in one wide sheet.
- **Chart**: don't add unless explicitly asked — charts are fragile across export targets (ODS/Google handle them differently). If asked, use openpyxl `BarChart` / `LineChart` / `PieChart`.

## Don't

- Don't write column widths blindly: estimate from content length (~1.2 char per width unit).
- Don't fabricate data: if a cell is unknown, leave it blank + add a note in a `_notes` column.
- Don't ignore the `cerase-office-converter` for cross-format — handles edge cases (frozen panes, conditional formatting) correctly during conversion.
