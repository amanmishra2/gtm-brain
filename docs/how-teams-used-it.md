# How each team used it

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

Marketing, Sales and customer experience (CX) each worked from GTM Brain, the shared go-to-market (GTM) context layer. It reached them as pushed briefs, alerts and drafts, and they could also ask Claude for anything else.

## Marketing

Marketing ran patient, therapist, and private practice acquisition, and reviewed everything in GTM Brain.
- Every Monday morning, the voice-of-customer digest brought the top emerging objections, proposed copy changes for each audience's pages, battlecard and objection updates proposed to GTM Brain, and an outbound list of therapists and practices.
- Every month, Marketing and I ran the drift check, which compared GTM Brain's claims, objections and ideal customer profile (ICP) with who booked and who joined, and opened a pull request where they differed.

## Sales

Sales reviewed the claims, proof points and objections, and worked all three audiences.
- **Patients:**
  - ran the matching calls for patients who chose a matching call in the survey, with a pre-call brief of the survey answers, the top matches and the approved wording;
  - could override a match, and booked an intro call with each matched therapist;
  - stayed with every patient, matched either way, until they booked their first paid session, and got their drop-off alerts until then.
- **Therapists:**
  - got high-intent lead alerts and a pre-call brief before each recruiting call;
  - brought in Operations for the administrative parts of onboarding.
- **Group practices:** there was one dedicated sales rep for group practices. The group-practice rep got the group-practice pre-call brief, and ran the group-practice reviews with a review brief.
- **After each call:** the HubSpot fields were filled from the call, and a follow-up draft waited in Gmail for the rep to edit and send.
- **Every Monday:** Sales and I ran the funnel bottleneck audit.

## CX

CX took over each patient once they had booked their first paid session, and each therapist once they had finished onboarding. Its main goal was to keep clients scheduling sessions, and it reviewed everything in GTM Brain.
- It got the later drop-off alerts for patients, and the availability alerts for therapists.
- It ran the therapist check-ins, with a check-in brief.
- With Sales, it got the alerts when a therapist paused or left, listing the matched patients who needed a new match, or when a practice contact changed jobs.
- Its escalations were summarized into Jira and a HubSpot note.
- Its playbooks were updated every two weeks.

## Operations

Operations helped Sales with the administrative parts of onboarding therapists.

## My revenue operations work

I was in revenue operations (RevOps), and I built GTM Brain. I also:
- worked the queue of records that didn't match automatically;
- ran the weekly funnel audit with Sales;
- ran the monthly drift check with Marketing;
- got flagged when an output failed its check after the repair retry.

## Asking Claude

Everyone had Claude connected to GitHub, with custom skills, so a question was answered straight from the GTM Brain repository. Reviewers who weren't technical also worked with the repository through Claude, which was already set up for everyone.

## Adoption

Adoption was voluntary. People wanted an easier way to get the information, so everyone was on board and gave feedback.

The workflows behind each of these are on [workflows](workflows.md).
