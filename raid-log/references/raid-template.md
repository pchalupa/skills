# RAID Log: <project name>

> A living register of **R**isks, **A**ssumptions, **I**ssues, and **D**ependencies.
> Reviewed regularly; entries are closed, never deleted; IDs are stable and never reused.

| | |
|---|---|
| **Log owner** | <name> |
| **Last reviewed** | <YYYY-MM-DD> |
| **Review cadence** | <e.g. weekly at sprint review> |

**Legend** — Likelihood / Impact / Severity: `HIGH` · `MEDIUM` · `LOW`.

---

## Risks
_Things that might happen and would hurt delivery if they did. Forward looking._

| ID | Risk | Mitigation | Likelihood | Impact | Owner | Status |
|------|------|------------|------------|--------|-------|--------|
| R-1 | *(example)* Activity API drives the Home dashboard and is unplanned at ~6 months of work; if it slips, Home can't ship for MVP on the planned date | Sequence the roadmap around API availability; design a reduced-feed Home fallback; push for an interim data source and a committed date | MEDIUM | HIGH | <@owner> | OPEN |

> Risk statuses: `OPEN` · `MITIGATING` · `REALISED` (materialised — close and open a matching Issue) · `CLOSED` (no longer possible).

## Assumptions
_Things we are treating as true to move forward. Track confidence, not likelihood._

| ID | Assumption | Impact if wrong | Owner | Status | Validated by |
|------|-----------|-----------------|-------|--------|--------------|
| A-1 | *(example)* Most backend and API issues are resolved by H2, which is what enables the app rebuild | Roadmap and the MVP launch date slip; several core APIs aren't available at build time | <@owner> | OPEN | |

> Assumption statuses: `OPEN` · `VALIDATED` (confirmed true) · `INVALIDATED` (proved false — assess impact, likely open a Risk or Issue).

## Issues
_Problems happening right now that need action._

| ID | Issue | Date raised | Owner | Severity | Status |
|------|-------|-------------|-------|----------|--------|
| I-1 | *(example)* API documentation is still locked and the team has no token to call the APIs, blocking integration design | <YYYY-MM-DD> | <@owner> | HIGH | OPEN |

> Issue statuses: `OPEN` · `IN PROGRESS` · `RESOLVED` · `CLOSED`. An issue realised from a risk notes it: *"realised from R-n"*.

## Dependencies
_Things we are waiting on from someone else. Track who logs it and what it blocks._

| ID | Dependency | Depends on | Blocks | Tracker | Status |
|------|-----------|------------|--------|---------|--------|
| DE-1 | *(example)* Native online auction registration and bidding can't be built or tested until the online APIs are delivered | Client's AMB team (Auction Management & Bidding) | Online registration + bidding feature development and QA | <@owner> | OPEN |

> Dependency statuses: `OPEN` · `AT RISK` · `MET` · `CLOSED`.

---

> Keep entries specific and actionable. Close items — don't delete them. Never reuse or renumber an ID.
