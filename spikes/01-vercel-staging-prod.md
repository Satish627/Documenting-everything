# Spike 01 — Vercel staging + prod (+ secrets)

**Task:** Plan #1 / Guide #3 · **Status:** Draft · **Mode:** design + research (no system access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*

> Goal from the onboarding guide: *"You can push to `dev` and see it on the staging domain; prod
> is isolated."* This spike de-risks **how** before we touch any real Vercel account.

---

## Problem

We need two fully separated environments — **staging** and **production** — so that testing never
touches live users, live customer funds, or live vendor endpoints (NIUM, Postmark, Penneo, Slack,
PLUNK, HubSpot). This is the foundation the other 8 tasks deploy through, and it's what makes
non-negotiable #3 ("staging is the only safe playground") actually enforceable.

The real risk isn't the domain routing — Vercel makes that easy. The risk is a **staging deploy
accidentally calling a vendor's production API** (e.g. Postmark emailing real customers, or a NIUM
sandbox call hitting live). So this spike is really about **environment separation + secret hygiene**,
with domain routing as the easy part.

## Questions to answer

1. How do we map git branches → environments → domains?
2. How are environment variables scoped per environment so staging can't reach prod vendors?
3. How do we guarantee staging points at **sandbox/test** vendor endpoints, not production?
4. What needs a real login to finish (the 🔒 items)?

## Findings (design against the assumed stack)

### 1. Branch → environment → domain

Vercel has three built-in environment types: **Production**, **Preview**, **Development**.
Proposed mapping:

| Git branch | Vercel environment | Domain |
|---|---|---|
| `main` | Production | `app.franklin...` (prod) |
| `dev` | Preview (promoted to a fixed staging alias) | `staging.franklin...` |
| feature branches | Preview | auto-generated preview URLs |

- `main → prod` and `dev → staging` is exactly the guide's ask.
- Vercel auto-creates a preview deployment per push; we pin `dev`'s preview to a **stable staging
  alias** so the staging domain is predictable.
- **Confirm on day 1:** the real domain names and whether a `staging.` subdomain already exists.

### 2. Environment variables, scoped per environment

Vercel lets each env var be set **separately for Production / Preview / Development**. This is the
core safety mechanism: the *same* variable name resolves to different values per environment.

```
POSTMARK_SERVER_TOKEN   → prod:  <live token>     preview: <test/sandbox token>
DATABASE_URL            → prod:  <prod db>         preview: <staging db>
NIUM_API_BASE_URL       → prod:  <live base>       preview: <sandbox base>
NIUM_API_KEY            → prod:  <live>            preview: <sandbox>
PENNEO_API_BASE_URL     → prod:  <live>            preview: <sandbox>
SLACK_WEBHOOK_URL       → prod:  <#alerts>         preview: <#alerts-staging>
PLUNK_API_KEY           → prod:  <live>            preview: <test>
HUBSPOT_TOKEN           → prod:  <live>            preview: <sandbox>
```

- **Non-negotiable #2:** these are names only. No real values live in the repo, in this doc, or in chat.
- **Staging must use a separate database** — never point staging at the prod DB (non-negotiable #3, #4).

### 3. Guaranteeing staging ≠ production vendors (the real risk)

Scoped env vars aren't enough on their own — a missing staging value could silently fall back or a
handler could be hardcoded. Defensive measures to propose:

- Add a single `APP_ENV` var (`production` | `staging`) and a startup **assertion**: if
  `APP_ENV=staging` but any vendor base URL looks like a production endpoint, **fail fast** on boot.
- Centralize all vendor clients behind one config module that reads base URLs from env — no vendor
  URL hardcoded anywhere in feature code.
- For the highest-risk vendors (Postmark, NIUM, Penneo), prefer providers' **explicit sandbox modes**
  over just swapping a key.

### 4. What needs access (🔒 day-1)

- Creating the Vercel project + connecting the GitHub repo.
- Setting the Production/Preview env var values.
- Configuring the real domains / staging alias.
- Confirming which vendors offer a true sandbox vs. only a separate key.

## Recommendation

Go with **branch-scoped environments + per-environment env vars + an `APP_ENV` fail-fast assertion**.
The assertion is the cheap insurance that turns "we set the vars carefully" into "staging *cannot*
call prod." Domain routing is the trivial part and follows Vercel defaults.

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design). Both come from the same day-1 conversation with my manager.

**Access to request** (→ Foundation block in `ACCESS_CHECKLIST.md`)

- [ ] GitHub repo access + Vercel team access — or confirmation the project already exists so I get added.
- [ ] The production domain + a `staging.` subdomain — or the go-ahead to create them.
- [ ] A staging database connection string — or the go-ahead to provision one.

**Facts to confirm** (information, not credentials)

- [ ] Is this greenfield, or is there already a Vercel project + repo wired up?
- [ ] Which vendors have a real sandbox mode vs. only a second API key? (Shapes Spikes 03, 04, 07, 08.)
- [ ] Do the domains / staging DB already exist, or am I setting them up from scratch?

## Next steps

- [ ] Confirm the stack + repo location with manager.
- [ ] Turn the env-var table into a real `.env.example` (names only) once the stack is confirmed.
- [ ] Draft the `APP_ENV` fail-fast check as a small snippet in the local Next.js skeleton.
