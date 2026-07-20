# Franklin — Operations Engineer workspace

Read this before doing anything in this repo. It is the standing context and the
non-negotiables for every session. `ONBOARDING_PLAN.md` holds the actual task backlog.

## Who / what this is

- **Satish** is joining **Franklin** as an **Operations Engineer** (starts ~late July 2026).
- **Franklin** is a Copenhagen-based **regulated fintech** — a financial platform for
  Danish D2C e-commerce (corporate payment cards, cashback, expense management + bookkeeping).
- Franklin is **not a bank**. It rides on partner banks:
  - **NIUM** — VISA principal member; issues cards, runs biometric verification, sends webhooks.
  - **Banking Circle** — holds customer funds in ring-fenced accounts.
- Real customer money, KYC, disputes, PII, and GDPR/EU obligations are all in scope.

## What this folder is (and is not)

- A **prep & planning workspace**, used **before** Satish has any system access.
- Everything produced here is **planning, spike docs, research, and mock-data prototypes**.
- **No live systems. No real credentials. No production data.** None of that is available yet.
- **Assumed stack** when writing code or examples: **Next.js + TypeScript on Vercel,
  PostgreSQL**. Treat this as an assumption to confirm on day 1 — flag anything that depends on it.

## Non-negotiables (these override speed)

1. **Safety before speed.** This is regulated fintech with real customer money and KYC data.
   Nothing customer-facing auto-sends; nothing touches funds, cards, or KYC state without a
   human explicitly confirming. **Draft, never send.**
2. **Secrets never get committed or pasted.** Postmark / NIUM / Penneo / Slack / PLUNK / HubSpot
   keys live in environment variables only — never hardcoded, never in git, never pasted into chat.
3. **Staging is the only safe playground.** `main → prod`, `dev → staging`. No script, migration,
   or webhook test ever points at a production endpoint or the production database. Prove it on
   staging first.
4. **Real customer data stays out of local files, logs, and chat.** Use synthetic/test data for
   prototypes. (The interview case's sample tickets are good synthetic data.)
5. **Customer-facing copy follows Franklin voice + the customer's language** (see below).
6. **Connect, don't reinvent.** Much infra already exists partially — the job is wiring existing
   pieces together with the tools already chosen. No new vendors/frameworks without a clear reason.
7. **Spike before building anything unclear.** A short spike doc (≈5 min to read, ≤30 min research)
   before committing to an approach.

## Working style (Franklin culture)

- Extreme ownership: if you spot it and it's unclear who owns it, you own it.
- Ship the smallest safe version: **make it work → make it pretty → make it fast.**
- Communication short, direct, kind. Come with a proposed solution, not just the problem.

## Franklin voice (for any customer-facing copy)

- Warm, human, first-name basis. Open `Hej [navn]`, sign off as a named person
  (`De bedste hilsner, Caroline fra Franklin`) — never "The Support Team".
- Plain language over jargon. Break multi-step things into a numbered checklist with **bold** actions.
- **Danish is primary; reply in English when the customer writes English.** Always match their language.
- One light emoji max, in greeting/sign-off — **never** when money or trust is at risk
  (fraud, frozen funds, failed payments).
- **Never** invent account-specific facts (balances, fees, refund status) the draft can't verify,
  and **never** give financial or legal advice.

## Key vendors — what each is for

| Vendor | Role |
|---|---|
| **Postmark** | Transactional email (templates, sending) |
| **PLUNK** | Marketing email / nudges / automation flows |
| **Vercel** | Hosting & deploys (`main → prod`, `dev → staging`) |
| **NIUM** | Partner bank: cards, biometric verification, webhooks |
| **Banking Circle** | Partner bank: ring-fenced customer funds |
| **Penneo** | E-signature (documents → signing flow) |
| **HubSpot** | CRM + support tickets |
| **Slack** | Internal alerts/notifications |
| **Retool** | Internal admin (being migrated *away from* for critical alerts) |
