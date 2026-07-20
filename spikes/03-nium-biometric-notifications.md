# Spike 03 — NIUM biometric notifications

**Task:** Plan #4 / Guide #4 (**critical**) · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*

> Goal from the guide: *"Complete biometric in a test account → notification pops up within 30
> seconds,"* which then triggers the next workflow (card prep). This is the **trigger point** that
> also unblocks the Penneo power-of-attorney automation (Spike 05 / Guide #9).

---

## Problem

When a user finishes NIUM biometric verification, we need to know **immediately** so we can start
the next step of their setup (card prep, and the LOA signing flow). Today there's no signal. We
want a NIUM webhook → a fast, reliable internal notification → a downstream trigger. Two things make
this delicate: it's a **KYC milestone** (compliance-sensitive), and the downstream actions can touch
cards, so the automated part must stop short of anything irreversible without a human.

## Questions to answer

1. Which NIUM event means "biometric done", and how do we receive it?
2. Where should the notification land — Slack, email, or dashboard?
3. How do we make the webhook reliable and secure (auth, idempotency, retries, <30s)?
4. How much of the downstream workflow do we automate vs. leave to a human?

## Findings

### 1. Receiving the event (confirm against NIUM webhook docs on day 1)

- NIUM sends **webhooks** for verification/KYC lifecycle events. We need the exact event
  type/status that represents *biometric completed / verification passed* — **this is the key
  unknown to confirm in the docs.** (There may be several statuses: initiated, pending, passed,
  failed, review — we only act on the "passed/completed" one.)
- Expose a webhook endpoint (e.g. `POST /api/webhooks/nium`) as a Next.js route handler.
- Map the incoming payload to our user via NIUM's customer/reference id → our `users` table.

### 2. Notification channel — recommendation: **Slack** (+ dashboard status)

| Option | Pros | Cons |
|---|---|---|
| **Slack** ✅ | Instant, visible to the ops team, matches the "critical alerts to Slack" direction (Guide #6) | Needs a channel + webhook |
| Email | Simple | Slower, easy to miss, clutter |
| Dashboard only | Persistent record | Nobody's watching it in real time |

**Recommend Slack for the human-facing alert**, *and* write the status to the user's record so the
dashboard reflects it too. This reuses the Slack integration we're building in Guide #6 — one
notification path, not three. (Confirm the channel with the team.)

### 3. Reliability + security (the part that actually matters)

A webhook handler for a KYC event must be defensive:

- **Verify authenticity.** Validate NIUM's webhook signature / shared secret before trusting any
  payload. Reject unsigned/invalid requests. Secret in env only (non-negotiable #2).
- **Respond fast, process async.** Acknowledge with `200` immediately, then do the work
  (lookup, notify, trigger) in the background — this keeps us well under the 30s target and stops
  NIUM from retrying just because we were slow.
- **Idempotency.** Store the NIUM event id; if we've seen it, no-op. Webhooks get delivered more
  than once — we must never double-notify or double-trigger.
- **Handle out-of-order / duplicate statuses.** Only advance state forward (e.g. ignore a
  "pending" that arrives after a "passed").
- **Retry/failure path.** If our downstream step fails, the notification still fires and the event
  is queued for retry — we never silently drop a passed-KYC event.
- **Sandbox only.** Point the whole thing at NIUM's sandbox; the endpoint must refuse if
  `APP_ENV=staging` but the payload looks like production (ties to Spike 01's fail-fast check).

### 4. How much to automate downstream (human-in-the-loop)

The notification and status update are safe to automate. The **card/settlement actions are not**,
on day one:

- **Automate now:** verify → notify Slack → update user status → **create a task/draft** for the
  next step (e.g. "ready for LOA / card prep").
- **Keep human-gated:** actually issuing a card or moving funds. Draft, don't fire (non-negotiable #1).
- This makes the biometric event the clean hand-off into **Spike 05 (Penneo LOA)** without wiring
  an irreversible chain end-to-end before anyone's confirmed it's safe.

### Handler shape (pseudocode)

```ts
// POST /api/webhooks/nium
export async function POST(req) {
  const raw = await req.text();
  if (!verifyNiumSignature(raw, req.headers)) return json(401);   // authenticity
  const event = JSON.parse(raw);

  ack(200);                                                        // respond fast

  if (await alreadyProcessed(event.id)) return;                   // idempotency
  await markProcessed(event.id);

  if (event.type === NIUM_BIOMETRIC_PASSED) {                     // confirm exact type
    const user = await findUserByNiumRef(event.customerRef);
    await setOnboardingStep(user.id, "biometric_passed");         // dashboard status
    await notifySlack(`✅ Biometric passed: ${user.company}`);     // human alert
    await enqueueNextStep(user.id, "prepare_loa");                // draft, not auto-fire
  }
}
```

## Recommendation

Build a **signed, idempotent, async NIUM webhook handler** that on the *biometric-passed* event
updates the user's status, alerts **Slack**, and **enqueues** (not auto-executes) the next step.
Reuse the Slack path from Guide #6. Keep every irreversible card/fund action human-gated for now.

## Open questions (align in week 1)

- Exactly which NIUM event type/status = biometric passed? (And what are the failure/review statuses?)
- How does NIUM sign webhooks (HMAC secret? header?) so we can verify them?
- Which reference id links a NIUM event to our user record?
- Which Slack channel, and is the Guide #6 Slack integration ready to reuse?
- What is the *intended* full downstream chain (card creation, settlement setup) — and which parts
  is ops comfortable automating vs. keeping manual?

## Next steps

- [ ] Read the NIUM webhook docs; pin down the exact "passed" event + signature scheme.
- [ ] Prototype the handler against a **mock NIUM payload** in the local skeleton (signature verify,
      idempotency, Slack format) — no real NIUM calls.
- [ ] Coordinate with Spike 05 (Penneo) on the `prepare_loa` hand-off contract.
- [ ] Confirm the Slack channel + reuse of the Guide #6 integration.
