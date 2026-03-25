---
name: job-apply
description: Apply to jobs via browser automation. Use when Uttej wants to submit applications, fill job forms, or apply to roles at a company.
---

# Job Apply

Browser automation for job applications.

## Shared Data

Profile and file paths: `/Users/uttejreddy/.claude/skills/job-data/profile.json`

## User Preferences

**⚠️ FOLLOW THESE RULES:**

1. **No excessive screenshots** — Provide text summaries of form state
2. **Use Playwright MCP** — Browser automation via `playwright` MCP server (Chrome, headed)
3. **Resume upload on Lever = ALWAYS manual** — Don't attempt automation
4. **Batch manual steps** — Fill everything automatable, THEN ask user once
5. **Know the profile** — Use documented values, don't ask what you already know
6. **ALWAYS write essay questions** — Even if marked optional

## Trigger Phrases

- "Apply to [Company]"
- "Submit applications for [Company]"
- "Let's apply to [Company]"
- "Fill out application for [Role]"

## Application Workflow

### Step 1: Lookup Roles

Get roles from tracker (use job-tracker skill):
```python
roles = get_company_roles(doc, "Anthropic")
# Returns: company, location, status, referrer, roles with URLs
```

### Step 2: Check Cover Letters

For each role, check if cover letter exists:
```
ravi_jobs/[Company]/[Role_Short_Name]_Cover.docx
```

**If missing:** Use cover-letter skill to generate, then return here.

### Step 3: Confirm with Uttej

```
🚀 Ready to apply to [Company]

Roles (5):
1. Senior SWE, Infrastructure ✅ Cover letter ready
2. Staff SWE, Cloud Inference ✅ Cover letter ready
3. Staff SWE, Inference ✅ Cover letter ready
4. Staff+ SWE, Infrastructure ✅ Cover letter ready
5. Staff+ SWE, Data Infrastructure ✅ Cover letter ready

Profile: ✅ Account exists
Resume: ${RESUME_FILE}

Proceed? (yes/no)
```

### Step 4: Apply to Each Role

1. Open role URL
2. Click Apply
3. Login if needed
4. Fill form (see Form Filling below)
5. Upload resume (manual on Lever)
6. Review all fields
7. **Confirm with Uttej before submit**
8. Submit

### Step 5: Update Tracker

After successful applications:
```python
target_row.cells[3].text = "Applied"
doc.save(tracker_path)
```

## Form Filling — Profile Values

**⚠️ USE THESE EXACT VALUES:**

| Field | Value |
|-------|-------|
| Full Name | ${FULL_NAME} |
| First Name | ${FIRST_NAME} |
| Last Name | ${LAST_NAME} |
| Preferred Name | **Uttej** |
| Email | **${EMAIL}** |
| Phone | ${PHONE} |
| Location | ${LOCATION} |
| Street Address | 923 N 98th ST |
| City | Seattle |
| State | Washington (or WA) |
| Postal Code | 98103 |
| County | King |
| Company | ${CURRENT_COMPANY} |
| LinkedIn | https://www.linkedin.com/in/${LINKEDIN_HANDLE}/ |
| GitHub | https://github.com/${GITHUB_HANDLE} |
| Portfolio | https://${GITHUB_HANDLE}.github.io |
| University | ${UNIVERSITY} |
| Work Auth | Yes (authorized) |
| Sponsorship | Yes (needs H-1B) |
| Visa Status | ${VISA_STATUS} |
| Languages | English, Hindi, Telugu |
| Veteran | I am not a Protected Veteran |
| Disability | **No, I do not have a disability** |
| Pronunciation | ${NAME_PRONUNCIATION} |

## Form Filling Best Practices

1. **Resume First** — Upload before other fields (may auto-fill)
2. **Verify Each Field** — Check email, preferred name after filling
3. **Re-fill after resume upload** — Auto-fill may overwrite values
4. **Check for Essays** — Fill them (see Essay Writing below)
5. **Dropdowns** — Verify selection actually saved
6. **Final Review** — List ALL fields before submit

## Essay Writing

**⚠️ ALWAYS read resume + job description FIRST!**

### Common Essay Questions

**"Favorite project / proudest accomplishment"**
- Pick project from resume that matches role themes
- Don't make up details — use actual numbers

**"Why this company?"**
- Reference specific things from job description
- Show genuine interest in their mission

### Project Matching

| Job Theme | Best Project |
|-----------|-------------|
| AI/ML, Agents | AI Agents Marketplace (2025) |
| Infrastructure, Scale | API Gateway Platform (2024-25) |
| Platform, DevEx | Control Plane (2022-24) |
| NLP, Automation | Virtual Agent Platform (2019-21) |
| Messaging, Reliability | Conversation Platform (2018-19) |

### Essay Structure

1. **Opening** — State the project and your role
2. **Challenge** — What made it hard/interesting
3. **Your contribution** — Specific technical decisions
4. **Impact** — Numbers, scale, outcomes
5. **Lesson** — What you learned (ties to their role)

**DO:**
- Use actual project details from resume
- Show scale/impact with numbers
- Describe technical decisions

**DON'T:**
- Say "this matches your role" directly
- Make up projects or embellish
- Write generic content

### Reading the Resume

```bash
python3 -c "from pypdf import PdfReader; r = PdfReader('${JOBS_DIR}/${RESUME_FILE}'); print(''.join(p.extract_text() for p in r.pages))"
```

## Portal-Specific: Lever

**⚠️ LEVER QUIRKS:**

1. **Resume upload does NOT work via automation** — User must upload manually
2. **Long dropdowns** — Use `select` action with exact value
3. **"How did you hear"** — Select "Company Website" or "LinkedIn"
4. **Signature fields** — Look for Name + Date at bottom
5. **"Additional information"** — OPTIONAL, skip unless needed

**Lever Checklist:**
- [ ] Fill basic info (name, email, phone, location, company)
- [ ] Fill links (LinkedIn, GitHub, Portfolio)
- [ ] Select languages (English, Hindi, Telugu)
- [ ] Fill Preferred Name: `Uttej`
- [ ] Fill Pronunciation
- [ ] Select Work Auth: Yes | Sponsorship: Yes
- [ ] Select University: `${UNIVERSITY}`
- [ ] Select "How heard": Company Website / LinkedIn
- [ ] Write essays if required
- [ ] Select Veteran status
- [ ] Select Disability
- [ ] Fill signature Name + Date
- [ ] **ASK USER TO UPLOAD RESUME**
- [ ] Verify ALL fields
- [ ] Submit

**University dropdown (Lever):**
```
browser action=act request={"kind": "select", "ref": "<combobox_ref>", "values": ["${UNIVERSITY}"]}
```

## Portal-Specific: Greenhouse

- Standard file upload usually works
- May have "Apply with LinkedIn" option
- Test upload before asking for manual help

## Portal-Specific: Workday

- Account-based — login first
- Profile auto-fill from account
- Multi-page forms

## Portal-Specific: Oracle

**⚠️ ORACLE-SPECIFIC QUIRKS:**

1. **Email verification required** — Sends code to email, must enter before continuing
2. **Multi-step flow** — Personal Info → Experience → More About You
3. **State/County dropdowns** — May need exact values (e.g., "Washington" not "WA")
4. **Education before Experience** — Form enforces this order
5. **Resume upload** — Usually works with standard upload, but verify
6. **Skills section** — Pre-suggests skills, add relevant ones
7. **Long text fields** — May have character limits (check before pasting)
8. **Save frequently** — Progress can be lost on back-navigation

**Oracle Experience Entry Format:**
```
Company: Expedia, Inc.
Location: ${LOCATION_SHORT}
Title: Senior Software Development Engineer
Start: June 2018
End: Present (check "Current")
Description: [Combined description of all projects/roles]
```

**Oracle Disability Options:**
- "Yes, I have a disability..."
- "No, I do not have a disability..."  ← USE THIS ONE
- "I do not wish to answer"

## Resume Upload

**Resume location:**
```
${JOBS_DIR}/${RESUME_FILE}
```

**Lever workaround:**
1. Fill ALL other fields first
2. Ask user to manually upload resume
3. Verify "Success!" appears
4. Re-verify form fields (auto-fill may overwrite)
5. Then submit

**Greenhouse/Workday:** Test automation first, fallback to manual.

## Browser Commands

Uses the `playwright` MCP server (Chrome, headed mode).

**Navigate:**
```
mcp__playwright__browser_navigate url="https://jobs.lever.co/company/role-id"
```

**Snapshot (find elements):**
```
mcp__playwright__browser_snapshot
```

**Click:**
```
mcp__playwright__browser_click element="Apply button" ref="<ref>"
```

**Type:**
```
mcp__playwright__browser_type element="email input" ref="<ref>" text="${EMAIL}"
```

**Select dropdown:**
```
mcp__playwright__browser_select_option element="dropdown" ref="<ref>" values=["Option Text"]
```

**Upload file:**
```
mcp__playwright__browser_file_chooser paths=["/path/to/file.pdf"]
```

**Screenshot:**
```
mcp__playwright__browser_screenshot
```

## Company Profiles

Track portal accounts in `skills/job-apply/portals.json`:

```json
{
  "anthropic": {
    "portal_url": "https://job-boards.greenhouse.io/anthropic",
    "portal_type": "greenhouse",
    "has_account": true,
    "notes": "Uses Google OAuth"
  }
}
```

## Pitfalls & Lessons Learned

**⚠️ HARD-WON LESSONS FROM FAILED ATTEMPTS:**

### 1. PDF Resume Reading

PDFs are binary — **Read tool shows gibberish**. Extract text with pypdf:

```bash
python3 -c "from pypdf import PdfReader; r = PdfReader('${JOBS_DIR}/${RESUME_FILE}'); print(''.join(p.extract_text() for p in r.pages))"
```

### 2. Email Verification Mid-Application

Some portals (Oracle, Workday) send verification codes during application.
- **Watch for:** "Enter code sent to your email"
- **Action:** Pause, ask Uttej for the code, then continue

### 3. Job Title Accuracy

**⚠️ Uttej's title at Expedia is "Senior Software Engineer" — NOT Principal/Staff!**

Don't assume titles from role descriptions. Use exact title from resume.

### 4. Multiple Roles at Same Company

When adding work experience, **combine all roles at same company into ONE entry**:
- Company: Expedia, Inc.
- Title: Senior Software Development Engineer
- Dates: June 2018 – Present
- Description: List all projects chronologically (AI Agents, API Gateway, Control Plane, etc.)

**DON'T** add separate entries for each internal role/project.

### 4a. Multiple Clients at Same Consulting Company (e.g. Hindsight)

Some employers are consulting firms where Uttej worked across multiple clients. The form has ONE description box for the whole employer — paste ALL clients into it in order, formatted exactly as they appear in the resume:

```
Client: [Client Name]  -  [Location]  [Date Range]
• [bullet]
• [bullet]

Client: [Client Name]  -  [Location]  [Date Range]
• [bullet]
```

Read the resume to get all clients and their bullets. Do NOT only use the first client.

### 5. ALL Work History Required

Many applications want **complete work history**, not just current job:

| Order | Company | Title | Dates |
|-------|---------|-------|-------|
| 1 | Expedia | Senior SDE | June 2018 – Present |
| 2 | Insperity | Business App Developer | Jan 2016 – June 2018 |
| 3 | Yatra TG Stays | Senior Software Engineer | July 2012 – July 2014 |

**Additional/Research (if form has section):**
- UH Research Assistant (Aug 2014 – Jan 2016)
- UH Software Developer (Jan 2016 – June 2016)

### 6. Education Required First

Some forms require education BEFORE work experience:

| Degree | School | Dates |
|--------|--------|-------|
| MS Computer Science | University of Houston | 2014 – 2016 |
| BE Computer Science | BITS Pilani – KK Birla Goa Campus | 2007 – 2012 |

### 7. Address Field Breakdown

Address may be split into multiple fields:
- **Street:** 923 N 98th ST
- **City:** Seattle
- **State:** WA / Washington
- **Postal Code:** 98103
- **County:** King (sometimes required!)

### 8. Skills from Suggestions

Forms may show suggested skills. Add relevant ones from resume:
- SQL, Java, Python, JavaScript/TypeScript
- Kubernetes, AWS, Docker, Terraform
- Kafka, Redis, PostgreSQL, DynamoDB
- Spring Boot, REST APIs

**DON'T add skills you can't back up in an interview.**

### 9. Multi-Step Application Forms

Enterprise portals (Oracle, Workday) have multiple steps:

```
Step 1: Personal Info → Name, Contact, Address
Step 2: Experience → Education, Work History, Skills, Resume
Step 3: More About You → EEO, Disability, Veteran, etc.
```

**Save progress between steps.** Some portals lose data on back-navigation.

### 10. Google Drive Long Paths

The actual resume path through Google Drive:
```
${JOBS_DIR}/${RESUME_FILE}
```

**Symlink shortcut:**
```
${JOBS_DIR}/${RESUME_FILE}
```

Use the symlink when possible, but fall back to full path if needed.

## Error Handling

| Issue | Action |
|-------|--------|
| Form field not found | Snapshot, describe to Uttej |
| Login failed | Prompt for manual login |
| Upload failed | Try alternative or manual |
| CAPTCHA | Pause, ask Uttej to solve |
| Rate limited | Wait and retry |
| Verification code | Ask Uttej for email code |
| Field validation error | Read error, fix value |
| Dropdown won't select | Try typing + arrow keys |

## Safety

**ALWAYS:**
- Confirm with Uttej before submitting
- Take screenshot before final submit
- Pause on unexpected fields
- Review ALL fields before clicking submit

**NEVER:**
- Auto-submit without confirmation
- Skip cover letter review
- Apply to roles not in tracker
- Assume fields are correct

## Multi-Role Applications

When applying to multiple roles at same company:

1. **First application:** Fill everything, ask for manual resume upload
2. **Subsequent:** Resume may auto-fill from session — check first
3. **Reuse essays** if similar roles, or tailor if different

---

*Application automation only. For role discovery, see job-roles. For cover letters, see cover-letter. For tracker, see job-tracker.*
