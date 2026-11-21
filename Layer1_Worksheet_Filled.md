# Building an AI SRE — Layer 1: Problem Discovery (filled worksheet)

**Central question:** What problem are we actually solving?
**Built from:** None (first layer)
**Deliverable:** Artifact 1 — Product Requirements Document (`Artifact_1_Product_Requirements_Document.md`)
**Sources:** Discussion 1 (catch-up notes, about 1.5 h) and Discussion 2 (tensions and assumptions, transcript `layer1_transcript.txt`)
**Status:** Draft. All assumptions are provisional. Owners are still to be named (see the gaps list at the end).

Team: ____________ Room: ____________ Date: 2026-11-19
Members: Siddharth Pg, ____________, ____________

---

## Part 1 — Method record per question

Each question follows the worksheet method: Frame → Knowns → Unknowns → Assumptions → Resolve. Assumption IDs (A1–A10) refer to the register in Part 4. Tension IDs (T1–T11) refer to the tensions resolved in Discussion 2.

### A. Understanding the problem

#### Q1.1 Identify the underlying problem
- **Frame:** Behind frequent failure alerts, what is the problem that actually costs the organization?
- **Knowns:** The organization runs several services. Alerts are frequent. Logs, incident history and documentation exist.
- **Unknowns:** What share of incidents repeat a known pattern and what share are new (T9).
- **Assumptions:** A1, A2, A6.
- **Decision:** Most of the engineer's incident time goes into finding the cause: gathering evidence scattered across logs, metrics, history, documentation and code. The investigation starts from the failing service's evidence. Past incidents are used as context when they exist. When nothing similar exists, the investigation falls back to the code and recent changes. Every possible cause is a hypothesis until evidence confirms it.

#### Q1.2 Identify the people responsible
- **Frame:** When failures happen and ownership is unclear, who should handle them?
- **Knowns:** Every service has someone accountable: an on-call engineer, a named engineer or, by default, a manager (A4).
- **Unknowns:** How each organization's on-call arrangement is known to the platform (deferred to Layer 2).
- **Assumptions:** A3 (rejected), A4.
- **Decision:** The on-call engineer for the affected service is notified and handles the incident. The person who wrote the code is not assumed to own fixing it. Writing code and owning its repair are different responsibilities (A3). Finding the author through code history is out of scope for the first version.

#### Q1.3 A service goes down at 2 AM
- **Frame:** What does a night-time outage do to the business, its users and its engineers?
- **Knowns:** The impact depends on what failed, which region, how many users are affected and whether the feature earns revenue.
- **Unknowns:** Each organization's cost per minute of outage.
- **Assumptions:** A8 (night-time response is slow).
- **Decision:**
  - **Business:** lost revenue, operational disruption and lost reputation.
  - **Users:** cannot use the affected function. Severity depends on how much they rely on it.
  - **Engineers:** woken up, planned work disrupted, stress.
  - A failure in one feature leaves the others usable. A failure in a shared dependency spreads much wider.

#### Q1.4 What an engineer does today before finding the cause
- **Frame:** Walk through the engineer's steps from the alert to the cause.
- **Knowns:** The steps are: decide whether the alert is real or noise, judge its priority, identify who is affected, collect logs and signals, then investigate.
- **Unknowns:** How long each step takes today.
- **Assumptions:** A6, A9.
- **Decision:** All of these steps are manual today, and the investigation step costs the most time (A6).

#### Q1.5 What parts of the investigation can be automated
- **Frame:** Which repeated checks can be taken off the engineer?
- **Knowns:** Engineers repeatedly check logs, metrics, deployment history and past incidents.
- **Unknowns:** None that affect Layer 1.
- **Assumptions:** A1.
- **Decision:** These can be automated:
  - collecting evidence across services and pinpointing where in the logs the error is
  - looking up past incidents
  - finding linked components and estimating impact
  - forming and ranking hypotheses
  - proposing a priority
  - building and testing candidate fixes in an isolated copy

#### Q1.6 Repetitive investigation vs human judgment
- **Frame:** How do we tell routine investigation apart from decisions that need experience?
- **Knowns:** The team placed humans at specific checkpoints (T5).
- **Decision:** A decision needs a human when any one of these is true:
  1. It depends on what the organization values, for example which of two failures matters more (priority).
  2. It changes shared code or production (approving the fix, deploying).
  3. It chooses how to spend scarce time, for example re-running the platform or fixing the problem by hand.

  Evidence gathering and testing hypotheses are routine work and can be automated.

#### Q1.7 Information needed for root cause
- **Decision:** Root cause needs the following information:
  - logs and failure traces
  - metrics and service health
  - code and repository context
  - deployment history and recent changes
  - dependencies between services
  - configuration, runtime and library versions
  - past incident records and how they were fixed

#### Q1.8 Wrong diagnosis and unnecessary fix
- **Decision:** A wrong diagnosis or an unnecessary fix can cause these outcomes:
  - The original issue stays unresolved and time is wasted.
  - The fix may break another feature.
  - Users keep seeing failures and lose trust.
  - Engineers do the work twice: they review the wrong fix, then redo the investigation (T8).

  The team prefers leaving an issue unresolved for a while over introducing a wider regression.

### B. Understanding the product

#### Q1.9 Capabilities needed immediately after an incident
- **Frame:** What must happen in the first minutes after an alert?
- **Decision (T1):** The platform itself is the first to process the alert. It does five things:
  1. Gathers evidence.
  2. Estimates impact and linked components.
  3. Proposes a priority.
  4. Notifies the on-call engineer, who approves or changes that priority.
  5. Gives a debugging direction: likely areas to look at and two or three hypotheses.

  The proposed priority is the first human checkpoint (T5).

#### Q1.10 From explaining an error to fixing it
- **Decision:** On top of explaining an error, the platform must do four more things:
  1. Rank hypotheses.
  2. Fix each hypothesis in an isolated copy of the application.
  3. Test the fix against the original failure and check for regressions.
  4. Present the evidence, the fix and the test results to the engineer.

  The engineer can ask for changes in conversation, and the platform reworks and retests the fix. The engineer never has to edit code by hand. These conversations stay within the incident's scope (T4).

#### Q1.11 Who authorizes a production-affecting fix
- **Decision (T6):** The on-call engineer authorizes the fix, at any hour. In the first version, any engineer able to review the fix may approve it; no specific owner is required. Deployment stays a human action.
- **Open:** Night-time approval takes 2–3 hours on average and varies by company policy (A8).

#### Q1.12 The platform says it fixed the problem, but errors continue
- **Decision (T7):** Verification has two stages.
  - **Before release:** Tests in the isolated copy reproduce the original failure and check for regressions. The engineer may add tests.
  - **After release, if errors continue:** First decide whether the cause is an external service or our own code.
    - **External service:** Nothing can be fixed on our side. Wait, or contact the provider.
    - **Our own code:** Compare the new errors with the old ones. A different error means some progress. The on-call engineer then decides whether to run the platform again with the new evidence or to fix the problem by hand.
- **Open:** How long to wait before doubting an external service will recover. Which observable fact in production proves recovery.

#### Q1.13 Root cause cannot be established because telemetry is incomplete
- **Decision (T10):** Incomplete evidence stops the fixing work but not the reasoning. The platform still does four things:
  1. Shares the hypotheses it has, with the evidence behind each one.
  2. States what data is missing and why that blocks a conclusion.
  3. Hands the incident to the on-call engineer.
  4. Continues only on the engineer's input.

  The platform never guesses.

#### Q1.14 Suggestion assistant vs engineering platform
- **Decision:** A suggestion assistant only advises, so its risk is limited. An engineering platform investigates, builds and tests fixes, and coordinates people. It therefore needs evidence a human can check, a record of every action, and controls on what it may change.

#### Q1.15 What makes engineers trust it
- **Decision (T11):** Engineers trust the platform when they can check its work:
  - Each claim points to the exact file, lines and dependency in the organization's own codebase, so engineers can verify it themselves.
  - Its reasoning is visible.
  - Its tests can be reproduced by the engineer.
  - Past incident records support its hypotheses.
  - It never invents evidence.
  - Humans keep control of the checkpoints in Q1.22.
- **Open:** Whether the platform earns more autonomy over time was raised but not answered.

### C. Defining the product outcome

#### Q1.16 Smallest genuinely useful product
- **Decision (T3):** The MVP must contain a working fix, because the demo must end in verified recovery and management judges the investment on the demo.
  - **In scope:**
    - evidence gathering and pinpointing the error in the logs
    - past-incident context
    - dependency and impact analysis
    - hypotheses
    - priority classification, verified by a human
    - fixing in an isolated copy
    - tests
    - human approval
  - **Out of scope for now:**
    - finding the code author through history
    - paging anyone other than the on-call engineer
    - rollback
    - deployment

#### Q1.17 End-to-end demonstration
- **Decision:** The demo uses the reference scenario and runs in this order:
  1. The alert arrives.
  2. The platform proposes a priority and the on-call engineer approves it.
  3. The platform gathers evidence and forms hypotheses.
  4. It fixes the problem in an isolated copy.
  5. It tests the fix against the original failure and for regressions.
  6. It presents the evidence and test results to the engineer.
  7. The engineer approves.
  8. A human deploys.
  9. Recovery is confirmed.
- **"Recovered" means the defect is gone (T2).**

#### Q1.18 Measurable outcomes
- **Decision (T8):** Two measures:
  - **Time to resolve,** compared with the organization's recorded baseline (A9). The team gave a 20–30% reduction as an example of a meaningful saving.
  - **Accuracy:** the share of fixes the engineer approves without rework.

  A rejected fix counts as a failure, because it costs both the platform's time and the engineer's. Stopping early and reporting when evidence is insufficient counts as correct behaviour, not a failure.

#### Q1.19 Actions the platform must never take
- **Decision (T4):** Never, even if an engineer instructs it:
  - deploy to production
  - remove a whole feature
  - make a major architectural change that contradicts the existing codebase
  - work outside the incident's scope, for example fixing the cart when the incident is in payments
  - act as a general coding assistant

  **Allowed only with human approval:** line-level changes within the fix, including deleting lines, and merging the fix.

### D. Reference scenario

#### Q1.20 Full sequence from alert to recovery
- **Decision:**
  1. The alert is received.
  2. The platform gathers evidence from the affected service and pinpoints the error.
  3. It checks past incidents for context.
  4. It analyses linked components and impact.
  5. It forms hypotheses and proposes a priority.
  6. The on-call engineer verifies the priority. **[human checkpoint 1]**
  7. The platform fixes each hypothesis in an isolated copy.
  8. It tests the fix against the original failure and for regressions.
  9. It presents the fix, evidence and tests. The engineer may iterate in conversation.
  10. The engineer approves the fix before it is merged. **[human checkpoint 2]**
  11. A human deploys.
  12. Recovery is confirmed.

  If evidence is insufficient at any step, the platform stops fixing and hands over (Q1.13). If tests fail, it returns to step 2.
- **Conflict to settle:** see "Consistency with the reference scenario" in Artifact 1, section 1.5.

#### Q1.21 Preserving information
- **Decision:** One incident record is updated after every step. It holds the evidence, with timestamps and source locations, plus the hypotheses, actions, reasoning, test results, missing data and human decisions. Anyone can trace a conclusion back to its source.

#### Q1.22 Exact points for a human
- **Decision (T5):**
  1. **Priority verification**, when the on-call engineer is notified. Priority depends on organizational context the platform does not have.
  2. **Fix approval** before the fix is merged. The engineer checks that the changes follow the root-cause analysis and that builds and tests pass.
  3. **Deployment**, which only a human performs.

  There are also two decision hand-offs:
  - when evidence is insufficient
  - when errors continue after release (run the platform again, or fix by hand)

---

## Part 2 — Conversation log

| Questions | Agent | Conversation link | Decision reached | Open unknowns and owner | Artifact section |
| --- | --- | --- | --- | --- | --- |
| Q1.1–Q1.22 (first pass) | Team discussion | Catch-up notes | First answers to all 22 questions, plus an added item 10 | Spread across T1–T11 | All |
| Analysis of the first pass | C | (add link) | Identified 11 tensions and 10 unstated assumptions | — | All |
| T1, T3, T5, T6 → Q1.9, Q1.16, Q1.17, Q1.22, Q1.11 | Team | `layer1_transcript.txt` | Platform processes the alert first; MVP includes fixing; two human checkpoints; on-call engineer approves | Night-time approval delay (owner: ___) | 1.2, 1.3, 1.5 |
| T2, T7 → Q1.12, Q1.17, Q1.20 | Team | `layer1_transcript.txt` | "Recovered" means the defect is gone; rollback is outside the platform; two-stage verification | How long to wait for an external service; what fact proves recovery in production (owner: ___) | 1.4, 1.6 |
| T4 → Q1.19 | Team | `layer1_transcript.txt` | Never-list agreed; works conversationally within the incident's scope | — | 1.5, 1.7 |
| T8, T9, T10, T11 → Q1.18, Q1.1, Q1.13, Q1.15 | Team | `layer1_transcript.txt` | Time and accuracy are the measures; history is context only; incomplete evidence stops fixing only; trust comes from traceability | Does autonomy change over time? (owner: ___) | 1.1, 1.4, 1.6 |
| A1–A10 | Team | `layer1_transcript.txt` | Register in Part 4 | Owners not yet named | 1.2, 1.6 |
| (all) | G | **not yet run** | — | — | — |

## Part 3 — Where Claude and ChatGPT disagreed

**Not yet recorded.** The worksheet requires separate conversations with Claude and ChatGPT, and this has not been done.

The team did settle several internal disagreements during Discussion 2:

| Disagreement | Positions | Resolution |
| --- | --- | --- |
| Should fixing be in the MVP? | Drop fixing and only identify and assign, vs keep fixing | Keep fixing, because Q1.17 requires a verified recovery |
| Human at priority-setting? | Humans only at the end, because a human at every step defeats the purpose, vs a human at priority too | Human at priority as well: it is quick, and priority depends on the organization |
| Conversational iteration or fixed output? | The engineer edits the fix by hand, vs the engineer asks the platform for changes | Conversational, but limited to the incident's scope |
| Who approves? | Service owner vs on-call engineer | On-call engineer |

## Part 4 — Assumption register (all provisional)

| ID | Assumption | Team position | Validation condition | Owner |
| --- | --- | --- | --- | --- |
| A1 | Logs, metrics and history exist and are reachable | Treated as known; at least 6 months of history, plus live logs and metrics, will be requested from the organization | Holds if the organization provides at least 6 months of history and live access. **Conflicts with Q1.13**, which plans for incomplete telemetry | ___ |
| A2 | Past incidents are documented well enough to learn from | Treated as known; the organization's own records are considered trustworthy | Holds if past incident records include what fixed each incident | ___ |
| A3 | Code history points to the right person | Rejected: writing code and owning its repair are different | Not needed while the on-call engineer handles incidents | ___ |
| A4 | One accountable owner exists per service | Holds: the on-call engineer, a named engineer, or a manager by default | Holds if each service maps to an on-call rota or a named fallback | ___ |
| A5 | The failure can be reproduced outside production | Holds, using an isolated copy of the same code | Holds if the reference failure can be triggered in the isolated copy | ___ |
| A6 | The main pain is investigating and gathering scattered information | Holds | Holds if measured incident time shows that investigation is the largest share | ___ |
| A7 | Noise can be told apart from real incidents reliably | Considered low risk, because a human verifies priority at the first checkpoint | Holds if alerts the platform classes as noise are still visible to the on-call engineer | ___ |
| A8 | A human is available to approve when needed | Holds: on-call engineers respond within minutes during working hours and in 2–3 hours on average at night, depending on company policy | Measured response time of the on-call engineer | ___ |
| A9 | A baseline for human-only resolution can be measured | Treated as known: time-to-resolve is recorded, by priority | Holds if records of past resolution times are obtained | ___ |
| A10 | Only engineers use the platform | Refined: the on-call engineer is the user; team leads, managers and support have read-only access | Holds if the readers confirm that read access is enough | ___ |

## Remaining gaps before Layer 1 can be closed

- [ ] Name an owner for every open unknown and for every assumption in Part 4.
- [ ] Run Claude and ChatGPT separately on Layer 1 and record where they disagree (Part 3).
- [ ] Add conversation links to Part 2.
- [ ] Settle the conflict with the reference scenario (Artifact 1, section 1.5).
- [ ] Settle the conflict between A1 and Q1.13.
- [ ] Answer the remaining open points: how long to wait for an external service, which fact proves recovery in production, and whether autonomy changes over time.
- [ ] Have a reviewer sign off and save the files in the team repository.
