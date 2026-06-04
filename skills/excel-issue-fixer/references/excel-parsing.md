# Excel Parsing

## parse script (run first, always)

```python
import pandas as pd
import json, sys

file = sys.argv[1]  # path to .xlsx
df = pd.read_excel(file, sheet_name=0)  # first sheet default

# normalize column names
df.columns = [c.strip().lower().replace(' ','_') for c in df.columns]

# map common column aliases
COL_MAP = {
    'id':          ['id','no','#','issue_id','row'],
    'title':       ['title','issue','name','summary','subject'],
    'description': ['description','detail','desc','note','remark','body'],
    'priority':    ['priority','severity','level','p'],
    'type':        ['type','category','area','domain','tag'],
    'assignee':    ['assignee','owner','assigned_to','assigned'],
    'status':      ['status','state'],
}

result = {}
for key, aliases in COL_MAP.items():
    for a in aliases:
        if a in df.columns:
            result[key] = a
            break

issues = []
for _, row in df.iterrows():
    issues.append({
        'id':          str(row.get(result.get('id',''), len(issues)+1)),
        'title':       str(row.get(result.get('title',''), '')),
        'description': str(row.get(result.get('description',''), '')),
        'priority':    str(row.get(result.get('priority',''), 'medium')),
        'raw_type':    str(row.get(result.get('type',''), '')),
    })

# filter empty rows
issues = [i for i in issues if i['title'] and i['title'] != 'nan']

print(json.dumps(issues, ensure_ascii=False, indent=2))
```

Save as `parse_excel.py`, run: `python parse_excel.py issues.xlsx`

## multi-sheet

```python
# list all sheets
xl = pd.ExcelFile(file)
print(xl.sheet_names)

# read specific sheet
df = pd.read_excel(file, sheet_name='Bugs')
```

## common formats handled

| Format | Handling |
|---|---|
| Column headers row 1 | default |
| Headers row 2+ | `header=N` param |
| Merged cells | `openpyxl` engine, unmerge first |
| Multiple sheets | iterate `sheet_names` |
| Thai/Unicode text | pandas handles, `ensure_ascii=False` in json |

## output contract

```json
[
  {
    "id": "1",
    "title": "Login fails when password has special chars",
    "description": "User reports 500 error when password contains @#$",
    "priority": "high",
    "raw_type": "Bug"
  }
]
```
