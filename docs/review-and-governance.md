# Review and governance

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

Every part of GTM Brain, the shared go-to-market (GTM) context layer, had human reviewers, and a person stayed in the loop by design.

## Who reviewed what

GTM Brain was reviewed by four teams (Marketing, Sales, CX, Leadership). CX is customer experience, and ICP is the ideal customer profile.

| Team | What it reviewed |
|---|---|
| Marketing | Everything |
| Sales | Claims, proof points and objections |
| CX | Everything |
| Leadership | The ICP and pain points |

Leadership's review was a sign-off.

## How review ran

- Review ran through pull requests with named reviewers, plus review meetings.
- An entry was approved only after review.
- Reviewers who weren't technical worked with the repository through Claude, which was already set up for everyone.

## An entry's life

1. **Drafted.** A change to an entry came from a person, from the weekly voice-of-customer digest (battlecard and objection updates), or from the monthly drift check.
2. **Proposed.** It went up as a pull request on the GitHub repository, with named reviewers.
3. **Reviewed.** The reviewers for that part read it in the pull request or at a review meeting.
4. **Approved.** Only then was the entry approved. Each claim entry recorded its status and who approved it (`approved_by`).
5. **Fixed if needed.** If an issue turned up later, the change could be rolled back, fixed and documented.

## Automated checks

- Scripts at each step checked that nothing was failing, and flagged any issue. Frequent human reviews ran alongside them.
- Every output was checked in Python against its schema, and every statement had to cite a source the model was given. A failed output got an automatic repair retry. If the retry failed too, the run stopped, no brief went out, and the failure was flagged to me (see [pipeline](pipeline.md#the-output-schema)).

## Disagreements

When Sales and Marketing disagreed on a claim or an objection response, the data decided, and experiments then confirmed it.

## Delivery channels

- **Slack:** alerts and pre-call briefs went to a Slack channel and mentioned the owner, with buttons to Create Gmail Draft, Snooze 24h and Mark Inaccurate.
- **HubSpot:** fields, flags and notes written through the API, tagged `[AI-Verified | Source ID]`.
- **Gmail drafts:** drafts were placed in the owner's drafts, never sent automatically. SMS went through Twilio.
- **GitHub:** the monthly drift pull requests on GTM Brain, reviewed by the GTM Brain reviewers.
- **Weekly digests and review packets:** Slack and Google Docs.
- **Jira:** escalation summaries for engineering.

## The quality loop

1. **Feedback score:** Useful against Inaccurate clicks. The target was at least 85% useful.
2. **Draft adoption and edit distance:** the share of drafts that were sent, and how much people changed them. Heavy edits on a section prompted a fix to its prompt.
3. **Alert cap:** at most 3 alerts per person per day, in real time, ranked by signal weight only.
4. **Matching outcomes:** did matched patients book, attend, and stay? These outcomes fed back into both matching and scoring.

Tuning reviews ran every two weeks with the Sales, Marketing and CX leads.
