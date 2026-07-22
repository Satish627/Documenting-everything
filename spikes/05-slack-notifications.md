# Spike 05 — Slack notifications (migrate off Retool)

**Task:** Plan #6 / Guide #6 (**critical alerts**) · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*
**Reused by:** [Spike 03](03-nium-biometric-notifications.md) and [Spike 04](04-penneo-power-of-attorney.md) — this is the one notification path they both call.

> Goal from the guide: *when something critical happens (failed payout, risk alert), a Slack
> notification arrives within 10 seconds.* Retool is slow and flaky for this; we want direct + reliable.

---

## Problem

Critical operational events need to reach the team **fast and reliably**. Retool as the middleman is
too slow/flaky for alerts where seconds matter. We want to detect events in our own system and push
them straight to Slack, with retries so a Slack hiccup never means a silently-dropped alert. This is
also the shared notifier the NIUM and Penneo spikes depend on — build it once, well.

## Questions to answer

1. What are the critical event types?
2. How do we detect events — DB triggers or an application-level emitter?
3. How do we deliver to Slack reliably (format, retries, <10s)?
4. How do we keep staging noise out of the prod channel?

## Findings

### 1. Event catalogue (confirm/extend with ops)

Starting set: **failed payout**, **risk/fraud alert**, **biometric passed** (from Spike 03),
**LOA signed / LOA failed** (from Spike 04), **onboarding completed**. Each event = a type + a small
payload (who, what, amount if relevant, link).

### 2. Detection — recommend an application-level event emitter (+ outbox), not raw DB triggers

| Approach | Notes |
|---|---|
| **App-level emitter + outbox table** ✅ | Testable, typed, retryable; events written in the same transaction as the state change, delivered by a worker |
| Raw Postgres triggers → webhook | Fires close to the data, but hard to test/version, and DB-side HTTP is fragile |

**Recommend:** emit a typed event in code at the point the state changes, persist it to an
**outbox** table, and have a worker deliver it to Slack. The outbox is what guarantees *at-least-once*
delivery and makes retries clean.

### 3. Delivery to Slack

- Use **Slack Incoming Webhooks** (or the Web API if we need richer control) with **Block Kit**
  message templates per event type — consistent, scannable alerts.
- **Retry with backoff** on failure; the outbox row stays "unsent" until Slack returns success, so a
  Slack outage just delays, never drops.
- Webhook URL / token in env only (non-negotiable #2).
- Well within 10s: emit → worker picks up → post. (Vercel Cron every minute is too slow for "10s" —
  use a short-interval worker / queue, or post inline from the emitter with the outbox as backstop.)

### 4. Staging vs prod channels

- Separate `SLACK_WEBHOOK_URL` per environment → `#alerts` (prod) vs `#alerts-staging` (preview), so
  test events never page the team (ties to Spike 01's per-environment vars).

## Recommendation

**Typed event emitter → outbox table → worker → Slack (Block Kit), with backoff retries and
per-environment channels.** This hits the 10s target, guarantees delivery, and gives NIUM/Penneo a
single `notifySlack(event)` to call instead of three bespoke integrations.

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design).

**Access to request** (→ Plan #6 block in `ACCESS_CHECKLIST.md`)

- [ ] `SLACK_WEBHOOK_URL` — **staging** channel (`#alerts-staging`) and **prod** channel (`#alerts`).
- [ ] Access to the **event source** / current Retool logic, so I can see what's monitored today.

**Facts to confirm** (information, not credentials)

- [ ] The authoritative list of "critical" events and their payloads.
- [ ] Where events originate today, and what the event **source of truth** is.
- [ ] Do we need a queue/worker, or is short-interval polling of the outbox acceptable for "10s"?
- [ ] Which Slack channels, and who owns them.

## Next steps

- [ ] Confirm the event catalogue with ops (what Retool sends today).
- [ ] Prototype the emitter + outbox + Block Kit formatting on **mock events** in the local skeleton.
- [ ] Define the shared `notifySlack(event)` contract used by Spikes 03/04.
- [ ] Decide worker vs. polling for the 10s SLA.
