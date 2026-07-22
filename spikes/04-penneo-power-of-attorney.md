# Spike 04 — Penneo power-of-attorney automation

**Task:** Plan #5 / Guide #9 (**critical, most complex**) · **Status:** Draft · **Mode:** design + research (no access yet)
**Assumed stack:** Next.js + TypeScript on Vercel, PostgreSQL — *confirm on day 1.*
**Consumes:** the `prepare_loa` hand-off from [Spike 03](03-nium-biometric-notifications.md).

> Goal from the guide: *user triggers "sign power of attorney" → gets an email link → signs in
> Penneo → it's automatically on NIUM ~30s after.* This shaves weeks off card activation. It's also
> the highest-stakes task here: a **legal document** (power of attorney) populated with customer
> data and filed with a partner bank.

---

## Problem

Getting a signed power of attorney to NIUM is a manual, multi-week friction point. We want to
automate the chain end-to-end. But every stage can fail independently, and the artifact is a
**legally binding document** — so the two dominant concerns are **(a) document accuracy** (wrong
company name / CVR / signatory on a legal doc is unacceptable) and **(b) durable, resumable state**
(never lose a doc mid-flow, never double-file).

## The chain (6 stages)

```
prepare_loa ─▶ 1. gather data ─▶ 2. generate PDF ─▶ 3. send to Penneo ─▶
              4. user signs (Penneo webhook) ─▶ 5. download signed PDF ─▶ 6. upload to NIUM ─▶ done
```

## Questions to answer

1. How do we guarantee the **document is correct** before anyone signs?
2. How do we **generate the PDF** on our stack?
3. How does the **Penneo integration** work (upload, signing link, signed webhook)?
4. How much auto-fires vs. stays human-gated — especially the **final NIUM upload**?
5. How do we track state so nothing is lost or double-filed?

## Findings

### 1. Document accuracy (the biggest risk)

- Pull user + company fields from the DB (name, CVR, address, signatory/role). **Validate required
  fields are present and well-formed before generating** — no blank/placeholder values on a legal doc.
- Recommend a **verification gate**: render a preview and require a human (ops, or the customer at
  trigger time) to confirm the details are right before it's sent for signature. This is the
  "draft, don't blindly fire" principle (non-negotiable #1) applied to a legal artifact.
- Never invent or guess a field. If data is missing → stop, flag, don't generate.

### 2. PDF generation (decision)

Vercel serverless makes a full headless-Chrome (Puppeteer) render heavy/fragile. Options:

| Approach | Notes |
|---|---|
| **`@react-pdf/renderer`** ✅ | Pure JS, no browser binary — fits serverless; template as components |
| HTML → PDF via headless Chromium | Familiar HTML/CSS, but heavy cold starts on Vercel; needs `@sparticuz/chromium` or similar |
| Dedicated PDF API service | Offloads render, but adds a vendor + sends customer data to a third party (privacy review) |

**Lean `@react-pdf/renderer`** for a legal template on this stack — deterministic output, no browser
dependency, no extra vendor. Confirm once we see the real template + stack.

### 3. Penneo integration (confirm against Penneo API docs on day 1)

- Create a **casefile / signing request**, **upload the unsigned PDF**, add the **signer**
  (the customer), and get a **signing link** to email them.
- Penneo sends a **webhook when the document is signed** → we **download the signed PDF**.
- Apply the same webhook discipline as Spike 03: **verify Penneo's signature**, **respond fast +
  process async**, **idempotent** on Penneo's event/case id, secrets in env only.

### 4. What auto-fires vs. human-gated

| Stage | Day-one stance |
|---|---|
| Gather data | Automate |
| Generate PDF | Automate, **but** behind a field-validation gate |
| Preview / verify details | **Human-gated** (legal accuracy) |
| Send to Penneo + email link | Automate after verify |
| User signs | The customer's own action (the real signature) |
| Download signed PDF | Automate |
| **Upload to NIUM** | Automate **in sandbox**; monitor + idempotent. Treat prod filing as sensitive — confirm with ops before enabling live auto-upload |

The customer signing *is* the human authorization for the legal act. The caution is on **accuracy
before signing** and **not double-filing** to NIUM after.

### 5. State tracking (durable + resumable)

Model an explicit **state machine** per LOA request, persisted:

```
prepare_loa → data_gathered → pdf_generated → pending_verification →
sent_for_signature → signed → downloaded → uploaded_to_nium → complete
                    └─▶ failed_<stage> (with reason, retryable)
```

- Persist current state + Penneo case id + NIUM ref + timestamps; surface it in the user's dashboard.
- Every transition idempotent; every external call retryable. A crash mid-flow resumes from the last
  persisted state — a signed document must **never** be lost before it reaches NIUM.

## Recommendation

Build the LOA flow as a **persisted state machine**: gather → validate → generate (`@react-pdf`) →
**human-verify** → Penneo signing → signed webhook (signed, idempotent) → download → NIUM upload.
Automate the mechanical stages; gate document accuracy with a verification step; keep the live NIUM
filing behind an explicit ops go-ahead until it's proven in sandbox. Reuse the NIUM client from
Spike 03 and the Slack notifier for status/failures.

## Open questions (align in week 1)

Two buckets: things I need **granted** (credentials/access) vs. things I need **told** (facts that
shape the design).

**Access to request** (→ Plan #5 block in `ACCESS_CHECKLIST.md`)

- [ ] `PENNEO_API_BASE_URL` + `PENNEO_API_KEY` — **sandbox** first.
- [ ] Penneo **webhook secret** (to verify the "signed" webhook).
- [ ] The **LOA template document** itself, from whoever owns the legal doc.
- [ ] The **NIUM upload endpoint** details (NIUM credentials already obtained in Plan #4).

**Facts to confirm** (information, not credentials)

- [ ] Who owns the LOA template, and what are its exact **required fields**?
- [ ] Penneo API specifics: casefile model, how the signing link is delivered, signature scheme.
- [ ] What format/metadata does the NIUM endpoint expect for the signed doc?
- [ ] Is the **customer or ops** the one who verifies details before signing?
- [ ] Is ops comfortable auto-filing to **live** NIUM once sandbox is proven, or stay human-triggered?
- [ ] Retention/GDPR: how long do we store the signed PDF, and where?

## Next steps

- [ ] Read the Penneo API docs; pin down casefile creation, signing link, and signed-webhook contract.
- [ ] Confirm the LOA template + required fields with whoever owns the legal doc.
- [ ] Prototype PDF generation with `@react-pdf` on **mock company data** in the local skeleton.
- [ ] Sketch the state-machine table schema (states, Penneo/NIUM refs, timestamps).
- [ ] Agree the `prepare_loa` hand-off contract with Spike 03.
