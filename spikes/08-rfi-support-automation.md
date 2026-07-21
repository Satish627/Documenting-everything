# Spike 08 — RFI: automate support responses

**Task:** Plan #9 / Guide #8 (**most ambiguous, AI-heavy**) · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL, HubSpot — *confirm on day 1.*
**Primary reference:** `documents/operational-engineer-case-ai-support-engine.md.pdf` — the AI Support
Draft Engine case maps almost directly onto this task; reuse its thinking.

> Goal from the guide: *an RFI comes in → the system categorizes it, notifies the right person, and a
> templated response can be sent.* RFIs = requests for information (regulatory, partner checks).

---

## Problem

RFIs arrive by email/portal and are handled manually. We want auto-categorization, routing to the
right owner, and templated responses — **without** a machine sending anything sensitive on its own.
This is regulated-fintech correspondence, so the guardrails matter more than the automation.

## Questions to answer

1. What's the pipeline?
2. How do RFIs get categorized and routed?
3. When does the system draft vs. stay silent and escalate?
4. How does this fit HubSpot?

## Findings

### 1. Pipeline (from the interview case)

```
ingest → classify → route → draft (grounded) → HUMAN REVIEW → send → capture the edit
```

Same shape the case scores on. **No auto-send** — a human always reviews before anything leaves
(non-negotiable #1). The captured edit is the signal that improves future drafts.

### 2. Categorization + routing

- Classify RFI by **type** (regulatory, partner KYC check, transaction query, other) and **owner**.
- Routing rules map type → responsible person; notify via the Slack path (Spike 05).
- Track completion status per RFI.

### 3. When to draft vs. escalate (the important part)

- **Draft a confident reply** only for well-understood, low-risk RFI types with a templated answer.
- **Stay silent + escalate** for compliance-sensitive or ambiguous RFIs — route to a human with no
  draft rather than a confident-but-wrong one. "Knowing when *not* to draft" is the whole game here.
- Drafts are **grounded** in approved templates / past resolved RFIs, never free-generated, and
  never contain financial/legal advice or unverifiable account facts (non-negotiable #5).

### 4. HubSpot fit

- RFIs live as **HubSpot tickets**. The system reads the incoming ticket, attaches the draft/route
  for the agent there, and writes status back — it slots into the existing workflow, not a greenfield tool.

### Language

- Danish primary, English when the sender writes English; Franklin voice (per `CLAUDE.md`).

## Recommendation

Build the RFI flow as the case's **ingest → classify → route → grounded draft → human review → send**
pipeline, inside **HubSpot tickets**, with an explicit **"draft vs. escalate-only" decision** and no
auto-send. Start with categorization + routing + a small grounded template set for the safe RFI types;
leave compliance-sensitive types as escalate-only. This is the last and most ambiguous task — expect
to spike it further once the real RFI history is available.

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design).

**Access to request** (→ Task 8 block in `ACCESS_CHECKLIST.md`)

- [ ] **HubSpot access** + `HUBSPOT_TOKEN` — **sandbox** first.
- [ ] Access to the **RFI intake channel(s)** (email inbox / partner portal).
- [ ] Access to the **resolved-RFI history**, if one exists, to ground drafts in.

**Facts to confirm** (information, not credentials)

- [ ] Where do RFIs actually arrive (email, partner portal, both), and do they already hit HubSpot?
- [ ] What RFI types exist, and which are **safe to template** vs. **always-escalate**?
- [ ] Is there a usable history of resolved RFIs at all?
- [ ] What's the routing map (type → owner)?
- [ ] Which model/tooling, and the cost/latency budget per RFI (operational concern from the case)?

## Next steps

- [ ] Re-read the interview case and lift its architecture/guardrails directly.
- [ ] Draft the RFI type taxonomy + routing map + the "draft vs. escalate" rules.
- [ ] Prototype classification on **mock RFIs** (reuse the case's sample inquiries) — no sending.
- [ ] Confirm the HubSpot ticket workflow + RFI intake channels.
