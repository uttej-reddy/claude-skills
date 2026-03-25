---
name: referral-ask
description: Generate referral request messages for LinkedIn or email. Use when Raki wants to ask someone for a referral at a company.
---

# Referral Ask

Generate personalized referral request messages.

## Shared Data

Profile and file paths: `/Users/uttejreddy/.claude/skills/job-data/profile.json`

## Trigger Phrases

- "Write referral message for [Company]"
- "Ask [Person] for referral at [Company]"
- "LinkedIn message for [Company] referral"
- "Referral request for [Company]"

## Workflow

1. **Look up company in tracker** — Get roles list (use job-tracker skill)
2. **Ask for contact info:**
   - "Who's your contact at [Company]? (Name + how you know them)"
   - "LinkedIn message or email?"
3. **Generate message** — Personalized, professional, not pushy
4. **Present for review** — Let Raki edit before sending
5. **Update tracker** — Add referrer name, change status to "Referral Requested"

## Message Templates

### LinkedIn Connection Request (300 char limit)

```
Hi [Name]! Hope you're doing well at [Company]. I'm exploring opportunities there and found some roles that align with my background (backend/infra/AI). Would you be open to a quick chat or possibly referring me? Happy to share my resume. Thanks!
```

### LinkedIn InMail / Email (Full Version)

```
Subject: Engineering Roles at [Company] — Referral Inquiry

Hi [First Name],

[OPENING — see variations below]

I'm a Staff Software Engineer at Expedia with 12+ years of experience in backend systems, distributed infrastructure, and AI platforms.

I'm very interested in [Company]'s mission around [COMPANY FOCUS] and found these roles that align well with my background:

1. [Role 1]
2. [Role 2]
3. [Role 3]
4. [Role 4]
5. [Role 5]

Would you be open to referring me to any of these roles? I'd also appreciate it if you could help set up a quick chat with the team's recruiter if possible.

Thank you for your time!

Best,
Uttej ${LAST_NAME}
https://${GITHUB_HANDLE}.github.io/

Role Links:
1. [URL 1]
2. [URL 2]
3. [URL 3]
4. [URL 4]
5. [URL 5]
```

## Opening Variations

**Ex-colleague:**
```
Hope things are going great at [Company]! It's been a while since [shared context — project, team, etc.].
```

**LinkedIn connection (met briefly):**
```
We connected a while back at [event/context]. Hope you're doing well!
```

**Friend of friend:**
```
[Mutual friend] mentioned you're at [Company] and thought I should reach out.
```

**2nd degree / Recruiter:**
```
I came across your profile while researching opportunities at [Company] and noticed we share a mutual connection.
```

**Same background (cold):**
```
I noticed we're both [UH alumni / from same region / worked at similar companies]...
```

## Closing Variations

**Close contact:**
```
Would you be able to refer me?
```

**Acquaintance:**
```
Would you be open to referring me, or pointing me to someone who might help?
```

**Cold outreach:**
```
If you're comfortable, I'd appreciate any guidance or a referral. No pressure either way!
```

## Key Points

**⚠️ ALWAYS include:**
- Portfolio URL: **https://${GITHUB_HANDLE}.github.io/** (has resume, GitHub, LinkedIn)
- Role URLs as plain text at the end (hyperlinks don't work on all platforms)
- Ask for BOTH: referral + recruiter intro if possible

**Character limits:**
- LinkedIn connection request: 300 chars
- LinkedIn InMail: 1900 chars
- Email: No limit

## Generating Role List

Pull roles from tracker and format:

**In message body:**
```
1. Staff SWE, Platform Infrastructure
2. Senior SWE, Distributed Systems
3. Staff SWE, AI/ML Platform
```

**URLs section (at end):**
```
Role Links:
1. https://boards.greenhouse.io/company/jobs/123456
2. https://boards.greenhouse.io/company/jobs/234567
```

If URLs aren't in tracker, use job-roles skill to fetch them first.

## After Sending

**Update tracker immediately:**
```python
row.cells[3].text = "Referral Requested"   # Status
row.cells[4].text = "[Contact Name]"       # Referrer
```

**Status progression:**
1. `Roles Found` → Roles discovered, no referrer yet
2. `Referral Requested` → Message sent, waiting for response
3. `Referral Confirmed` → They agreed to refer
4. `Applied` → Application submitted

## Review Format

Present to Raki before sending:

```
📤 **LinkedIn Message for [CONTACT] @ [COMPANY]**

---

**Subject:** Engineering Roles at [Company] — Referral Inquiry

---

[Full message text]

---

✅ Looks good
✏️ Edit (tell me what to change)
🗑️ Discard
```

## Customization

Raki may request adjustments:
- Different tone (formal vs casual)
- Shorter version for connection requests
- Specific talking points to include
- Different ask (just referral, just chat, both)

**Always apply requested changes while keeping the core structure.**

---

*Referral messages only. For tracker updates, see job-tracker. For applications, see job-apply.*
