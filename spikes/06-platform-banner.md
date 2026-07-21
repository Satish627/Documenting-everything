# Spike 06 — Platform banner system

**Task:** Plan #7 / Guide #5 (**quick win**) · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*

> Goal from the guide: *go to an admin panel, write "We're doing maintenance right now" → it shows
> to all live users.* Types: error / warning / info. Dismissable. Config in DB or env.

---

## Problem

When something's down or maintenance is happening, users should see a clear banner instead of
silence. We need a top-of-app component with severity styles, a way for a non-engineer to set the
message, and a config source that updates **without a redeploy**.

## Questions to answer

1. Where does the banner config live?
2. How do clients pick up a change quickly?
3. What does the admin surface look like?

## Findings

### 1. Config source — recommend **DB-backed**, not env

- Env vars require a redeploy to change → too slow for "post a maintenance notice right now".
- A small `banners` table (or a single active-banner row) lets ops change the message instantly.
- Fields: `id`, `message` (Danish + English), `severity` (`error|warning|info`), `active`,
  `dismissible`, `starts_at`, `ends_at`, `updated_by`.

### 2. Client pickup

- App fetches the active banner on load and revalidates on an interval (e.g. SWR / short poll) so a
  change reaches live users within a minute without a deploy.
- Respect `starts_at` / `ends_at` for scheduled maintenance windows.
- Dismiss: manual close persists per-user (localStorage keyed by banner id) so it doesn't nag;
  auto-dismiss after a set time optional per banner.

### 3. Admin surface — Retool

- The guide points at Retool for admin. A tiny Retool app over the `banners` table (set message,
  severity, active on/off, window) is the fastest path — no custom admin UI needed.

### 4. Component

- Renders at the top of the app; color/icon by severity. Keep the design aligned with Franklin's
  design guidelines. Bilingual message chosen by user language.

## Recommendation

**DB-backed banner config + a small Retool admin over it + a top-of-app component that revalidates
on a short interval.** Simplest thing that lets a non-engineer post a notice instantly to all live
users — a clean quick win with no risky surface.

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design).

**Access to request** (→ Task 6 block in `ACCESS_CHECKLIST.md`)

- [ ] **Retool access** (to build the admin surface).
- [ ] Franklin **design guidelines / design system** for the severity styles.
- [ ] (Banner config lives in the database I already have.)

**Facts to confirm** (information, not credentials)

- [ ] One global banner at a time, or multiple/targeted by segment? (Recommend starting with one global.)
- [ ] Is Retool the agreed admin surface here?
- [ ] Do banners need bilingual copy, or is the audience's language known?

## Next steps

- [ ] Sketch the `banners` table schema.
- [ ] Build the banner component in the local skeleton with mock config (all three severities).
- [ ] Confirm Retool as the admin surface + get design guidelines.
