# Franklin Onboarding — Prioritized Task Plan

Your starting plan for the 9 tasks in `documents/satish_onboarding_guide.pdf`.
Read `CLAUDE.md` first — it holds the non-negotiables every task must respect.

## How to use this

- **You have no system access yet.** So the value *right now* is in the **Prep** column of each
  task: spikes, research, architecture sketches, draft copy, and mock-data prototypes. The
  **Needs access** column lists what waits until day 1.
- **Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL. Confirm on day 1.
- **Spike doc rule:** for anything unclear, write a short spike doc (≈5 min read, ≤30 min research)
  before proposing a build. Keep them in a `spikes/` folder.
- Legend: 🟢 do now (no access) · 🔒 needs access · ⭐ priority tier.

## Recommended sequence

Ordered by **priority + dependency + build-confidence**, not by the guide's numbering.

| # | Task | Guide # | Tier | Why here |
|---|------|---------|------|----------|
| 1 | Vercel staging + prod | 3 | ⭐ P0 foundation | Everything ships safely through this; enables the "staging only" rule |
| 2 | Nudge emails (incomplete onboarding) | 1 | ⭐ P1 | Explicitly **High Priority** in the guide; core business value |
| 3 | Month-end close reminder template | 2 | ⭐ P1 quick win | Shares Postmark with #2; small, self-contained |
| 4 | NIUM biometric notifications | 4 | ⭐ P1 critical | Trigger point for card setup; **unblocks task #5 below** |
| 5 | Penneo power-of-attorney automation | 9 | ⭐ P2 critical | Biggest integration; depends on NIUM (#4) |
| 6 | Slack notifications (migrate off Retool) | 6 | ⭐ P2 | Reliability of critical alerts (failed payouts, risk) |
| 7 | Platform banner system | 5 | ⭐ P3 quick win | Standalone, smallest scope; good confidence builder |
| 8 | PLUNK setup (marketing/nudges) | 7 | ⭐ P3 | Marketing automation; overlaps conceptually with #2 |
| 9 | RFI automate support responses | 8 | ⭐ P3 | Most ambiguous + AI-heavy; closest to the interview case |

---

## Day-1 readiness checklist (do before you start)

- [ ] Draft an **access request list** — one line per system, why you need it, what level:
      Postmark, PLUNK, Vercel, NIUM, Banking Circle, Penneo, HubSpot, Slack, the database, Retool, GitHub/repo.
- [ ] Collect and skim the **API docs** for each vendor (see `CLAUDE.md` table).
- [ ] Stand up a **local Next.js + TypeScript skeleton** to prototype against (mock data only).
- [ ] Set up a `spikes/` folder and a note-taking habit.
- [ ] List the **open questions** per task (below) to align with your manager in week 1.

---

## The tasks

### 1. Vercel: staging + prod  *(guide #3 — P0 foundation)*
**Goal:** `dev` deploys to a staging domain; `main` deploys to prod; prod is isolated.

- 🟢 **Prep:** Diagram the branch→environment→domain flow. List the env vars each vendor
  integration will need (names only, no values). Write a spike on secret management on Vercel
  (per-environment env vars) and how staging avoids hitting live vendor endpoints.
- 🔒 **Needs access:** Create the two Vercel environments, wire env vars, configure domain routing.
- **Success:** push to `dev` → visible on staging; prod untouched.

### 2. Nudge emails for incomplete onboarding  *(guide #1 — P1, High Priority)*
**Goal:** Daily/weekly job emails users stuck mid-onboarding; they come back and finish.

- 🟢 **Prep:** Sketch the pipeline (query "incomplete" users → personalize → send via Postmark →
  scheduled job). Draft the **Danish + English** email copy in Franklin voice (name, which steps
  are missing, CTA into dashboard). Define what "incomplete onboarding state" means as a query
  (assumptions doc). Prototype the personalization with mock rows.
- 🔒 **Needs access:** Real onboarding-status schema, Postmark keys, scheduler wiring.
- **Success:** email goes out, users return and finish. *Non-negotiable: human-reviewed before any real send.*

### 3. Postmark "Month-End Close Reminder" template  *(guide #2 — P1 quick win)*
**Goal:** Template lives in Postmark and can be test-sent.

- 🟢 **Prep:** Write **two versions** — V1 with dynamic data (categorization progress, receipts),
  V2 generic fallback. Danish + English, Franklin voice, light emoji (this isn't money-at-risk),
  CTA to dashboard. Define the dynamic fields V1 needs.
- 🔒 **Needs access:** Build in Postmark template builder; test-send.
- **Success:** template in Postmark, test send works.

### 4. NIUM biometric notifications  *(guide #4 — P1 critical)*
**Goal:** Biometric completion fires a notification within 30s, triggering the next workflow.

- 🟢 **Prep:** Study NIUM webhook docs; spike on which event = "biometric done". Design the
  notification target (Slack vs email vs dashboard — recommend + justify). Map the downstream
  trigger (card creation / settlement / **the Penneo LOA in task #5**). Draft webhook handler
  pseudocode + idempotency/retry thinking.
- 🔒 **Needs access:** NIUM sandbox/webhooks, a test account to complete biometrics.
- **Success:** complete biometric in a test account → notification within 30s.

### 5. Penneo power-of-attorney automation  *(guide #9 — P2 critical, depends on #4)*
**Goal:** User triggers signing → PDF generated → Penneo signing flow → on signed, auto-upload to NIUM.

- 🟢 **Prep:** Map the full chain (DB pull user+company → generate PDF from template → Penneo
  upload + signing link → Penneo webhook on signed → download → upload to NIUM → status in
  dashboard). Spike a PDF generator library choice. Draft the document template. Identify every
  failure point and where a human must stay in the loop (this is legal/financial — highest caution).
- 🔒 **Needs access:** Penneo API, NIUM upload endpoint, real user/company data.
- **Success:** trigger → email link → sign in Penneo → on NIUM within ~30s. *Draft/verify, never auto-fire against prod.*

### 6. Slack notifications — migrate off Retool  *(guide #6 — P2)*
**Goal:** Critical events (failed payout, risk alert) hit Slack within 10s, reliably.

- 🟢 **Prep:** List the critical event types and a message template per type. Spike: DB triggers vs
  an event-logger/worker to detect events. Design direct Slack webhook integration + retry logic
  if Slack is down. Prototype message formatting with mock events.
- 🔒 **Needs access:** Event source (DB/logger), Slack webhook URL.
- **Success:** critical event → Slack within 10s.

### 7. Platform banner system  *(guide #5 — P3 quick win)*
**Goal:** Admin writes a message; it shows to all live users; error/warning/info types; dismissable.

- 🟢 **Prep:** Design the banner component (top-of-app, severity variants, auto-dismiss/manual close)
  and where config lives (DB vs env). Sketch the admin interface (likely Retool). Build a mock
  component in the local skeleton.
- 🔒 **Needs access:** Retool admin, DB/config store, deploy.
- **Success:** write "We're doing maintenance" in admin → shows to all live users.

### 8. PLUNK setup — marketing & nudges  *(guide #7 — P3)*
**Goal:** Log into PLUNK, see segments, send a test email to yourself.

- 🟢 **Prep:** Study PLUNK docs. Draft a **segmentation strategy** (e.g. `has card`,
  `incomplete onboarding`, `dormant`). Design automation flows (welcome series, dormant nudges) and
  a template library outline. Note the overlap/boundary with task #2 (Postmark nudges vs PLUNK).
- 🔒 **Needs access:** PLUNK account, user data for segments.
- **Success:** segments visible in PLUNK, test email received.

### 9. RFI — automate support responses  *(guide #8 — P3, most ambiguous)*
**Goal:** Incoming RFI auto-categorized, routed to the right person, templated response ready.

- 🟢 **Prep:** This is closest to the AI Support Draft Engine case in `documents/`. Reuse that
  thinking: ingest → classify → route → draft → **human review** → send. Draft RFI-type categories,
  routing rules, and response templates. Define **when to draft vs. stay silent and escalate**
  (compliance-sensitive RFIs). Prototype categorization on mock RFIs.
- 🔒 **Needs access:** Support platform / HubSpot integration, real RFI history.
- **Success:** RFI comes in → categorized → right person notified → templated response sendable.
  *Non-negotiable: no auto-send; human reviews compliance-sensitive replies.*

---

## Open questions to align on in week 1

- What exactly defines "incomplete onboarding state" in the DB? (task #2)
- Postmark vs PLUNK — which owns which emails? (tasks #2, #3, #8)
- Preferred notification channel for NIUM biometric — Slack, email, or dashboard? (task #4)
- Which events count as "critical" for Slack alerts, and where's the event source? (task #6)
- Confirm the stack (Next.js/TS/Postgres?) and where the app repo lives.
- How much of each integration already exists ("connecting pieces")?
