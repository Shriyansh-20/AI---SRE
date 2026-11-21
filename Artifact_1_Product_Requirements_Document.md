# Artifact 1 — Product Requirements Document

**Layer:** 1 — Problem Discovery
**Built from:** None (first layer)
**Supporting record:** `Layer1_Worksheet_Filled.md` (method record, conversation log, assumption register)
**Status:** Draft. All assumptions are provisional. Reviewer sign-off is pending.

| Section | Questions this section answers | Done |
| --- | --- | --- |
| 1.1 Problem statement | Q1.1, Q1.3, Q1.4, Q1.7, Q1.8 | ☑ |
| 1.2 Target users | Q1.2, Q1.6, Q1.11, Q1.22 | ☑ |
| 1.3 User journey | Q1.4, Q1.9, Q1.20, Q1.21 | ☑ |
| 1.4 Core use cases | Q1.5, Q1.9, Q1.10, Q1.12, Q1.13, Q1.17 | ☑ |
| 1.5 Product boundaries | Q1.6, Q1.11, Q1.14, Q1.19, Q1.22 | ☑ |
| 1.6 Success definition | Q1.15, Q1.16, Q1.18 | ☑ |
| 1.7 Explicit non-goals | Q1.14, Q1.16, Q1.19 | ☑ |

---

## 1.1 Problem statement

On-call engineers lose most of their incident time finding the cause of a failure. They gather evidence scattered across logs, metrics, past incidents, documentation, code and recent changes before any fix can begin. The cost falls on three groups:

- **Businesses** lose revenue and reputation while the service is down.
- **Users** cannot use the failing function.
- **Engineers** are woken at night, pulled off planned work and put under stress.

A wrong diagnosis makes things worse. The issue stays unresolved, a new regression may appear, and the work is done twice. The problem persists because the evidence is fragmented and every incident is investigated by hand from scratch.

*Answers Q1.1, Q1.3, Q1.4, Q1.7, Q1.8.*

## 1.2 Target users

| Role | Relationship to the product | What they need |
| --- | --- | --- |
| On-call engineer (primary user) | Receives the incident, verifies priority, reviews and approves the fix, deploys, decides what happens next if errors continue | A fast direction, evidence they can check, a tested fix and clear hand-offs |
| Approving engineer | Any engineer able to review the fix (first version). No specific code owner is required | The complete chain from evidence to diagnosis to fix to test results |
| Accountable fallback (manager) | Takes responsibility for a service with no on-call engineer | To know an incident exists and who is handling it |
| Team leads, managers, support (readers) | Read-only view of incidents | Status, impact, and what was done and why |

The author of the failing code is **not** assumed to own its repair.

*Answers Q1.2, Q1.6, Q1.11, Q1.22.*

## 1.3 User journey (the on-call engineer's view)

1. **Notified.** The engineer receives an incident with impact, affected components, a proposed priority and two or three hypotheses with their evidence.
2. **Verifies priority.** They approve or change the priority, because priority depends on what the organization values. This is also where they decide whether the alert is noise; the first version has no separate noise filter. *[human checkpoint 1]*
3. **Waits while the platform works.** It fixes the hypotheses in an isolated copy and tests each fix against the original failure and for regressions.
4. **Reviews.** They see one record that traces evidence → root cause → change → test results, with links to every source. They may ask for changes in conversation; the platform reworks and retests.
5. **Approves.** They approve the change before it is merged. *[human checkpoint 2]*
6. **Deploys.** The engineer, or another human, deploys the change.
7. **Confirms recovery.** The defect is gone: the services are back up and the server no longer reports errors.
8. **If errors continue after release:**
   - If an external service is failing, they wait or contact its provider. How long to wait depends on the service; there is no fixed limit, so the engineer judges each case.
   - If the failure is in their own code, they choose between running the platform again with the new evidence or fixing the problem by hand.
9. **If evidence is insufficient at any point:** they receive the evidence and hypotheses the platform has, plus a list of what is missing, and give their input to direct the next step.

The incident record is updated at every step and stays as the history of the incident.

*Answers Q1.4, Q1.9, Q1.20, Q1.21.*

## 1.4 Core use cases

| # | Use case | Outcome for the engineer |
| --- | --- | --- |
| UC1 | Alert arrives: gather evidence, pinpoint the error, estimate impact, propose a priority | Starts with a direction instead of a blank page |
| UC2 | Investigate using past incidents as context, or the code and recent changes when nothing similar exists | Hypotheses backed by evidence, for repeat and new incidents alike |
| UC3 | Build and test a fix in an isolated copy, against the original failure and for regressions | A fix that has already been shown to work before review |
| UC4 | Present the fix and iterate on the engineer's feedback, within the incident's scope | No manual code editing; changes stay within the incident |
| UC5 | Evidence is insufficient: stop fixing, share hypotheses and the missing data, hand over | No blind fixes and no wasted review time |
| UC6 | Errors continue after release: tell an external failure from our own; hand the next decision to the engineer | A clear choice instead of a repeated wrong claim |
| UC7 | End-to-end demo on the reference scenario, ending in verified recovery | Management sees the whole cycle working |

*Answers Q1.5, Q1.9, Q1.10, Q1.12, Q1.13, Q1.17.*

## 1.5 Product boundaries

The platform's actions fall into three groups.

**The platform does on its own:**

- evidence gathering
- past-incident lookup
- impact and dependency analysis
- hypotheses
- proposing a priority
- fixing and testing in an isolated copy
- keeping the incident record

**Only with a human decision:**

- the final priority, including whether the alert is noise
- merging the fix
- deployment, which a human performs
- what to do when evidence is missing
- what to do when errors continue after release

**Never, even if an engineer instructs it:**

- deploy to production
- remove a whole feature
- make a major architectural change that contradicts the existing codebase
- work outside the incident's scope
- act as a general coding assistant

**Rationale:**

- A human is placed where a decision depends on organizational context (priority), changes shared code or production (merge, deploy), or spends scarce engineering time (re-run the platform or fix by hand).
- Everything repetitive is automated.
- A suggestion assistant only advises. This platform acts, so it must keep every claim checkable and every action recorded (Q1.14).

**Consistency with the reference scenario — decided.** In the reference scenario the new version is live and returning server errors. The platform's part is to fix that production error: it builds and tests the fix, then waits for the on-call engineer's approval. Its work ends there. Rollback stays out of scope for two reasons. The right recovery action depends on the kind of error, and a rollback would need human approval anyway. Until the fix is approved and deployed, any interim recovery of the running service is the on-call engineer's call, outside the platform. The team accepts that night-time approval can take 2–3 hours (A8).

*Answers Q1.6, Q1.11, Q1.14, Q1.19, Q1.22.*

## 1.6 Success definition

The platform succeeds when engineers resolve incidents faster than their own recorded baseline, without losing quality.

| Measure | Definition | Counts as failure |
| --- | --- | --- |
| Time to resolve | Alert to defect gone (services back up, server no longer reporting errors), compared with the organization's recorded time-to-resolve by priority (A9). The team gave a 20–30% reduction as an example of a meaningful saving | — |
| Accuracy | Share of fixes approved without rework | A rejected fix: the platform's time and the engineer's are both wasted |
| Correct restraint | Stopping and reporting when evidence is insufficient | Proceeding blindly to a fix without enough evidence |

**Trust conditions** (Q1.15):

- Each claim points to the exact file, lines and dependency in the organization's own codebase.
- Its reasoning is visible.
- Its tests can be reproduced by the engineer.
- Past incident records support its hypotheses.
- It never invents evidence.

Trust does not turn into more autonomy over time: the human checkpoints stay fixed.

**MVP** (Q1.16): everything in "does on its own" in section 1.5, plus human approval. The MVP has to include fixing, because management judges the investment on a demo that ends in verified recovery.

*Answers Q1.15, Q1.16, Q1.18.*

## 1.7 Explicit non-goals

- **Deploying, rolling back or restoring a previous version.** Humans own release and recovery of the running service.
- **Finding the author of failing code through history, or paging anyone other than the on-call engineer** (first version).
- **Removing features, redesigning architecture, or writing new features.** The platform fixes bugs; it does not build features.
- **Fixing anything outside the current incident's scope.**
- **Acting as a general coding assistant or chatbot.**
- **Deciding business priority, or dismissing an alert as noise, without a human.** The first version has no automatic noise filter.
- **Gaining autonomy over time.** The human checkpoints are permanent.
- **Producing a root cause or a fix when evidence is insufficient.**

*Answers Q1.14, Q1.16, Q1.19.*

---

## Assumptions (provisional)

The full register, with validation conditions, is in `Layer1_Worksheet_Filled.md`, Part 4. In summary:

- **A1, A2, A9** were treated as known by the team but are not yet evidenced. For A1 the team's position is that logs are always available in practice. Q1.13 covers only the edge case that question poses, where telemetry is incomplete.
- **A3** was rejected.
- **A4–A8 and A10** hold, or were refined, with conditions attached.
- **Owners are not yet named.**

## Resolved items (decided 2026-11-21)

| Item | Decision | Source |
| --- | --- | --- |
| Reference scenario vs rollback | The platform fixes the production error and waits for on-call approval; its work ends there. Rollback is out of scope: it depends on the kind of error and needs human approval anyway | 1.5, Q1.20 |
| Complete vs missing logs | Logs are always available in practice. Q1.13 answers that question's edge case only | A1, Q1.13 |
| Waiting on an external service | No fixed limit; it depends on the service, and the on-call engineer judges each case | Q1.12 / T7 |
| Fact that proves recovery | The services are back up and the server no longer reports errors | Q1.12, Q1.17 |
| Autonomy over time | None: the human checkpoints are permanent | Q1.15 / T11 |
| Noise | No noise filter in the first version. The human decides noise at priority verification, and every alert reaches the on-call engineer | A7, Q1.9 |

## Acceptance checks

- [x] Every required section is present and complete.
- [x] Every section lists the questions it answers, and every question in the layer is answered by at least one section.
- [ ] Every assumption is marked provisional, with a named owner and a validation condition. *(Conditions are written; owners are missing.)*
- [ ] Every disagreement between Claude and ChatGPT that affected a decision is recorded with the team's resolution. *(The ChatGPT comparison has not been run.)*
- [x] The artifact is consistent with every artifact it is built from. *(No earlier artifacts. Consistent with the reference scenario; the rollback decision is recorded in 1.5.)*
- [ ] The artifact was produced with the team's coding agent and saved in the team repository. *(It is saved in the SRE_Agent folder; that folder is not yet a repository.)*

Reviewed by: ____________ Date: ____________
