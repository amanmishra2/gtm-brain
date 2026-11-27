# Context layers

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

GTM Brain, the shared go-to-market (GTM) context layer, drew on three layers of context: Layer 1, canonical truth; Layer 2, structured state; and Layer 3, unstructured signals.

Without reviewed ground truth, a model writes generic copy. It can also promise a therapist who isn't available, quote insurance coverage that isn't true, or offer terms nobody approved.

## Layer 1: canonical truth

Layer 1 was the GitHub repository. Each entry carried metadata tags, so a workflow loaded only the entries that matched the audience, state and persona in front of it.

**Format.** Markdown documents in a folder for each type, with a structured file for each entry.

**What it held.** The repository held nine Layer 1 entry types, plus the CX playbooks as a tenth. CX is customer experience, and ICP is the ideal customer profile.

| Entry type | Contents | Tags | Reviewed by | Updated |
|---|---|---|---|---|
| ICP and segments | Patients: issues, goals, therapy-style preferences, in person or teletherapy, how they pay. Therapists: license type, states, specialties, modalities, insurance panels, how many platforms they're on. Practices: size, specialties, billing needs. | `audience`, `state`, `segment` | Leadership, Marketing, CX | Quarterly |
| Personas and pain points | A persona and its pains for each audience | `audience`, `persona` | Leadership, Marketing, CX | Quarterly |
| Positioning, messaging matrices, value props | The core value for each audience, a pain-to-message map, and brand voice | `audience`, `persona`, `stage` | Marketing, CX | Quarterly |
| Claims, proof points, case studies | Each claim with its proof, and the case studies approved for each audience | `audience`, `claim_type` | Sales, Marketing, CX | Monthly |
| Objection handling | Patients: cost, insurance, fit, wait time. Therapists: fees, client volume, admin. Practices: switching billing. Each with an approved response. | `audience`, `objection_type` | Sales, Marketing, CX | Monthly |
| Competitor battlecards | Other therapy platforms: strengths, gaps, and how to answer them | `competitor`, `audience` | Sales, Marketing, CX | Monthly, or when something changes |
| Pricing, packaging and insurance guardrails | Session prices, accepted insurance, therapist terms, group-practice billing fees: what may be said, and what needs a person | `audience`, `payer` | Marketing, CX | As needed |
| GTM rules and ownership | Who owns which lead type and stage, routing rules, and response times (a 30-minute re-notify and a 1-hour step-in) | `lead_type`, `owner` | Marketing, CX | As needed |
| Clinical-language rules | No diagnosis, no outcome promises | `audience` | Marketing, CX | As needed |
| CX playbooks | — | — | Marketing, CX | Every two weeks |

The CX playbooks also lived in Google Docs.

**A claim entry.** A claim or proof-point entry had an id, its wording, its evidence, the audience, an owner, a status, who approved it (`approved_by`) and a tone. It had no review date and no do-not-use field. Claims and approved messaging were separate entries.

*Synthetic example, not company data.*
```markdown
**CLAIM ENTRY: claim.therapist.<claim-name>**
* id: claim.therapist.<claim-name>
* wording: <the claim, in approved words>
* evidence: case.therapist.caseload-growth
* audience: therapists
* owner: <owning team>
* status: approved
* approved_by: <reviewer>
* tone: <tone>
```

How entries were reviewed and approved is on [review and governance](review-and-governance.md).

## Layer 2: structured state

n8n and Python synced it into PostgreSQL.
- **HubSpot:**
  - contacts (patients and therapists) and companies (group practices);
  - lifecycle stage, lead source, lead score, owner and routing group;
  - Clay enrichment fields;
  - the three lines (patients, therapists, group practices) in one HubSpot pipeline, a standard deal pipeline.
- **The in-house platform and its database:** the patient survey, the therapist pre-call form, the algorithm's top therapist matches, each therapist's calendar and earliest openings, bookings, cancellations and sessions attended. Webhooks sent new events to Slack.
- **Billing and insurance (Stripe):** payment method, insurance on file, and, for practices, billing-service activity.
- **Freshdesk metrics:** open tickets, priority, service-level agreement (SLA) breaches, satisfaction scores.
- **Web analytics (GA4):** page views and form starts on the patient, therapist and practice pages.

## Layer 3: unstructured signals

- **Google Voice calls:** patient matching calls, therapist calls and practice calls, turned into text by Google Voice's own transcripts. Calls were recorded with consent.
- **Gmail threads.**
- **Slack:** internal team channels.
- **Freshdesk tickets:** the full ticket text and agent notes.
- **Survey and pre-call forms:** patients' free-text survey answers, and therapists' pre-call form answers. Both lived in the in-house platform.
- **LinkedIn Sales Navigator:** saved leads, job changes and replies, for therapist and practice outbound. Its data came in through the Sales Navigator–HubSpot integration, through Clay.
- **Web intent:** repeat visits to the therapist and practice pages. Patients were not tracked this way.

## No records in the repository

The GTM Brain repository held no patient, therapist or member records. Before a model saw a record from Layer 2 or Layer 3, the fields its task didn't need were dropped (see [pipeline](pipeline.md#identity-resolution)).
