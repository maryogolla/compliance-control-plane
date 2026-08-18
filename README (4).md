# Operations Control Plane — a design exercise

**[Live demo](https://maryogolla.github.io/compliance-control-plane/)**

How I would structure compliance operations for a B2B platform licensor operating across
multiple regulated markets.

Built from **public sources only**. Not a description of any company's actual systems.

## The framing

A platform licensor typically holds no gaming licence and no customer funds — its local
partners hold those. So its obligation is not "keep our licence." It is to ensure the
platform lets each licensee satisfy what *their* regulator requires, across every market
at once, without a change made for one market silently breaking another.

That is a requirements-mapping and change-propagation problem, which is an operations
discipline.

## What's here

- **Control plane** — market graph where edges are shared controls, so the graph encodes
  propagation risk. Plus six telemetry panels.
- **Architecture** — four lanes: ingest, automation, register, and the escalation ladder.
  Judgement visibly leaves the automation and goes to a person with authority.
- **Automation layer** — fourteen Make.com scenarios, each with its trigger, what it
  writes to and where it stops. A hard line marks where automation ends, plus the list of
  what must never be a scenario.
- **Change management** — ten steps to remember, thirteen stages to interrogate, with the
  four stages people skip (SOP update, training, effective-date tracking, post-launch
  monitoring) called out.
- **Control anatomy** — the ten questions that make a control real, worked in full for
  CTL-KYC-01 and applied as a template to every control. `provable` is computed, and five
  of sixteen controls fail it.
- **Provenance** — every control traced from statute through regulator instrument and
  licence condition down to dated evidence, with two computed alerts: orphaned controls
  (no authority up the chain) and superseded chains (a dead rule still being enforced).
- **Register** — seventeen requirements, each carrying the instrument it derives from or
  flagged unsourced.
- **Markets** — click any node on the graph for the drill-down, which states shared
  dependencies in words: which other markets a change would reach, and through which
  control.
- **Versions** — policies, SOPs, scenarios and configuration all versioned, with approver,
  approval date, effective date and a plain-language diff. A config change is a control
  change.
- **Ownership** — nine departments across three lines of defence, a RACI matrix that flags
  rows with no accountable department or more than one, and departmental load.
- **Doc health** — stale documents, orphaned documents, undocumented controls,
  process-doc drift and training gaps.
- **End to end** — fifteen stages across four lanes, from a regulator publishing something
  to being able to prove you complied. Press play to walk it; each stage links through to
  the view holding its detail.
- **Change simulator** — pick a regulatory change, watch the blast radius compute, and see
  which shared controls put other markets at risk.
- **Approach** — the reasoning, including what I would need to learn.

Every identifier in the app is a link, every status-strip metric filters into the view it
summarises, and every count in the strip is computed from the underlying data rather than
typed, so the figures cannot drift apart from the views they summarise.

## Two principles

**Automate execution, not accountability.** Execution is what a scenario does;
accountability is who answers for it. A workflow tool can hold the first and can never
hold the second. Status, routing, reminders, evidence capture, escalation triggers —
automate all of it. Whether a discrepancy is acceptable, whether an alert is a true match,
whether to accept a risk — those stay with a person who holds the authority. The
corollary: automation should make escalation *easier*, not rarer.

**Audit preparation becomes continuous audit readiness.** Evidence captured as activity
happens makes an audit a query. Evidence reconstructed afterwards makes it a crisis.

## Build guide

Step-by-step, from an empty file to a live URL: [BUILD_GUIDE.md](BUILD_GUIDE.md)

## Author

Mary Ogola · maryogolla040@gmail.com · [linkedin.com/in/mogola](https://linkedin.com/in/mogola)

## Sources

Kenyan entries are cited to primary sources — the Gambling Control Act 2025 (Act No. 14 of
2025), the Proceeds of Crime and Anti-Money Laundering Act 2009, the Data Protection Act
2019 and the Finance Act 2025 — and marked `verified` in the provenance view. Everything
else is marked `illustrative`. Staleness is measured from a fixed as-at date so the figures
stay reproducible.

*Illustrative data. Design exercise from public sources. Not legal advice. Not affiliated
with or endorsed by any company referenced.*
