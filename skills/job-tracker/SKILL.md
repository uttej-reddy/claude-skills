---
name: job-tracker
description: Manage job application tracker (Google Drive docx). Use when checking application status, updating tracker, adding companies, or viewing the pipeline.
---

# Job Tracker

Manage the job application tracker spreadsheet.

## Shared Data

Profile and file paths: `/Users/uttejreddy/.claude/skills/job-data/profile.json`

## Tracker Location

```
${JOBS_DIR}/${TRACKER_FILE}
```

## Tracker Structure

**One row per role.** Columns:
| # | Company | Location | Status | Referrer | Roles |
|---|---------|----------|--------|----------|-------|

- **#** — Company number (same across all roles for the same company)
- **Company** — Company name (vertically merged across all rows for the same company)
- **Location** — Location (vertically merged across all rows for the same company)
- **Status** — Per-role status (see values below)
- **Referrer** — Who can refer (empty if none yet)
- **Roles** — Single role title as a hyperlink to the job posting

`#`, `Company`, and `Location` cells are vertically merged (vMerge) across rows belonging to the same company so they appear as one cell visually.

## Status Values

| Status | Meaning |
|--------|---------|
| `Search Roles` | Need to find roles |
| `Roles Found` | Have roles, need referrer |
| `Referral Requested` | Sent message, waiting for response |
| `Referral Confirmed` | They agreed to refer |
| `Ready to Apply` | Can apply now |
| `Applied` | Already submitted |
| `Recruiter Call Done` | Had initial call |
| `Interview: [Date]` | Interview scheduled |
| `In Loop: [Dates]` | Onsite scheduled (e.g. Feb 16-17) |
| `Stalled` | No response, recruiter ghosted, blocked |
| `Rejected` | Application rejected |
| `Limited Roles` | Few/no relevant positions |
| `Closed` | Not pursuing |

## Reading the Tracker

**Quick text extraction:**
```bash
unzip -p "${JOBS_DIR}/${TRACKER_FILE}" word/document.xml | sed 's/<[^>]*>//g' | tr -s ' \n' ' '
```

**With python-docx (preserves structure):**
```python
from docx import Document

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

for row in table.rows[1:]:  # Skip header
    cells = [cell.text.strip() for cell in row.cells]
    print(f"{cells[1]}: {cells[3]}")  # Company: Status
```

## Column Validation

**⚠️ ALWAYS validate columns before ANY update!**

```python
from docx import Document

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]
headers = [cell.text.strip() for cell in table.rows[0].cells]
expected = ['#', 'Company', 'Location', 'Status', 'Referrer', 'Roles']

if headers != expected:
    print(f"❌ Column mismatch!")
    print(f"   Expected: {expected}")
    print(f"   Found:    {headers}")
    # STOP - do not proceed
else:
    print("✅ Columns valid")
```

**Column index mapping:**
- 0: # (row number)
- 1: Company
- 2: Location
- 3: Status
- 4: Referrer
- 5: Roles

## Check for Existing Company

**⚠️ ALWAYS check before adding a new row!**

```python
def find_company_rows(doc, company_name):
    """Returns list of row indices for a company (may be multiple), or []"""
    table = doc.tables[0]
    indices = []
    for i, row in enumerate(table.rows[1:], 1):
        if row.cells[1].text.strip().lower() == company_name.lower():
            indices.append(i)
    return indices

# Usage
existing = find_company_rows(doc, "Stripe")
if existing:
    print(f"✅ Found at rows {existing} — ADD new role row for this company")
else:
    print(f"➕ Not found — ADD new company block")
```

## Updating the Tracker

**Update status for a specific role (match by role text or URL):**

```python
from docx import Document
from docx.oxml.ns import qn

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

def get_role_url(row):
    """Return the hyperlink URL from the Roles cell of a row."""
    cell = row.cells[5]
    for para in cell.paragraphs:
        for child in para._p:
            if child.tag == qn('w:hyperlink'):
                r_id = child.get(qn('r:id'))
                if r_id:
                    rel = cell.part.rels.get(r_id)
                    return rel.target_ref if rel else ''
    return ''

target_url = "https://careers.adobe.com/us/en/job/R164562/..."
for row in table.rows[1:]:
    if get_role_url(row) == target_url:
        row.cells[3].paragraphs[0].clear()
        row.cells[3].paragraphs[0].add_run("Applied")
        break

doc.save("${JOBS_DIR}/${TRACKER_FILE}")
```

## Adding a New Company

New company = one row, no vMerge needed yet (single role).

```python
from docx import Document
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

next_num = len(table.rows)  # header + existing data rows
new_row = table.add_row()
new_row.cells[0].paragraphs[0].add_run(str(next_num))
new_row.cells[1].paragraphs[0].add_run("New Company")
new_row.cells[2].paragraphs[0].add_run("Seattle, WA")
new_row.cells[3].paragraphs[0].add_run("Roles Found")
# cells[4] referrer — leave blank
add_hyperlink(new_row.cells[5].paragraphs[0], "Role Title", "https://...")

doc.save("${JOBS_DIR}/${TRACKER_FILE}")
```

## Adding a Role to an Existing Company

When a company already has rows, append a new row after its last row and extend the vMerge on `#`, `Company`, `Location`.

```python
from docx import Document
from docx.oxml.ns import qn
from docx.oxml import OxmlElement
from copy import deepcopy

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

def apply_vmerge(cell, restart):
    tc = cell._tc
    tcPr = tc.find(qn('w:tcPr'))
    if tcPr is None:
        tcPr = OxmlElement('w:tcPr')
        tc.insert(0, tcPr)
    for vm in tcPr.findall(qn('w:vMerge')):
        tcPr.remove(vm)
    vMerge = OxmlElement('w:vMerge')
    if restart:
        vMerge.set(qn('w:val'), 'restart')
    tcPr.append(vMerge)

# 1. Find all rows for this company
company_name = "Adobe"
company_rows = [r for r in table.rows[1:] if r.cells[1].text.strip() == company_name]
last_row = company_rows[-1]

# 2. Clone the last row's TR and insert after it
new_tr = deepcopy(last_row._tr)
last_row._tr.addnext(new_tr)
new_row = next(r for r in table.rows if r._tr is new_tr)

# 3. Clear new row's merged cells (they'll continue the merge)
for col in [0, 1, 2]:
    new_row.cells[col].paragraphs[0].clear()

# 4. Fill status and role
new_row.cells[3].paragraphs[0].clear()
new_row.cells[3].paragraphs[0].add_run("Roles Found")
new_row.cells[5].paragraphs[0].clear()
add_hyperlink(new_row.cells[5].paragraphs[0], "New Role Title", "https://...")

# 5. Re-apply vMerge for all company rows (restart on first, continue on rest)
all_company_rows = [r for r in table.rows[1:] if r.cells[1].text.strip() == company_name]
for i, row in enumerate(all_company_rows):
    for col in [0, 1, 2]:
        apply_vmerge(row.cells[col], restart=(i == 0))

doc.save("${JOBS_DIR}/${TRACKER_FILE}")
```

## Adding Hyperlinks to Roles

```python
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def add_hyperlink(paragraph, text, url):
    part = paragraph.part
    r_id = part.relate_to(url, "http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink", is_external=True)
    hyperlink = OxmlElement('w:hyperlink')
    hyperlink.set(qn('r:id'), r_id)
    new_run = OxmlElement('w:r')
    rPr = OxmlElement('w:rPr')
    c = OxmlElement('w:color')
    c.set(qn('w:val'), '0563C1')
    u = OxmlElement('w:u')
    u.set(qn('w:val'), 'single')
    rPr.append(c)
    rPr.append(u)
    new_run.append(rPr)
    text_elem = OxmlElement('w:t')
    text_elem.text = text
    new_run.append(text_elem)
    hyperlink.append(new_run)
    paragraph._p.append(hyperlink)

# Usage
roles_cell = target_row.cells[5]
roles_cell.text = ""  # Clear existing
para = roles_cell.paragraphs[0]
add_hyperlink(para, "1. Staff SWE", "https://...")
para.add_run("\n")
add_hyperlink(para, "2. Senior SWE", "https://...")
```

## Extracting Role URLs

Each role is its own row. Collect all rows for the company:

```python
from docx.oxml.ns import qn

def get_company_roles(doc, company_name):
    """Extract all roles and per-role status for a company."""
    table = doc.tables[0]
    roles = []
    company_meta = None

    for row in table.rows[1:]:
        name = row.cells[1].text.strip()
        if name.lower() != company_name.lower():
            continue

        if company_meta is None:
            company_meta = {
                'company': name,
                'location': row.cells[2].text.strip(),
                'referrer': row.cells[4].text.strip(),
            }

        # Extract hyperlink from Roles cell
        roles_cell = row.cells[5]
        url, text = '', roles_cell.text.strip()
        for para in roles_cell.paragraphs:
            for child in para._p:
                if child.tag == qn('w:hyperlink'):
                    r_id = child.get(qn('r:id'))
                    if r_id:
                        rel = roles_cell.part.rels.get(r_id)
                        url = rel.target_ref if rel else ''
                    text = ''.join(t.text or '' for t in child.findall('.//' + qn('w:t')))

        roles.append({
            'name': text.strip(),
            'url': url,
            'status': row.cells[3].text.strip(),
        })

    if not company_meta:
        return None
    company_meta['roles'] = roles
    return company_meta
```

## View Commands

When Raki asks about tracker/pipeline:
1. Read current tracker
2. Group by status
3. Summarize what needs action

**Summary format:**
```
📋 Job Pipeline

**Action Needed (3):**
- Anthropic — Roles Found, no referrer
- Stripe — Ready to Apply
- Temporal — Search Roles

**Waiting (2):**
- OpenAI — Referral Requested (Jane)
- Databricks — Referral Requested (Mike)

**In Progress (1):**
- Netflix — Interview: Feb 10

**Applied (4):**
- Google, Meta, Amazon, Apple
```

---

*Tracker operations only. For role discovery, see job-roles. For applications, see job-apply.*
