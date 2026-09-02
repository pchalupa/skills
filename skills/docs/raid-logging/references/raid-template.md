# RAID Log: <project name>

> A living register of **R**isks, **A**ssumptions, **I**ssues, and **D**ependencies. Reviewed regularly; entries are closed, never deleted; IDs are stable and never reused.

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
| R-1 | *(example)* A third-party API may miss its announced release date; if it does, the integration milestone will slip | Confirm the release date with the provider; agree on a fallback data source; track the dependency weekly | MEDIUM | HIGH | <@owner> | OPEN |

> Risk statuses: `OPEN` · `MITIGATING` · `REALISED` (materialised — close and open a matching Issue) · `CLOSED` (no longer possible).

## Assumptions
_Things we are treating as true to move forward. Track confidence, not likelihood._

| ID | Assumption | Impact if wrong | Owner | Status | Validated by |
|------|-----------|-----------------|-------|--------|--------------|
| A-1 | *(example)* The identity provider will support the required SSO protocol before integration begins | Authentication work and the release milestone will slip | <@owner> | OPEN | |

> Assumption statuses: `OPEN` · `VALIDATED` (confirmed true) · `INVALIDATED` (proved false — assess impact, likely open a Risk or Issue).

## Issues
_Problems happening right now that need action._

| ID | Issue | Date raised | Owner | Severity | Status |
|------|-------|-------------|-------|----------|--------|
| I-1 | *(example)* The test environment is unavailable, blocking integration verification | <YYYY-MM-DD> | <@owner> | HIGH | OPEN |

> Issue statuses: `OPEN` · `IN PROGRESS` · `RESOLVED` · `CLOSED`. An issue realised from a risk notes it: *"realised from R-n"*.

## Dependencies
_Things we are waiting on from someone else. Track who logs it and what it blocks._

| ID | Dependency | Depends on | Blocks | Tracker | Status |
|------|-----------|------------|--------|---------|--------|
| DE-1 | *(example)* Reporting cannot be tested until a representative test dataset is available | Data platform team | Reporting verification | <@owner> | OPEN |

> Dependency statuses: `OPEN` · `AT RISK` · `MET` · `CLOSED`.

---

> Keep entries specific and actionable. Close items — don't delete them. Never reuse or renumber an ID.
