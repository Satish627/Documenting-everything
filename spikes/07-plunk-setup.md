# Spike 07 — PLUNK setup (marketing & nudges)

**Task:** Plan #8 / Guide #7 · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*
**Boundary with:** [Spike 02](02-nudge-emails.md) (Postmark nudges) — see ownership note below.

> Goal from the guide: *log into PLUNK, see all our segments, send a test email to yourself.*
> PLUNK is the email marketing/automation service for campaigns and nudges.

---

## Problem

We want targeted email campaigns and lifecycle automations without hand-building everything. PLUNK
provides segments, flows, and templates. The real design work is **what segments**, **which flows**,
and **where PLUNK ends and Postmark begins** — so we don't send two overlapping "finish onboarding"
emails from two systems.

## Questions to answer

1. Where's the line between PLUNK and Postmark?
2. What segments do we need?
3. What automation flows and templates?
4. What consent/GDPR rules apply?

## Findings

### 1. Postmark vs PLUNK (the key boundary)

| System | Owns |
|---|---|
| **Postmark** | **Transactional / system** email — the onboarding nudges (Spike 02), month-end reminders, receipts. Tied to a user's own account state. |
| **PLUNK** | **Marketing / lifecycle** — campaigns, dormant-user re-engagement, feature announcements, welcome series. Consent-based. |

**Recommendation:** keep onboarding nudges in **Postmark** (they're transactional and per-user), and
use **PLUNK** for consent-based marketing. Document this split so we never double-send. *Confirm with
the team — this is a judgment call worth aligning on early.*

### 2. Segments (behaviour-based)

Starter set: `has_card`, `incomplete_onboarding`, `dormant` (no activity in N days),
`active_high_spend`, `no_card_yet`. Each maps to a query over our data synced into PLUNK.

### 3. Automation flows + templates

- **Welcome series** (post-onboarding), **dormant re-engagement**, **feature announcement** campaigns.
- A small **template library** in Franklin voice (Danish + English), reusing the voice rules in `CLAUDE.md`.

### 4. Consent / GDPR

- Marketing email needs a lawful basis + unsubscribe. Segment membership must respect marketing
  consent and suppression — never email someone who opted out (non-negotiable: privacy/EU).

## Recommendation

Set up PLUNK for **consent-based marketing only**, with a clear behavioural segment scheme and a
documented **Postmark = transactional / PLUNK = marketing** split. Draft the segment definitions and
a couple of flows now; wiring the data sync waits for access.

## Open questions (align in week 1)

- Confirm the Postmark/PLUNK ownership split.
- How does user data get into PLUNK (sync, API, manual)?
- Where is marketing consent stored, and what's the unsubscribe mechanism?
- Which campaigns does marketing actually want first?

## Next steps

- [ ] Draft segment definitions + the two or three highest-value flows.
- [ ] Write the ownership-split doc (Postmark vs PLUNK) and align with the team.
- [ ] Draft a starter template in Franklin voice (Danish + English).
