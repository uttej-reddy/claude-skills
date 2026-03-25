---
name: cover-letter
description: Write tailored cover letters for job applications. Use when Raki needs a cover letter for a specific role or company.
---

# Cover Letter

Write tailored cover letters for job applications.

## Shared Data

Profile and file paths: `/Users/uttejreddy/.claude/skills/job-data/profile.json`

## Folder Structure

```
ravi_jobs/
├── ${RESUME_FILE}          # Master resume
├── Microsoft_Cover_Letter.docx               # Example template
├── Salesforce_Agentforce_Principal_Backend_Cover.docx
├── Salesforce_AI_Forward_Deployed_Engineer_Cover.docx
├── Salesforce_Distributed_Systems_Cover.docx
├── [Company]/                                # Company folder
│   ├── [Role1]_Cover.docx
│   └── [Role2]_Cover.docx
```

**Root:** `${JOBS_DIR}`

## Trigger Phrases

- "Write cover letter for [Company] [Role]"
- "Cover letter for [Role]"
- "Draft a letter for [Company]"

## Workflow

1. **Identify the role** — by company + role name or URL
2. **Read the job description** — fetch from careers page if needed
3. **Read existing cover letters** — for style reference
4. **Read resume** — for accurate project details
5. **Write tailored letter** — match achievements to role requirements
6. **Save to company folder** — create folder if needed
7. **Confirm location** to Raki

## Reading the Resume

**⚠️ ALWAYS read the resume before writing — don't make up projects!**

```bash
python3 -c "from pypdf import PdfReader; r = PdfReader('${JOBS_DIR}/${RESUME_FILE}'); print(''.join(p.extract_text() for p in r.pages))"
```

## Example Cover Letters

Read these for style/tone reference:
- `Microsoft_Cover_Letter.docx`
- `Salesforce_Agentforce_Principal_Backend_Cover.docx`
- `Salesforce_Distributed_Systems_Cover.docx`

## Style Guide

**Opening:** Start with referral mention if applicable, then state the role.
```
Hi, I'm reaching out upon referral from [Name] to apply for [Position] at [Company]...
```
OR
```
Hi, I'm applying for the [Position] at [Company]...
```

**Body - Key Achievements to Highlight:**

| Project | Year | Key Themes | Numbers |
|---------|------|------------|---------|
| **AI Agents Marketplace** | 2025 | Multi-agent, Claude SDK, A2A protocol | 6 agents, 80% effort reduction, 3 months, 5 engineers |
| **API Gateway Platform** | 2024-25 | Spring Cloud Gateway, UDP model | 20 engineers, 10K+ TPS, 10x scalable |
| **Control Plane** | 2022-24 | Config management, partner onboarding | 75% faster (3mo→10d) |
| **Virtual Agent Platform** | 2019-21 | NLU, skill graphs | 35% auto-resolution |
| **Conversation Platform** | 2018-19 | Kafka, omnichannel | millions msgs/day, 99.9% uptime |

**Match project themes to role requirements (indirectly, don't be obvious)**

**Tone:** Direct, confident, achievement-focused. No fluff. Show impact with numbers.

**Closing:** Express eagerness, tie back to company mission.

## Writing Process

### Step 1: Read the Job Description

Identify key themes:
- What tech stack?
- What scale?
- What problems?
- What team culture?

### Step 2: Pick Relevant Projects

Match job themes → project themes:

| Job Theme | Best Project Match |
|-----------|-------------------|
| AI/ML, Agents | AI Agents Marketplace |
| Infrastructure, Scale | API Gateway Platform |
| Platform, DevEx | Control Plane |
| NLP, Automation | Virtual Agent Platform |
| Messaging, Reliability | Conversation Platform |

### Step 3: Write the Letter

**Structure:**
1. Opening — Role + referral (if any)
2. Relevant experience paragraph (2-3 achievements)
3. Why this company/role
4. Closing

**Length:** 250-400 words

### Step 4: Save

```bash
# Create company folder if needed
mkdir -p "${JOBS_DIR}/[Company]"
```

**Filename:** `[Role_Short_Name]_Cover.docx`

Example: `Staff_GenAI_System_Engineer_Cover.docx`

## Creating the DOCX

```python
from docx import Document

doc = Document()
doc.add_paragraph("Hi,")
doc.add_paragraph("")
doc.add_paragraph("[Opening paragraph]")
doc.add_paragraph("")
doc.add_paragraph("[Body paragraph - achievements]")
doc.add_paragraph("")
doc.add_paragraph("[Why company paragraph]")
doc.add_paragraph("")
doc.add_paragraph("[Closing]")
doc.add_paragraph("")
doc.add_paragraph("Best,")
doc.add_paragraph("Uttej ${LAST_NAME}")

doc.save("${JOBS_DIR}/[Company]/[Role]_Cover.docx")
```

## Template

```
Hi,

I'm applying for the [POSITION] at [COMPANY]. [Optional: I was referred by NAME.]

[ACHIEVEMENT 1 - most relevant project with numbers and impact]

[ACHIEVEMENT 2 - supporting experience that shows breadth]

[WHY COMPANY - specific reason tied to their mission/product/challenge]

I'd welcome the opportunity to discuss how my experience aligns with [COMPANY]'s goals.

Best,
Uttej ${LAST_NAME}
```

## Review Format

Present to Raki before saving:

```
📝 **Cover Letter: [Company] - [Role]**

---

[Full cover letter text]

---

✅ Save to `[Company]/[Role]_Cover.docx`
✏️ Edit (tell me what to change)
🗑️ Discard
```

---

*Cover letters only. For applications, see job-apply. For tracker, see job-tracker.*
