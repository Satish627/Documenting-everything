# Spike 02 — Nudge emails for incomplete onboarding

**Task:** Plan #2 / Guide #1 (**High Priority**) · **Status:** Draft · **Mode:** design + draft copy (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL, Postmark for sending — *confirm on day 1.*

> Goal from the guide: *"Email goes out, people come back and finish onboarding."* We lose people
> halfway through; a smart, personalized nudge after X days lifts completion.

---

## Problem

Some users start onboarding and stall before finishing. We want an automated, personalized email
that reminds them what's left and pulls them back to finish — without spamming, and without a
machine ever sending something wrong to a real customer. Four moving parts: **(1)** a query that
defines "stuck", **(2)** personalization (name + which steps remain), **(3)** a scheduled job,
**(4)** Postmark templates in Franklin voice, Danish + English.

## Questions to answer

1. What exactly is an "incomplete onboarding state"?
2. What's the nudge **cadence**, and when do we **stop**?
3. How do we personalize safely (name + missing steps) without inventing account facts?
4. What are the templates — and in which language?

## Findings

### 1. Defining "incomplete onboarding state" (assumptions — confirm on day 1)

Model onboarding as an ordered checklist. A user is "incomplete" if they created an account but
haven't finished all required steps. **Assumed steps** (confirm against the real schema):

1. Account created
2. Company details added
3. **KYC / biometric verification** (NIUM)
4. **Power of attorney signed** (Penneo)
5. Account funded / first top-up
6. First card created

Proposed query shape (pseudocode, to validate against the real schema):

```sql
-- users who started but haven't completed all required steps,
-- and whose last activity is older than the nudge threshold
select u.id, u.first_name, u.email, u.language, u.created_at, o.completed_steps
from users u
join onboarding_status o on o.user_id = u.id
where o.completed = false
  and u.created_at < now() - interval '2 days'
  and (o.last_nudge_at is null or o.last_nudge_at < now() - interval '3 days')
  and u.marketing_suppressed = false;   -- respect unsubscribes/bounces
```

> The exact table/column names are the **biggest open question** — see below.

### 2. Cadence + stop conditions

A short, escalating sequence beats an indefinite drip:

| Nudge | Trigger | Tone |
|---|---|---|
| 1 | 2 days after signup, still incomplete | friendly reminder |
| 2 | 5 days | "here's exactly what's left" + offer help |
| 3 | 10 days | last nudge, low-pressure |

**Stop immediately when** any of: onboarding completed · user unsubscribed · email bounced/complained ·
3 nudges already sent. Record `last_nudge_at` + `nudge_count` so the job is **idempotent** — never
double-sends on a re-run. Scheduled daily (Vercel Cron), which naturally covers the day-based thresholds.

### 3. Safe personalization

- Merge only: **first name** and the **list of remaining steps** (rendered from `completed_steps`
  as plain-language labels). Nothing else.
- **Never** state balances, fees, or any account figure (non-negotiable #5) — a nudge only says
  *which steps remain*, never account specifics.
- Render remaining steps as a numbered checklist with **bold** actions (Franklin voice).
- Language from `u.language`; default Danish.

### 4. Draft templates (Franklin voice)

Onboarding is a friendly context (not money-at-risk), so one light emoji is fine. Merge fields in `{{ }}`.

**Danish (primary)**

```
Emne: Du er næsten i mål med Franklin, {{firstName}} 👋

Hej {{firstName}}

Du er kun få skridt fra at komme helt i gang med Franklin. Her er, hvad der mangler:

{{#each remainingSteps}}
- **{{this.label}}** — {{this.why}}
{{/each}}

Det tager typisk under 10 minutter. Klik her, så guider vi dig det sidste stykke:

👉 Fortsæt din opsætning: {{dashboardUrl}}

Sidder du fast, eller er noget uklart? Sig endelig til — så hjælper vi dig igennem det.

De bedste hilsner,
Caroline fra Franklin
```

**English (when `language = en`)**

```
Subject: You're almost set up with Franklin, {{firstName}} 👋

Hej {{firstName}}

You're just a few steps away from being fully up and running with Franklin. Here's what's left:

{{#each remainingSteps}}
- **{{this.label}}** — {{this.why}}
{{/each}}

It usually takes under 10 minutes. Pick up where you left off here:

👉 Finish your setup: {{dashboardUrl}}

Stuck, or something unclear? Just say the word and we'll walk you through it.

Best regards,
Caroline from Franklin
```

Example rendered step labels (plain language, with the *why*): *"**Bekræft din identitet** — så vi kan
aktivere din konto"* / *"**Underskriv fuldmagten** — sidste skridt før dit kort er klar."*

## Recommendation

Build it as: a **daily Vercel Cron job** → runs the "incomplete + due for nudge" query →
renders the per-user remaining-steps list → sends via **Postmark templates** (Danish/English) →
writes back `last_nudge_at` / `nudge_count`. Keep the step definitions and copy in one place so
they stay consistent with the actual product flow. **Draft/review before any real send** — first
live run goes to a test/internal address only (non-negotiable #1).

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design).

**Access to request** (→ Plan #2 block in `ACCESS_CHECKLIST.md`)

- [ ] `POSTMARK_SERVER_TOKEN` — **test** token first (live only at go-live).
- [ ] Postmark account access (to build/inspect templates).

**Facts to confirm** (information, not credentials)

- [ ] The real onboarding tables/columns, and the canonical list of required steps.
- [ ] Where `language` is stored, and whether a suppression/unsubscribe flag already exists.
- [ ] Postmark vs PLUNK — which owns these onboarding nudges? (Boundary with Spike 07.)
- [ ] Cadence: are the 2/5/10-day thresholds sensible for our funnel, or is there data to tune them?
- [ ] Is this **transactional** or **marketing** under consent rules — do we need an unsubscribe link?

## Next steps

- [ ] Confirm the onboarding schema + required steps with manager.
- [ ] Turn the step list into a shared `remainingSteps` definition in the local skeleton (mock data).
- [ ] Prototype the render (mock user rows → rendered Danish + English email) — no sending.
- [ ] Build the two Postmark templates once access lands; test-send to an internal address first.
