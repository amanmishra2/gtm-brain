# GTM Brain: the full write-up

> A write-up of a system I built at a two-sided mental-health marketplace. It ran inside company systems, so the code isn't public.

These pages are the detailed write-up of GTM Brain, the shared go-to-market (GTM) context layer, written as a retrospective product requirements document (PRD): why I built it, how it worked, what its outputs looked like and how each team worked with it. The [main README](../README.md) is the short version.

## The pages

1. [Overview](overview.md): the business it served, my role, the problem, goals and non-goals, the principles, the topology and the main decisions.
2. [Context layers](context-layers.md): the reviewed GitHub repository, the structured state and the unstructured signals the workflows drew on.
3. [Pipeline](pipeline.md): identity resolution, scoring, context assembly, the schemas, the system-prompt rules and validation.
4. [Workflows](workflows.md): every workflow by timing, with sample outputs, the funnels and who got which alert.
5. [How each team used it](how-teams-used-it.md): Marketing, Sales, customer experience (CX), Operations and my own revenue operations (RevOps) work.
6. [Review and governance](review-and-governance.md): who reviewed what, how review ran, the automated checks, the delivery channels and the quality loop.
7. [Results](results.md): adoption, the pilot before rollout, and the headline result.

## How to read them

- Read them in order, or go straight to the page you need.
- Every sample brief, alert, payload and entry is synthetic: made-up people, no dates and no real wording. Each has a line saying so directly above it.
- Each workflow id names its timing: RT for real time, JIT for before a call or review, POST for after a call or escalation, WK for weekly and MO for monthly.
- Acronyms are spelled out the first time they appear on each page.
