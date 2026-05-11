# Lemlist Campaign Dashboard — Workflow Instructions

This is the source-of-truth workflow for updating the campaign dashboard.
Run this workflow each time a new Lemlist campaign analysis is available in Slack.

---

## SOURCE OF TRUTH — SLACK

Channel: DM Jason Baudier (ID: U08DC8EFEKD)

Find the MOST RECENT message that contains one of:
- "Weekly Lemlist Campaign Analysis"
- "Weekly Lemlist Analysis"

Additional signals (at least 2 must be present):
- contains "CAM-"
- contains "batch"
- contains "reply rate"
- contains "segment"

If the analysis is split across multiple parts (e.g. Part 1/3, 2/3, 3/3), read ALL parts sent on the same date and treat them as one combined analysis.

Rules:
- Only process the latest valid message set (same date)
- Ignore replies unless they contain full analysis
- Ignore noise / unrelated messages

If no valid message is found: STOP and return an error

---

## STEP 1 — EXTRACT DATA

Extract ALL relevant data from the message.

**CAMPAIGN (per CAM code):**
- campaign ID (e.g. CAM-XXX)
- batch name
- region / country
- week / date range
- tone (if available)

**KPIs:**
- total contacts
- total messages
- reply rate
- positive reply rate
- bounce rate
- any other KPI mentioned

**SEGMENTS:**
For each ICP / segment:
- name, code, contacts, deliverability, reply rate, max reply reference, color, performance note

**INSIGHTS:**
- takeaways (ALL)
- what worked / what didn't work
- next actions

**REPLIES:**
For EVERY individual reply mentioned in the message, extract:
- campaign (CAM-XXX)
- channel (Email or LinkedIn)
- date
- sender (Florent, Henry, Axel, etc.)
- contact (email address or anonymized ID)
- contactName (if mentioned or inferable)
- org (organization, if mentioned)
- text (verbatim reply text)
- tone — classify into ONE of:
  - `hot` — explicit meeting request, confirmed availability, clear "yes"
  - `warm` — positive engagement, soft interest, reciprocal invite, regret about missing
  - `not_interested` — explicit or soft rejection, "not a fit", schedule conflict
  - `already_equipped` — mentions a competitor tool or existing solution
  - `ambiguous` — opener only ("Hi Henry,"), single emoji, no content captured
  - `wrong_target` — contact says they are not the right person
  - `bug` — contact addresses sender by wrong name (template variable error)
- interpretation — 1–2 sentence analysis of what this reply signals about tone, ICP fit, and messaging
- messagingLink — optional, 1 sentence linking the reply to a messaging insight (why the hook worked or didn't)

**KPI SNAPSHOT:**
- batch-level metrics
- global-level metrics if present

---

## STEP 2 — TRANSFORM INTO DASHBOARD FORMAT

Convert extracted data into the EXACT structure used in `index.html`.

Rules:
- Translate everything into English
- Follow EXACT same object structure as existing entries
- Do NOT invent missing data
- Keep values realistic and consistent

### CAMPAIGNS object fields:
id, batch, region, week, tone, kpis (6 cards), segments, takeaways, leads, whatWorked, whatDidnt, nextActions, kpiSnapshot (batch + global), sidebarBadges (3 items)

### REPLIES object fields:
```
{
  campaign: 'CAM-XXX',
  channel: 'Email' | 'LinkedIn',
  date: 'Apr XX, 2026',
  sender: 'Florent' | 'Henry' | 'Axel' | ...,
  contact: 'email@domain.com' | 'lea_XXXX',
  contactName: 'First Last',
  org: 'Organization Name',
  text: 'Verbatim reply text',
  tone: 'hot' | 'warm' | 'not_interested' | 'already_equipped' | 'ambiguous' | 'wrong_target' | 'bug',
  interpretation: 'Analysis of tone and intent...',
  messagingLink: '💬 Insight about the hook or messaging...' | null,
}
```

---

## STEP 3 — OPEN AND UPDATE FILE

Open `index.html`. Two arrays must be updated: CAMPAIGNS and REPLIES.

### A. CAMPAIGNS array (line ~232)

**If a campaign with the same `id` does NOT exist → INSERT:**
- Insert the new campaign object at **index 0** (top of array)
- Do NOT delete or reorder previous campaigns

**If a campaign with the same `id` ALREADY EXISTS → UPDATE:**
- Update the following fields with data from the new analysis:
  - `batch` — extend name if new batches are covered (e.g. "All Batches + Pre-Event")
  - `week` — extend date range if new data goes beyond the current range
  - `kpis` — update values to reflect the latest available data
  - `segments` — add new segments; update existing segment notes and metrics
  - `leads` — add new hot/warm leads; do NOT remove existing ones
  - `takeaways` — prepend new takeaways; keep existing ones
  - `whatWorked` — prepend new items
  - `whatDidnt` — prepend new items
  - `nextActions` — prepend urgent new actions; keep existing ones
  - `kpiSnapshot` — update batch and global metrics
  - `sidebarBadges` — update to reflect the most relevant current state
- Do NOT remove or overwrite data that is still accurate
- Do NOT touch `id`, `region`, `tone` unless they have changed

**Campaign separation rule:** One dashboard entry = one unique CAM code. Never merge different CAM codes.

### B. REPLIES array (located after CAMPAIGNS)

**This section is ALWAYS updated, whether campaigns are new or existing.**

**Duplicate check:** Before inserting each reply, check if a reply with the same `campaign` + `contact` + `date` already exists in the array.

If it already exists → SKIP.

If new:
- Insert the new reply object at **index 0** (top of the REPLIES array)
- Do NOT modify existing reply entries
- Do NOT delete or reorder previous replies
- Preserve formatting and syntax EXACTLY

---

## STEP 4 — VALIDATION

Before saving:
- Ensure valid JavaScript syntax
- Ensure valid HTML structure
- Ensure no missing commas or brackets
- Ensure both CAMPAIGNS and REPLIES arrays are intact

If any doubt: STOP and do not save

---

## STEP 5 — SAVE AND COMMIT

- Save the file
- Commit with message: `"update: add latest lemlist campaign analysis"`
- Push to branch `claude/adoring-mendel-SxkxA`
- If direct push to `main` is blocked: create a PR from a clean branch and merge it immediately

Do NOT modify any other file.

---

## GLOBAL SAFETY RULES

- Do NOT rewrite the whole file
- Do NOT modify HTML structure, CSS, or chart/layout code
- Do NOT guess missing data
- Do NOT hallucinate metrics
- The `<meta name="robots" content="noindex, nofollow">` tag must be present in `<head>`

---

## CAMPAIGN SEPARATION RULES

If the Slack message contains multiple CAM codes (e.g. CAM-004 and CAM-005):
- Create one separate CAMPAIGNS entry per code
- Each code gets its own KPIs, segments, takeaways, sidebar entry
- Replies are attributed to their correct CAM code individually

---

## EXPECTED RESULT

- New campaign(s) inserted at top of CAMPAIGNS (if new CAM code)
- Existing campaign(s) updated with latest data (if CAM code already exists)
- New replies added at top of REPLIES (if not duplicate)
- Previous reply entries unchanged
- Dashboard fully functional

---

## FINAL OUTPUT (MANDATORY)

After completion, return:
- Campaign ID(s) added (or skipped if duplicate)
- Number of new replies added
- Total campaigns in dashboard
- Total replies in dashboard
