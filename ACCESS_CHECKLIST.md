# Access Request Checklist (phased)

Request access **task by task**, not all at once. Each block below lists only what that task needs —
so when you start a task, you ask for that block and nothing more. Keeps requests small, justified,
and easy for your manager to approve.

Blocks are numbered by **`Plan #`** (the sequence in `ONBOARDING_PLAN.md`) so the references line up
across every file. `Plan #1` is the foundation you need before anything else.

## How to use this

- Work top-down. Only open the block for the task you're **actually starting**.
- Most keys come in **two values**: a **test/sandbox** one (for staging) and a **live** one (for prod).
  You often only need the *test* value first — request live when you're ready to go to production.
- Keys go into **Vercel's env settings, scoped per environment** — never committed to the repo
  (the repo only holds `.env.example` with names). See `CLAUDE.md` non-negotiable #2.
- `APP_ENV` you set yourself in Vercel — it's not a request.
- Legend: 🔑 secret/key · 🖥️ account/tool access · 🌐 domain · 🗄️ database.

---

## Plan #1 — Foundation: Vercel staging + prod *(Guide #3)*
*One-time — request before anything else. Enables [Spike 01](spikes/01-vercel-staging-prod.md).*

- [ ] 🖥️ **GitHub repo access** to the Franklin app (write/PR).
- [ ] 🖥️ **Vercel team access**, with the app project connected to the GitHub repo.
- [ ] 🌐 **Production domain** + a **`staging.` subdomain**.
- [ ] 🗄️ **Staging database** connection string (`DATABASE_URL`, staging).
- [ ] 🗄️ **Production database** connection string (`DATABASE_URL`, prod) — can wait until first prod deploy.

> After this you can create both environments and set `APP_ENV` yourself.

---

## Plan #2 — Nudge emails *(Guide #1)*
*Request when you start building the nudge flow.*

- [ ] 🔑 **`POSTMARK_SERVER_TOKEN`** — **test** token first (live later, at go-live).
- [ ] 🗄️ Confirmation of the **onboarding-status schema** (tables/columns for "incomplete") — you
      already have the staging DB; this is about knowing the shape.
- [ ] (Scheduler = Vercel Cron — no key needed.)

## Plan #3 — Month-end close template *(Guide #2)*
*Request when you build the template.*

- [ ] 🖥️ **Postmark template access** (same Postmark account as Plan #2 — likely nothing new to request).

## Plan #4 — NIUM biometric notifications *(Guide #4)*
*Request when you start the webhook handler.*

- [ ] 🔑 **`NIUM_API_BASE_URL`** + **`NIUM_API_KEY`** — **sandbox** first.
- [ ] 🔑 **NIUM webhook signing secret** (to verify incoming webhooks).
- [ ] 🖥️ A **NIUM test account** you can push through biometric verification.
- [ ] 🔑 **`SLACK_WEBHOOK_URL`** (staging channel) — *this task notifies via Slack, so you may want
      the staging Slack webhook early even though Slack is formally Plan #6.*

## Plan #5 — Letter of Authorization / Penneo power-of-attorney *(Guide #9)*
*Request when you start the signing flow.*

- [ ] 🔑 **`PENNEO_API_BASE_URL`** + **`PENNEO_API_KEY`** — **sandbox** first.
- [ ] 🔑 **Penneo webhook secret** (to verify the "signed" webhook).
- [ ] 🖥️ The **NIUM upload endpoint** details for filing the signed doc (you already have NIUM keys from Plan #4).
- [ ] 📄 The **LOA template** + its required fields (from whoever owns the legal doc).

## Plan #6 — Slack notifications *(Guide #6)*
*Request when you build the shared notifier.*

- [ ] 🔑 **`SLACK_WEBHOOK_URL`** — **staging channel** (`#alerts-staging`) and **prod channel** (`#alerts`).
- [ ] 🖥️ Access to the **event source** (whatever Retool monitors today) so you know what to detect.

## Plan #7 — Platform banner *(Guide #5)*
*Request when you build the banner.*

- [ ] 🖥️ **Retool access** (to build the admin surface).
- [ ] 🖥️ Franklin **design guidelines** for the severity styles.
- [ ] (Config lives in the DB you already have.)

## Plan #8 — PLUNK *(Guide #7)*
*Request when you set up marketing automation.*

- [ ] 🖥️ **PLUNK account access**.
- [ ] 🔑 **`PLUNK_API_KEY`** — **test** first.
- [ ] 🖥️ How **user data syncs into PLUNK**, and where **marketing consent** is stored.

## Plan #9 — RFI support automation *(Guide #8)*
*Request when you start the RFI pipeline.*

- [ ] 🖥️ **HubSpot access** + 🔑 **`HUBSPOT_TOKEN`** — **sandbox** first.
- [ ] 🖥️ The **RFI intake channel(s)** (email/portal) and any **resolved-RFI history** to ground drafts.

---

## Quick copy-paste for your manager (Foundation)

> Hi — starting on the Vercel staging/prod setup. To get going I'll need: GitHub repo access
> (write), Vercel team access with the project linked to the repo, our production domain + a
> `staging.` subdomain, and a **staging** database connection string. I'll ask for individual
> vendor keys (Postmark, NIUM, etc.) as I reach each task, starting with a **Postmark test token**
> for the nudge emails. Thanks!
