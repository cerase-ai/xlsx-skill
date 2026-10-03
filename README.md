# xlsx-skill

A Cerase skill that has the assistant produce a spreadsheet: an Excel workbook
(`.xlsx`), an OpenDocument spreadsheet (`.ods`) or a Google Sheet, with data,
formulas and, when asked, charts. `source-to-artifact` calls it when the target
is tabular, for example a report, a data summary, a financial model or a list
with formulas. The caller passes the data (a CSV, TSV or JSON file in the
workspace, a pasted table, or data extracted from a document), the target
format, the file name and, optionally, how to split it into sheets.

## What the assistant does

- **`xlsx`:** writes a script with `openpyxl` (header row, data rows, formulas,
  column widths, one sheet per logical group), saves the file in the workspace
  and attaches it to the reply.
- **`ods`:** builds the `.xlsx` first, then converts it with
  `cerase-office-converter.convert_xlsx_to_ods`.
- **`gsheet`:** calls `google-workspace.sheets_create` with the title and the
  rows, and returns the link.

Rules: a bold, filled, frozen header row; formulas such as `=SUM(...)` instead
of computed values; dates as `YYYY-MM-DD`, currency with its symbol and two
decimals, percentages with one decimal; column widths estimated from the
content; charts (`BarChart`, `LineChart`, `PieChart`) only when asked for; an
unknown value left blank with a note in a `_notes` column, never invented.

## Requirements

- Python with `openpyxl` wherever the assistant runs code, for `xlsx` and as
  the first step of `ods`.
- The `cerase-office-converter` connector for `ods`.
- A Google Workspace connector exposing `sheets_create` for `gsheet`.

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
