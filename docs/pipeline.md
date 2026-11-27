# Pipeline

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

Every GTM Brain workflow ran through the same steps: identity resolution, deterministic triggers and scoring, context assembly, then model synthesis and validation. n8n and Python ran them, with state in PostgreSQL. GTM is go-to-market.

## Identity resolution

Patients and therapists are people, and practices are organizations, so the business-to-business (B2B) "root domain" match fit only practices.
- **Canonical ids:** the HubSpot record id, linked to the in-house platform's user id.
- **Matching waterfall:**
  1. an exact match on the platform user id or the HubSpot id;
  2. an exact match on normalized email;
  3. an exact match on normalized phone (how Google Voice calls got attached);
  4. therapists: license or National Provider Identifier (NPI) number;
  5. practices: root domain (free email providers excluded) and NPI number, plus a therapist-to-practice mapping.
- **Quarantine:** a signal that didn't match deterministically went to a review queue, which I worked as revenue operations (RevOps). A model never guessed the match.
- **Data minimization:** before a model saw a record, fields the task didn't need were dropped.
- **Consent:** there were three kinds of consent: recording consent on calls, opt-outs for email and SMS, and consent to use survey answers.

## Matching and triggers

A model never decided who a patient was matched with, whether a patient was dropping off, or whether a therapist was available. Code computed those first and passed them to the model.

**Matching.** Patients filled in a detailed survey, and in it chose either an automatic match by the in-house algorithm or a matching call with Sales. The algorithm matched them to therapists by their needs and goals, using a mix of:
- therapy-style preferences;
- the issues they wanted help with;
- location: in person or teletherapy;
- therapist expertise.

The ranking also weighed each therapist's availability, and the Sales rep could override a match. After an automatic match, the patient booked their own intro calls. Either way, Sales had the patient until their first paid session, and got their drop-off alerts.

**Triggers:**
- **A patient was dropping off** when they:
  - told us they were stopping sessions;
  - asked for their profile to be deleted;
  - or stopped scheduling sessions and didn't respond to us.
- **A therapist was unavailable** when they:
  - had no availability on their calendar;
  - told us they were full;
  - or didn't reply within 48 hours when we asked for their availability.

  In each of these cases, that therapist wasn't matched to that patient, and their future matching didn't change. The step existed only to match each patient with an active, available therapist, with the least onboarding friction.
- **Friction:** matched patients with no opening that fit, or open high-priority Freshdesk tickets about billing.
- **Practice risk:** billing-service activity dropped, or the practice's contact changed.

## Time decay

Each signal *i* got the weight `W_i = A_i × e^(−λ·t_i)`:
- `t_i` was its age in days, and `A_i` was how much its source counted;
- λ = 0.03, with source weights from 1.0 to 2.0;
- for example, a survey submission 2.0, a Freshdesk ticket 1.5, a web visit 1.0 (therapists and practices only);
- signals under a cut-off weight were dropped before the prompt was built;
- the rest were ranked by weight: the top-weighted ones went into the prompt (see the token budget below), and real-time alerts were ranked by signal weight only.

## Context assembly and the token budget

Every prompt was capped at 16,000 input tokens, split by code:
- **System instructions and output schema:** about 1,500 tokens (fixed).
- **Canonical-truth slice:** about 3,000 tokens, filtered to the audience, state, segment and persona.
- **Structured-state snapshot:** about 1,500 tokens, a JSON summary from HubSpot, matching, scheduling and Freshdesk.
- **Rolling signals:** about 10,000 tokens, the top-weighted calls, threads and tickets. Signals were summarized after 30 days; the last 14 days went in verbatim.

## The input payload

This is the payload for a patient matching brief (WF-JIT-01a, on [workflows](workflows.md#wf-jit-01a-patient-matching-pre-call-brief)).

*Synthetic example, not company data.*
```json
{
  "workflow_id": "WF-JIT-01a",
  "patient": {
    "id": "pt_ref_A",
    "survey": {
      "issues": ["anxiety", "work stress"],
      "goals": ["manage stress at work"],
      "style_preferences": ["structured", "practical"],
      "location": "teletherapy",
      "times": ["evening"]
    }
  },
  "matches": [
    {"therapist_id": "th_ref_B", "rank": "<rank>", "matched_on": ["issue:anxiety", "style:structured", "location:teletherapy", "expertise:anxiety"], "earliest_opening": "<slot>"},
    {"therapist_id": "th_ref_C", "rank": "<rank>", "matched_on": ["issue:work stress", "location:teletherapy"], "earliest_opening": "<slot>"}
  ],
  "canonical_truth": {
    "objections": [{"id": "objection.patient.cost", "approved_response": "<approved wording>"}],
    "messages": [{"id": "message.patient.first-call", "text": "<approved wording>"}]
  },
  "signals": [
    {"source_id": "survey_A", "type": "survey", "content": "<free-text answer, minimized>"},
    {"source_id": "ticket_A", "type": "freshdesk", "content": "<insurance question>"}
  ]
}
```

## The output schema

Each output was validated in Python before delivery. `PatientMatchBrief` had these required fields:
- `what_they_told_us`, a list of items, each with a `citation_id`;
- `top_matches`, each with its therapist, `why`, `earliest_opening` and `citation_id`;
- `watch_outs`, each with a GTM Brain id;
- `opening_line`, with a GTM Brain id.

`TherapistPreCallBrief` and `PostCallSync` followed the same pattern.

If validation failed, or a statement lacked a `citation_id` that matched an input `source_id`, the orchestrator ran an automatic repair retry. If the retry failed too, the run stopped, no brief went out, and the failure was flagged to me.

## The system-prompt rules

The system prompt opened: "You prepare briefs for a mental-health marketplace's matching, recruiting and marketing teams." Its rules:
- **Only facts in the payload.** If something is missing, write "Unconfirmed".
- **Cite everything.** Every bullet carries the `source_id` it came from.
- **No clinical advice.** No diagnosis, no treatment advice, no outcome promises.
- **Commercial guardrails.** No prices, insurance coverage, discounts or therapist terms unless they appear in `canonical_truth`.
- **Minimum necessary.** Repeat only the patient details the brief needs. Never quote survey answers in outbound drafts.
- **Tone.** Warm and plain for anything a patient will read; short and direct for internal briefs.
- **Output.** Valid JSON for the given schema, nothing else.
