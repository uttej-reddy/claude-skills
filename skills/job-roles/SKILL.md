---
name: job-roles
description: Discover and search for job roles at companies. Use when finding roles at a company, searching career pages, or adding roles to the tracker.
---

# Job Roles

Discover and search for relevant roles at target companies.

## Shared Data

**FIRST STEP: Read `/Users/uttejreddy/.claude/skills/job-data/profile.json` before doing anything else. Use its values to replace all `${PLACEHOLDER}` variables throughout this skill.**

| Placeholder | JSON key |
|---|---|
| `${LOCATION_SHORT}` | `location_short` |
| `${JOBS_DIR}` | `jobs_dir` |
| `${TRACKER_FILE}` | `tracker_file` |

## Trigger Phrases

- "Find roles at [Company]"
- "Search [Company] careers"
- "What roles at [Company]?"
- "Add [Company] to my list"

## Workflow

**⚠️ CRITICAL: Never auto-discover. Always ask first.**

When Uttej invokes role discovery, ask:

> "New company on your mind, or picking from pending list?"

### If New Company

1. Ask which company
2. Search careers page / job boards
3. Present relevant roles (filtered by criteria below)
4. Ask if any look good to add to tracker

### If Pending from List

1. Read tracker, find companies with status "Search Roles"
2. Ask which one to explore
3. Search that company's careers page
4. Present roles

## Search Strategy

**Search queries:**
1. `[Company] careers software engineer staff principal`
2. `[Company] jobs backend distributed systems`
3. Check company careers page directly (usually `[company].com/careers`)

**Common career page patterns:**
- `boards.greenhouse.io/[company]`
- `jobs.lever.co/[company]`
- `[company].wd5.myworkdayjobs.com`
- `careers.[company].com`

## Role Selection Criteria

**Target roles:**
- Senior Software Engineer
- Staff Software Engineer
- Backend Engineer
- Distributed Systems Engineer
- Platform Engineer
- AI/ML Platform (not ML Engineer titles)

**Location filter:** ${LOCATION_SHORT} or Remote (US)

**Select 5 roles per company**

## Sorting by Job ID

**Higher Job ID = more recently posted**

Sort ALL roles by Job ID descending:
```
1. SWE, Product Security (ID: 7545459)   ← highest = most recent
2. Staff SWE, Platform (ID: 7517258)
3. Principal SWE, Infra (ID: 7409691)
4. SWE, Backend (ID: 7396679)
5. SWE, Data (ID: 7277110)               ← lowest = oldest
```

## Presenting Roles

Format for Uttej:

```
🔍 **[Company] Roles** (Seattle / Remote US)

1. **Staff SWE, Platform Infrastructure**
   https://boards.greenhouse.io/company/jobs/123456

2. **Senior SWE, Distributed Systems**
   https://boards.greenhouse.io/company/jobs/234567

3. **Staff SWE, AI Platform**
   https://boards.greenhouse.io/company/jobs/345678

4. **Principal Engineer, Backend**
   https://boards.greenhouse.io/company/jobs/456789

5. **Senior SWE, Data Infrastructure**
   https://boards.greenhouse.io/company/jobs/567890

---
Add all to tracker? Or pick specific ones?
```

## Adding to Tracker

After Uttej confirms roles:

1. **Check if company exists** (see job-tracker skill)
2. **Update or add row** with:
   - Company name
   - Location (Seattle / Remote US)
   - Status: `Roles Found`
   - Roles: Numbered list with hyperlinks

**Use job-tracker skill for the actual update.**

## Web Search Tips

**Greenhouse boards:**
```
site:boards.greenhouse.io "[company]" "staff software engineer" "seattle"
```

**Lever boards:**
```
site:jobs.lever.co "[company]" "backend" "remote"
```

**LinkedIn (fallback):**
```
site:linkedin.com/jobs "[company]" "staff engineer" "seattle"
```

## Browser Automation (if needed)

For dynamic career pages that need scrolling/filtering:

```python
# Navigate to careers page
browser action=open url="https://boards.greenhouse.io/anthropic"

# Take snapshot to find filter controls
browser action=snapshot

# Apply location filter if available
browser action=act request={"kind": "click", "ref": "location-filter"}
browser action=act request={"kind": "type", "ref": "search", "text": "Seattle"}
```

## Exclusion Rules

**Skip these titles:**
- Machine Learning Engineer (unless "ML Platform")
- Data Scientist
- Research Scientist
- Frontend Engineer (unless Full Stack)
- Mobile Engineer
- QA/Test Engineer
- Support Engineer
- Solutions Engineer (unless technical)

**Skip these locations:**
- Non-US
- US locations other than Seattle/Remote (unless Uttej says otherwise)

---

*Role discovery only. For tracker updates, see job-tracker. For applications, see job-apply.*
