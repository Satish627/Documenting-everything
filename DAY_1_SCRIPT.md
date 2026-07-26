# Day 1 Script

One page to keep open. Goal: **prepared + calm + ship something small by Friday.** At Franklin,
that beats eager-with-lots-of-questions. Lead with a proposal, ask few sharp questions, touch
nothing risky.

---

## Opening move (do this first — it *is* the impression)

> "Before starting I read the playbook and the 9 tasks, and drafted spike docs + a phased access
> plan for each. Could I get 10 minutes to walk you through how I'm sequencing it — mostly so you
> can tell me where I've got it wrong?"

Then show `ONBOARDING_PLAN.md` + `spikes/`. Hand over the **Foundation block** of `ACCESS_CHECKLIST.md`.

---

## The 6 questions worth asking on Day 1

Each phrased as *"here's my assumption — correct me,"* not open-ended.

1. **What already exists?** "The guide says I'm mainly connecting pieces — can someone show me the
   repo, DB schema, and any half-done integrations before I design anything?" *(highest-ownership question)*
2. **Priorities.** "I'd do Vercel staging/prod first, then nudge emails (High Priority). Matches your
   urgency, or is something more on fire?"
3. **Foundation access.** Repo, Vercel team, staging DB, domains — plus the fact: greenfield or does a
   project/repo already exist?
4. **Stack.** "Assuming Next.js + TS on Vercel + Postgres — right? What conventions do I match (deploys,
   code review, CI)?"
5. **Guardrails.** "What can I never touch as the new person — prod data, KYC state, anything moving
   funds — and who signs off when a task needs it?"
6. **Manager & comms.** "Who's my day-to-day manager, how do you like updates, and when do I walk over
   vs. message?"

---

## Say this out loud (shows judgment)

> "I have detailed per-task questions — the exact NIUM event, Postmark-vs-PLUNK ownership, the LOA
> template fields — but I'll bring those when I pick up each task, not front-load them now."

---

## Avoid (impression-killers)

- Touching prod, or pushing code, before I know the conventions. **Read first.**
- Asking what the playbook already answers (office codes, lunch, vacation).
- Working in DMs — **default to open channels.**
- Overpromising a timeline before I've seen what exists.

---

## First-week arc

- [ ] Get Foundation access; confirm stack + conventions.
- [ ] See what already exists for each task (avoid reinventing) — use `REPO_SYNC_IN.md`.
- [ ] Ship the **Vercel staging/prod foundation** — small, visible, unblocks everyone.
- [ ] Friday: short note in the team channel — what I set up, what I confirmed, what I'm picking up next.

> Rule of thumb the whole week: **make it work → make it pretty → make it fast.** Results over effort.
