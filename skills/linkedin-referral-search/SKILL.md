---
name: linkedin-referral-search
description: Search LinkedIn for potential referrers at a target company. Use when Raki wants to find people who can refer him at a company.
---

# LinkedIn Referral Search

Find potential referrers at target companies via LinkedIn search.

## Trigger Phrases

- "Find referrers at [Company]"
- "Who do I know at [Company]"
- "LinkedIn search for [Company]"
- "Find connections at [Company]"

## Workflow

### Step 1: Confirm Target

Ask Raki:
> "Looking for referrers at **[Company]**. Any specific criteria?"
> - Titles: Engineering, Staff+, Manager?
> - Location: Seattle preferred?
> - Connection degree: 1st, 2nd, or both?

### Step 2: Open LinkedIn

```
browser action=start profile="openclaw"
browser action=open url="https://www.linkedin.com/search/results/people/"
```

**Check if logged in.** If not, pause and ask Raki to log in manually.

### Step 3: Search with Filters

**Build search URL:**
```
https://www.linkedin.com/search/results/people/?keywords=[Company]&network=["F","S"]&origin=FACETED_SEARCH
```

**URL Parameters:**
| Param | Value | Meaning |
|-------|-------|---------|
| `keywords` | Company name | Search term |
| `network` | `["F"]` | 1st connections only |
| `network` | `["S"]` | 2nd connections only |
| `network` | `["F","S"]` | 1st + 2nd connections |
| `geoUrn` | `103644278` | United States |
| `currentCompany` | Company ID | Filter by current employer |

### Step 4: Apply Filters via UI

1. **Navigate** to people search
2. **Enter** company name in search
3. **Click** "All Filters"
4. **Select** "Connections" → 1st and 2nd
5. **Select** "Current company" → [Target Company]
6. **Apply** filters

**Use human-like delays between actions (3-5 seconds).**

### Step 5: Extract Candidates

Take snapshot, extract:
- Name
- Title
- Location
- Mutual connections count
- Profile URL

**Limit to first 10 results.** Don't paginate aggressively.

### Step 6: Present to Raki

```
🔍 **Potential Referrers at [Company]**

**1st Connections (3):**
1. **Jane Doe** — Staff SWE @ [Company]
   📍 Seattle · 👥 Direct connection
   🔗 linkedin.com/in/janedoe

2. **John Smith** — Engineering Manager @ [Company]
   📍 SF · 👥 Direct connection
   🔗 linkedin.com/in/johnsmith

**2nd Connections (5):**
3. **Alice Wong** — Senior SWE @ [Company]
   📍 Seattle · 👥 12 mutual connections
   🔗 linkedin.com/in/alicewong

...

---
Who should I draft a message for?
```

### Step 7: Handoff to referral-ask

When Raki picks someone:
1. Get their profile URL
2. Use **referral-ask** skill to draft message
3. Open their LinkedIn profile for Raki to send

## Safety Rules

**⚠️ DO:**
- Use visible browser (`profile="openclaw"`)
- Add delays between actions (3-5 seconds)
- Limit to 10-15 results per search
- Stop if CAPTCHA or "unusual activity" warning appears

**⚠️ DON'T:**
- Paginate through hundreds of results
- Run multiple searches in rapid succession
- Scrape/save large amounts of data
- Use headless browser

## Rate Limiting

| Action | Safe Limit |
|--------|-----------|
| Searches per session | 5-10 |
| Profile views per session | 15-20 |
| Delay between actions | 3-5 seconds |
| Sessions per day | 2-3 |

## Finding Company ID (for filters)

If needed for advanced filtering:
1. Go to company page: `linkedin.com/company/[company-name]`
2. View page source or network requests
3. Look for company ID (numeric)

Or just use the keyword search — usually sufficient.

## Common Titles to Prioritize

| Priority | Titles |
|----------|--------|
| High | Staff SWE, Principal Engineer, Engineering Manager |
| Medium | Senior SWE, Tech Lead |
| Lower | SWE, Junior Engineer |

People at higher levels often have more referral influence.

## Mutual Connections

**Prioritize by mutual connections:**
- 10+ mutual → Warm intro possible
- 1-9 mutual → Can mention mutual connection
- 0 mutual (2nd degree) → Cold but still valid

## Error Handling

| Issue | Action |
|-------|--------|
| "Sign in required" | Pause, ask Raki to log in |
| CAPTCHA | Stop, ask Raki to solve |
| "Commercial use limit" | Stop for the day |
| No results | Try broader search, different keywords |

## Integration with Tracker

After finding referrer, update job tracker:
1. Add referrer name to "Referrer" column
2. Keep status as "Roles Found" until message sent
3. After message sent → "Referral Requested"

---

*Search only. For messaging, use referral-ask skill.*
