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

**Columns (in order):**
| # | Company | Location | Status | Referrer | Roles |
|---|---------|----------|--------|----------|-------|

- **#** — Row number
- **Company** — Company name
- **Location** — Seattle, Remote US, etc.
- **Status** — Current tracking state (see values below)
- **Referrer** — Who can refer (empty if none yet)
- **Roles** — Role titles (numbered list with hyperlinks)

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
def find_company(doc, company_name):
    """Returns row index if found, None if not"""
    table = doc.tables[0]
    for i, row in enumerate(table.rows[1:], 1):
        if row.cells[1].text.strip().lower() == company_name.lower():
            return i
    return None

# Usage
existing = find_company(doc, "Stripe")
if existing:
    print(f"✅ Found at row {existing} — UPDATE this row")
else:
    print(f"➕ Not found — ADD new row")
```

## Updating the Tracker

```python
from docx import Document

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

# 1. Validate columns
headers = [cell.text.strip() for cell in table.rows[0].cells]
assert headers == ['#', 'Company', 'Location', 'Status', 'Referrer', 'Roles']

# 2. Find row by company
company_name = "Anthropic"
target_row = None
for row in table.rows[1:]:
    if row.cells[1].text.strip() == company_name:
        target_row = row
        break

# 3. Update specific columns
if target_row:
    target_row.cells[3].text = "Applied"  # Status
    doc.save("${JOBS_DIR}/${TRACKER_FILE}")
```

## Adding a New Company

```python
from docx import Document

doc = Document("${JOBS_DIR}/${TRACKER_FILE}")
table = doc.tables[0]

# Get next row number
next_num = len(table.rows)

# Add row
new_row = table.add_row()
new_row.cells[0].text = str(next_num)
new_row.cells[1].text = "New Company"
new_row.cells[2].text = "Seattle"
new_row.cells[3].text = "Search Roles"
new_row.cells[4].text = ""  # Referrer
new_row.cells[5].text = ""  # Roles

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

```python
from docx.oxml.ns import qn

def get_company_roles(doc, company_name):
    """Extract roles and URLs for a company"""
    table = doc.tables[0]
    
    for row in table.rows[1:]:
        if row.cells[1].text.strip().lower() == company_name.lower():
            roles = []
            roles_cell = row.cells[5]
            
            for para in roles_cell.paragraphs:
                for elem in para._p.iter():
                    if elem.tag.endswith('hyperlink'):
                        r_id = elem.get(qn('r:id'))
                        if r_id:
                            rel = doc.part.rels.get(r_id)
                            url = rel.target_ref if rel else None
                            text = ''.join(t.text for t in elem.iter() if t.text)
                            roles.append({'name': text.strip(), 'url': url})
            
            return {
                'company': row.cells[1].text.strip(),
                'location': row.cells[2].text.strip(),
                'status': row.cells[3].text.strip(),
                'referrer': row.cells[4].text.strip(),
                'roles': roles
            }
    return None
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
