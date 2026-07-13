# Tool: openpyxl (Excel parsing)

RobNTex reads `.xlsx` test-case files via **openpyxl** (pure-Python, no Office required).
Read this before parsing any Excel file.

## Setup & preflight

- Preflight: `python3 -c "import openpyxl; print(openpyxl.__version__)"`.
  - If missing: `pip install openpyxl` (or `pip install --break-system-packages openpyxl`
    on system-managed Python).
- Do NOT parse `.xls` (legacy binary). If the user provides one, ask them to save-as `.xlsx`.
- Do NOT try to read the file as CSV — Excel formulas and merged cells won't survive.

## Reading pattern

The canonical parse loop for one workbook (all sheets = all suites):

```python
from openpyxl import load_workbook

wb = load_workbook("<path.xlsx>", data_only=True, read_only=True)
# data_only=True → get computed formula values, not the formula strings.
# read_only=True → memory-safe for large workbooks.

for sheet in wb.worksheets:
    rows = list(sheet.iter_rows(values_only=True))
    if not rows:
        continue
    header = [str(c).strip().lower() if c else "" for c in rows[0]]
    # Build a column-name → index map from the header row.
    col = {name: header.index(name) for name in header if name}
    for row in rows[1:]:
        if row is None or all(c is None for c in row):
            continue  # skip fully empty rows
        # Extract fields by col name — never by position.
        tc_id = row[col["tc-id"]]
        ...
```

**Rules:**
- Always match column names **case-insensitively** and by name — never by column index. Users
  reorder columns.
- Treat empty cells as `None`, not `""`. `str(None).strip()` is a bug in disguise.
- Skip rows that are entirely empty. Do NOT use them as TC separators (see `tc-format.md`).

## Grouping rows into TCs

Rows sharing the same `TC-ID` belong to one TC. The parser must:

1. Group by `TC-ID` in dict-of-lists.
2. For each group, take **Title, Module, Priority, Preconditions, Test Data, Stateful, Jira**
   from the **first non-empty value** across the group's rows (users often fill these only
   on the first row).
3. Sort the group's rows by `Step #` ascending.
4. Validate: `Step #` is contiguous starting at 1; every step has `Action` and `Expected`.

## `Test Data` field parsing

The `Test Data` column may be:
- A JSON object string: `{"qatarId": "TEST-QID-001", "cptCode": "99213"}` → `json.loads(...)`.
- A key=value list: `qatarId=TEST-QID-001; cptCode=99213` → split on `;` then `=`, strip whitespace.

Try JSON first; fall back to the key=value form. If both fail, emit a warning and store the
raw string under `_raw`.

## Gotchas

- Merged cells: openpyxl returns the value only in the top-left cell of a merge; other cells
  read as `None`. Don't merge cells in `Action`/`Expected` columns — treat merged headers as
  a linter warning to the user.
- Formulas: `data_only=True` requires Excel to have saved computed values. If the user opened
  the file only in LibreOffice, formulas may show as `None`. Warn the user.
- Encoding: openpyxl returns Python `str` for text (Unicode-safe by default) — Arabic in cells
  works.
- Large files: prefer `read_only=True` for anything > 5 MB.
