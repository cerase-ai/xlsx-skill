# xlsx-skill

A Cerase skill that has the assistant produce a spreadsheet: an Excel workbook
(`.xlsx`), an OpenDocument spreadsheet (`.ods`) or a Google Sheet, with data
and formulas. `source-to-artifact` calls it when the target is a table, for
example a report, a data summary, a financial model or a list with totals.
The caller passes the data (a CSV, TSV or JSON file in the workspace, a pasted
table, or data extracted from a document), the target format, the file name
and, optionally, how to split it into sheets.

## What the assistant does

- **`xlsx`:** calls `cerase-office-converter.create_xlsx` with the rows of
  each sheet (numbers as numbers, formulas as strings starting with `=`, dates
  as `YYYY-MM-DD` strings) and, per sheet, the number of header rows, an Excel
  number format per column, column widths and the cell to freeze panes at. The
  converter writes the workbook to `outputs/` in the workspace and returns its
  path, and the assistant attaches it with `[[attach: <path>]]`.
- **`ods`:** builds the `.xlsx` first, then converts it with
  `cerase-office-converter.convert_xlsx_to_ods`.
- **PDF of the table:** builds the `.xlsx` first, then converts it with
  `cerase-office-converter.convert_xlsx_to_pdf`.
- **`gsheet`:** builds the `.xlsx`, uploads it to the person's Drive with
  `google-workspace.uploadFile` and `convertToGoogleFormat: true`, and gives
  the person the link of the new Google Sheet. When the Google Workspace
  connector is not among the assistant's connectors, it says the
  organisation's administrator has to assign it and sends the `.xlsx`
  instead.

Rules: one bold, shaded, frozen header row and no title rows above it;
formulas such as `=SUM(...)` instead of computed values; an Excel number
format per column for money, shares and dates; columns without a set width
sized to their content; several sheets when the data has natural groups; no
charts; an unknown value left blank with a note in a `Note` column, never
invented. The assistant's container has no Python or LibreOffice, so the
assistant never builds the file itself, and it never pastes file content or
base64 in the chat.

## Requirements

- The `cerase-office-converter` connector for every format: `create_xlsx`
  builds the `.xlsx`, `convert_xlsx_to_ods` converts it for `ods`, and
  `convert_xlsx_to_pdf` converts it to PDF.
- A Google Workspace connector exposing `uploadFile` for `gsheet`.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The instructions the assistant loads: `name` and `description` frontmatter, then the method per format and the rules. |
| `cerase.json` | Marketplace manifest: namespace `studio.guidance`, name `xlsx`, display name, description, licence. |
| `i18n.yaml` | Italian display name and description for the Marketplace; not sent to the assistant. |
| `LICENSE` | MIT licence text. |

## Installation

Published in the Cerase Marketplace as `studio.guidance/xlsx`
([marketplace page](https://marketplace.cerase.ai/en/p/studio.guidance/xlsx)).
A Cerase appliance does not attach it by default: an administrator installs it
from the Marketplace.

## License

MIT. See [LICENSE](LICENSE).
