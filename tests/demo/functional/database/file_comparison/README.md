# File-to-File Comparison Demo (CSV / XLS / XLSX + multi-sheet)

Demonstrates `compare_csv_files` and `compare_excel_files` across every file
format, plus exactly how multi-sheet workbooks behave. All test data is
**generated at runtime** under `test_data/file_comparison/` -- nothing binary is
committed, and it runs with zero manual setup.

## Multi-sheet behaviour (important)

By default `compare_excel_files` compares **one sheet**:

1. **Default = first sheet only.** With `sheet_name=0` (the default), only the
   first sheet is compared. A difference on any other sheet is silently NOT
   detected -- a green result only means the FIRST sheet matched.
2. **Validate a whole workbook by looping over its sheets:**
   ```python
   import pandas as pd
   for sheet in pd.ExcelFile(src).sheet_names:
       result = compare_excel_files(src, tgt, sheet_name=sheet, key_columns=[...])
   ```
3. **Or compare every sheet in one call** with `sheet_name=None`. It returns a
   `MultiSheetComparisonResult` with per-sheet results in `sheet_results`.
   `key_columns` applies to every sheet, so leave it out (rows match by
   position) or loop as above when sheets have different keys.

## `.xls` (legacy) note

pandas 3.x can **read** `.xls` (via `xlrd`) but can no longer **write** it, so
the demo writes its `.xls` fixtures with `xlwt` directly. The `.xls` tests
`skip` automatically if `xlwt` isn't installed.

## Prerequisites

```bash
pip install pandas openpyxl     # CSV + XLSX
pip install xlwt xlrd           # only for the legacy .xls tests
```

## Run

```bash
pytest tests/demo/functional/database/file_comparison/test_file_comparison.py -v -s
```
