# Repo Sync-In — reading the codebase & making an impact

Franklin's guide says it plainly: *"much of this exists partially — you're mainly connecting
pieces."* So the job on each task is **read what's there → find the gap → wire the smallest missing
piece → ship.** This is the template for doing that, task by task.

Use it with `ONBOARDING_PLAN.md` (the tasks), the `spikes/` (the design + open questions), and
`DAY_1_SCRIPT.md` (day-1 behaviour).

---

## The method (run once per task)

1. **Run it first.** Get the repo building + deploying on staging before reading a line. You can't
   reason about code you can't run.
2. **Trace one flow end-to-end** before touching anything — follow the actual code path.
3. **Fill the exists/missing table** (below). Filled rows = reuse. Empty rows = your actual work.
4. **Match their conventions** — module layout, naming, error handling, test style. Code that looks
   like the existing code earns trust fast.
5. **Ship the smallest connecting piece**, make it work → pretty → fast, then own the gaps you spot.

---

## Per-task template (copy this block for each task)

### Task: __________  (Plan #__ / Guide #__)

**Exists vs. missing**

| Piece it needs | Exists? | Where / notes |
|---|---|---|
| | ☐ | |
| | ☐ | |
| | ☐ | |

**Smallest change to ship:** _the one gap I can wire end-to-end first_

**Grounded questions** (each = "here's what I found, confirm my read"):
- …
- …

**Gaps I'll own** (missing test, stale doc, un-idempotent job, TODO): …

---

## What to look for, per task (starter checklist)

Grep/trace for these when you fill each table:

- **Nudge emails (#2/#1):** onboarding state on the user? a "stuck users" query? Postmark client
  already wired? a cron/scheduled-job pattern? a `lastNudgeAt`/suppression field?
- **Month-end template (#3/#2):** existing Postmark templates + how they're stored/versioned.
- **NIUM biometric (#4/#4):** an existing webhook handler pattern? signature-verification helper?
  where user↔NIUM ids are mapped?
- **LOA / Penneo (#5/#9):** any PDF generation already? a state-machine/status pattern? existing
  NIUM upload code?
- **Slack (#6/#6):** current Retool notification logic (to replace)? an event/outbox table? a Slack
  client already?
- **Banner (#7/#5):** existing top-of-app layout slot? a config table or feature-flag system? Retool admin?
- **PLUNK (#8/#7):** how user data would sync out? where marketing consent lives?
- **RFI (#8):** HubSpot integration already? a ticket/routing model? resolved-RFI history?

---

## Questions that land (the shape)

Grounded and confirm-style — never helpless, never reinventing:

- ✅ "I traced the onboarding flow — the user tracks completed steps but I don't see a `lastNudgeAt`.
  Tracked elsewhere, or the gap I should fill?"
- ✅ "Is Postmark already wired for any existing email? I'll reuse that client rather than add one."
- ✅ "Where do scheduled jobs live today — a cron pattern I should follow, or is this the first?"
- ❌ "How do nudge emails work?" (shows you didn't read the code)

---

## Making an impact (ops-engineer lane)

- **One shipped connecting piece in week one > a big half-built thing.** Pick the task where the most
  pieces already exist.
- **Own the gaps you find** — extreme ownership: found it, unclear who owns it → you own it.
- **Reliability is your impact:** idempotency, retries, monitoring, closing loops. The job that
  *never double-sends* beats the one that just sends.
- **Write down what you learned** — a short "how X works" note after tracing a flow helps the next
  person and proves you understand the system, not just your ticket.

> The move that does both at once: for your first task, bring your manager the filled-in
> **exists / missing / smallest-change** block above. It's your sync-in *and* your first impact.
