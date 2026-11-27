# Workflows

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

GTM Brain's workflows ran by timing: in real time, before a call, after a call, weekly and monthly. Briefs and alerts went to a Slack channel, mentioning the owner. Group practices were part of every workflow that fit them: the lead alert, the contact-change alert, the pre-call brief, the reviews, the post-call run, and the weekly and monthly runs. GTM is go-to-market, and CX is customer experience.

## The catalog

| Id | Timing | Team | What it produced | Delivered to |
|---|---|---|---|---|
| WF-RT-01 | Real time, within 5 minutes | Sales | High-intent lead alert: the speed-to-lead Slack alert | Slack channel, mentioning the owner |
| WF-RT-02 | Every hour | Patients: Sales until the first paid session is booked, then CX. Therapists: CX | Patient drop-off and therapist-availability alert | Slack channel, mentioning the owner |
| WF-RT-03 | Real time, under 15 minutes | Sales, CX | A therapist pausing or leaving, or a practice contact changing jobs | Slack channel, mentioning the owner; HubSpot flag |
| WF-JIT-01a | 2 hours before the call | Sales | Patient matching pre-call brief | Slack channel, mentioning the owner |
| WF-JIT-01b | 2 hours before the recruiting call | Sales | Therapist pre-call brief | Slack channel, mentioning the owner |
| WF-JIT-01c | 2 hours before the call | Sales (the group-practice rep) | Group-practice pre-call brief | Slack channel, mentioning the owner |
| WF-JIT-02 | 24 hours before the review or check-in | CX (therapist check-ins), Sales (group-practice reviews) | Group-practice review and therapist check-in brief | Google Docs, and a Slack channel mentioning the owner |
| WF-POST-01 | 15 minutes after the call | Sales | HubSpot fields filled and a follow-up draft | HubSpot API, Gmail draft |
| WF-POST-02 | 15 minutes after an escalation | CX | Escalation summary and product-feedback log | Jira, HubSpot note |
| WF-WK-01 | Monday 07:00 | Marketing | Voice-of-customer digest, copy changes, outbound list | Slack channel, Google Docs |
| WF-WK-02 | Monday 07:30 | Sales and revenue operations (RevOps, me) | Funnel bottleneck and stalled-lead audit | Slack channel |
| WF-MO-01 | First business day of the month | Marketing and RevOps (me) | Messaging, battlecard and ideal customer profile (ICP) drift check, opened as a pull request on GTM Brain | GitHub pull request, reviewed by the GTM Brain reviewers |

All sample outputs below are synthetic: made-up people, no dates, and no real wording.

## Real-time workflows

### WF-RT-01: high-intent lead alert

- **Trigger:** a HubSpot lead-score jump, or a new survey, pre-call form or practice inquiry. For therapists and practices only, repeat visits to their pages.
- **Inputs:** the HubSpot record, the lead source and routing group, Clay enrichment, the last call summary, and the matching battlecard or objection entries.
- **Human step:** the owner reached out. The lead-routing rules applied: a 30-minute re-notify and a 1-hour step-in.

*Synthetic example, not company data.*
```markdown
**HIGH-INTENT THERAPIST LEAD: Therapist B (routing group: on several platforms)**
* Trigger: filled in the pre-call form and opened the fee page twice today [Form] [Web]
* Why it matters: licensed in a state where matched patients are waiting for openings [Matching: openings]
* Likely objection: platform fees. Approved response: objection.therapist.fees [GTM Brain]
* Next step: call today. [Open HubSpot record] [Open email draft]
```

### WF-RT-02: patient drop-off and therapist-availability alert

- **Trigger:** the drop-off and unavailable-therapist rules on [pipeline](pipeline.md#matching-and-triggers), checked every hour.
- **Inputs:** the in-house platform's matches, bookings and availability, Freshdesk tickets, billing and insurance status from Stripe, and the pricing and insurance guardrails.
- **Human step:** the owner decided and reached out. The model only drafted.
  - A patient's alert went to the Sales rep who had them until their first paid session was booked, and to CX after that, whichever way they were matched.
  - A patient who asked for their profile to be deleted got no win-back draft. Profile deletion was the only case with no win-back message.
  - A therapist alert went to CX, which kept therapist availability accurate. In each unavailable-therapist case, the therapist wasn't matched to that patient, and their future matching didn't change.

*Synthetic example, not company data.*
```markdown
**DROP-OFF ALERT: Client A (stopped scheduling)**
* Signal: no session booked since their last one was cancelled; two check-in messages unanswered [Platform] [HubSpot]
* Context: their therapist has no evening openings left [Platform: calendar]
* Playbook: offer to rebook with the same therapist, or with the next match that has evening openings [Matching]
* Actions: [Open draft] [Mark Useful / Inaccurate]
```

### WF-RT-03: a therapist pauses or a practice contact changes

- **Trigger:** a therapist set their availability to paused or left; or Sales Navigator showed a practice contact changing jobs.
- **Output:** for therapists, which matched patients needed a new match; for practices, who the new contact was and the approved re-introduction message.

## Pre-call workflows

### WF-JIT-01a: patient matching pre-call brief

- **For:** patients who chose a matching call with Sales in the survey.
- **Trigger:** a matching call was booked in the in-house platform, whose webhook posted to Slack. The brief arrived 2 hours before the call.
- **Inputs:**
  - the patient's survey submission;
  - the in-house algorithm's top therapist recommendations;
  - the earliest available appointment for each recommended therapist;
  - objection and messaging entries for patients;
  - any open Freshdesk tickets.
- **Human step:** the Sales rep checked fit, talked it through with the patient, could override the match, and booked an intro call with each matched therapist. Sales stayed with the patient until they booked their first paid session, then handed them to CX.

*Synthetic example, not company data.*
```markdown
**PRE-CALL BRIEF: PATIENT MATCH, Client A (call later today)**
* What they told us [Survey]: wants help with anxiety and work stress; goal: manage stress at work; prefers a
  structured, practical style; teletherapy, evenings
* Top matches [Matching algorithm]:
  * Therapist B: expertise in anxiety; structured, practical style; teletherapy. Earliest: Tuesday evening
  * Therapist C: expertise in work stress; teletherapy, evenings only. Earliest: Thursday evening
  * Therapist D: expertise in anxiety; in person or teletherapy. Earliest: Wednesday morning
* Watch-out: asked about cost in the survey. Approved wording: objection.patient.cost [GTM Brain]
* Opening: message.patient.first-call [GTM Brain]
```

### WF-JIT-01b: therapist pre-call brief

- **For:** therapists in the recruiting process. It reached the Sales rep 2 hours before the recruiting call.
- **Inputs:**
  - the pre-call form the therapist was asked to fill in (license type and states, specialties, modalities, insurance panels, desired caseload, availability, current platforms);
  - everything else collected along the way: the lead source, Clay enrichment, the lead score, email engagement, notes from earlier calls, Sales Navigator activity;
  - the therapist ICP, objections and proof points from GTM Brain.
- **Human step:** the Sales rep ran the recruiting call. After it, Sales brought in Operations for the administrative parts of onboarding, and the therapist moved to CX once onboarding was finished.

*Synthetic example, not company data.*
```markdown
**PRE-CALL BRIEF: THERAPIST, Therapist B**
* Pre-call form [Form]: licensed in two states; CBT and EMDR; in network with two insurers; wants a part-time caseload
* What we know [HubSpot] [Clay]: came from the website; on several other platforms; opened the last two emails
* Fit [GTM Brain: icp.therapist]: strong. Patients in their states are waiting for openings [Matching: openings]
* Likely objections: fees and client volume. Approved responses: objection.therapist.fees, objection.therapist.volume
* Proof to use: case.therapist.caseload-growth [GTM Brain]
* Ask: how many new clients a week they can take, and their earliest start
```

### WF-JIT-01c: group-practice pre-call brief

- **For:** group practices. It reached the group-practice rep in Sales 2 hours before the call.
- **Inputs:**
  - the practice's inquiry;
  - Clay enrichment and the lead score;
  - notes from earlier calls;
  - the practice ICP, objections and proof points from GTM Brain.

### WF-JIT-02: group-practice review and therapist check-in brief

It prepared for:
- **group-practice reviews:** for example billing-service activity, open Freshdesk tickets, and the approved messaging for practices;
- **therapist check-ins:** for example caseload against stated capacity, open slots, and patient feedback.

CX ran the therapist check-ins, and the group-practice rep in Sales ran the group-practice reviews. The packets went to Google Docs and Slack.

## Post-call workflows

### WF-POST-01: HubSpot fill and follow-up draft

- **Trigger:** a Google Voice call ended, and its transcript was ready.
- **Steps:**
  1. It extracted the fields for that call type:
     - patient: preferences, chosen therapist, booked slot, open questions;
     - therapist: onboarding stage, caseload, start date;
     - practice: needs, decision process, next step.
  2. It wrote them to HubSpot, each tagged `[AI-Verified | Source ID]`, so each value showed where it came from.
  3. It drafted a Gmail follow-up that summarized the next steps and linked approved material. The draft was never sent automatically.

*Synthetic example, not company data.*
```markdown
**POST-CALL SYNC: Therapist B**
* HubSpot updated: onboarding stage -> paperwork sent; desired caseload -> part-time [Call]
* Follow-up drafted: next steps, the onboarding checklist link, the approved fee summary [GTM Brain: pricing.therapist]
* [Open draft] [Undo HubSpot changes]
```

### WF-POST-02: escalation summary

A Freshdesk escalation was summarized into Jira, for engineering, and into a HubSpot note. Repeated product issues fed the weekly digest.

## Weekly workflows

### WF-WK-01: voice-of-customer digest, copy changes and outbound list

- **Inputs:**
  - the week's Google Voice call text and Freshdesk tickets;
  - the reasons patients didn't book and therapists didn't join;
  - the current patient, therapist and practice page copy;
  - Sales Navigator and HubSpot high-intent leads with no open deal.
- **Output:**
  - the top emerging objections, with quotes stripped of identifiers;
  - proposed copy changes for each audience's pages;
  - battlecard and objection updates, proposed to GTM Brain for review;
  - an outbound list of therapists and practices.

*Synthetic example, not company data.*
```markdown
**WEEKLY VOICE-OF-CUSTOMER DIGEST: Marketing**
* Emerging objection, patients [Calls] [Freshdesk]: whether their insurance is accepted, raised before booking. Quote, identifiers stripped: "<quote>"
* Emerging objection, therapists [Calls] [HubSpot]: how the fees compare with other platforms. Quote, identifiers stripped: "<quote>"
* Proposed copy change, patient pages: answer the insurance question near the top, using objection.patient.insurance [GTM Brain]
* Proposed for review: an update to objection.therapist.fees, sent to its reviewers as a pull request [GTM Brain]
* Outbound list [Sales Navigator] [HubSpot]: therapists and practices with high intent and no open deal
* [Open in Google Docs] [Mark Useful / Inaccurate]
```

### WF-WK-02: funnel bottleneck audit

I ran it with Sales. It showed where leads stalled, in each funnel:
- **Patients:** survey, then the match (by the algorithm, or on a matching call with Sales), then intro calls (booked by Sales, or by the patient after an automatic match), then first paid session booked (the hand-over to CX), then ongoing sessions.
- **Therapists:** lead, then pre-call form, then recruiting call, then onboarding (with Operations), then first client. The hand-over to CX came once onboarding was finished.
- **Practices:** inquiry, then call, then proposal, then signed.

It was the automated side of the review of the user journey that we ran whenever conversion or lead response rates dropped.

## Monthly workflow

### WF-MO-01: messaging, battlecard and ICP drift check

I ran it with Marketing.
- **What it did:**
  - compared GTM Brain's claims, objections and ICP with what actually happened: who booked and who didn't, who joined and who didn't;
  - where they differed, it drafted a pull request on the GitHub repository;
  - the GTM Brain reviewers approved or rejected it.
- **Why it mattered:** it kept canonical truth current without anyone rewriting it by hand.

## Who got which alert

- **Patients:** Sales had every patient, matched either way, until they booked their first paid session, and got their drop-off alerts. Then Sales handed them to CX, which got the later alerts.
- **Therapists:** Sales recruited them and got their pre-call briefs. Once onboarding was finished, they moved to CX, which got their availability alerts.
- **Group practices:** the group-practice rep in Sales got their pre-call briefs and ran their reviews.
- **The cap:** real-time alerts had a cap of at most 3 alerts per person per day, ranked by signal weight only.
