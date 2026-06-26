# Issues Analysis — Tosca DI Dogfooding
_Iteration 2 — revised per critic-review-1.md_

## 1. Distribution

### By Severity

| Severity | Count | % of total |
|---|---|---|
| Major | 14 | 82% |
| Minor | 3 | 18% |
| **Total** | **17** | |

The 82% Major rate is unusually high for a UX triage table. It reflects the nature of this dogfooding session: most friction encountered was execution-blocking, not cosmetic. Minor issues (#7, #15, #16) are all navigational or informational — they do not block the primary task.

### By Issue Type

| Issue Type | Count | Issues |
|---|---|---|
| Feedback | 6 | #2, #4, #6, #8, #9, #11, #12 — minus #13 (reclassified; see note) |
| Help & guidance | 8 | #1, #3, #5, #10, #13, #14, #15, #17 |
| Usability | 3 | #7, #13 (also Help & guidance — see note), #16 |

> **Reclassification note — #13:** Issue #13 ("SQL Editor shows 'Unknown Connection', Run SQL button disabled with no tooltip") was previously classified as Feedback. The dominant failure is a missing affordance: the disabled button provides no tooltip or guidance. This matches Help & guidance, not Feedback. Feedback issues in this taxonomy describe cases where the system communicates something but communicates it incorrectly; #13 describes a case where the system communicates nothing. Reclassified to Help & guidance. Distribution updated: Feedback = 6, Help & guidance = 8.

### By Feature Area

| Feature Area | Count | Issues |
|---|---|---|
| Data Integrity | 6 | #1, #2, #3, #4, #5, #6 |
| Launcher | 6 | #8, #9, #10, #11, #12, #14 |
| Launcher / Builder | 1 | #12 (see tagging note below) |
| Builder — SQL Editor | 1 | #13 |
| Data Integrity / Launcher | 1 | #17 |
| Agents / Run tests | 1 | #15 |
| Playlists / Run tests | 1 | #16 |

> **Tagging note — #12:** The source table lists #12 under both "Launcher" and "Launcher / Builder". The more precise tag is "Launcher / Builder" — the silent failure surfaces in the Builder UI after a Launcher-triggered event. Count adjusted: Launcher standalone = 5 issues (#8, #9, #10, #11, #14); Launcher / Builder = 1 issue (#12).

### By Screen/Component (derived from issue table)

| Screen/Component | Issue count | Issues |
|---|---|---|
| DI Connections — Create connection form | 3 | #3, #4, #5 |
| Tricentis Launcher — component/extension install | 4 | #8, #9, #10, #11 |
| Builder — SQL Editor | 3 | #1, #2, #13 |
| Builder — after failed run | 1 | #12 |
| DI Connections — Test Connection | 1 | #6 |
| Windows Security | 1 | #14 |
| All Assets — Test Cases search | 1 | #7 |
| Agents page | 1 | #15 |
| Playlist — Test run dropdown | 1 | #16 |
| Tosca DI Executor — first run | 1 | #17 |

---

## 2. Top 3 Recurring Patterns

### Pattern 1: Silent failure with no root cause

The product surfaces an error state but withholds the information needed to diagnose or fix it. This pattern appears in 6 logged issues and is further evidenced by 2 unlogged branch nodes — it is the dominant failure mode across the entire session.

Logged evidence:
- **#2** — "SQL command execution timed out" names the symptom, not the cause (query, connection, or server)
- **#6** — Test Connection runs server-side but gives no indication; timeout is misread as connection failure
- **#8** — TBox quarantined by antivirus; UI shows only "Component executable not found" — actual cause in log file only
- **#11** — Extension helper exe blocked; error names the file but provides no actionable steps
- **#12** — Test run silently fails when extension cannot connect; user returns to Builder with no error
- **#17** — Executor path named in error but no instruction to create the folder

Unlogged evidence (branch nodes with no corresponding issue):
- **Branch D1** — "Exactly 1 source must be defined" error provides no guidance on how to define a source correctly for CSV
- **Branch D2** — "Expected [Select] Actual: Input" error contradicts D1; two error messages in the same configuration loop point in opposite directions

All 6 logged cases are Major severity. All are Feedback type. The common structure: the system detects a failure state and surfaces a message, but the message stops at symptom identification. No cause, no next step, no documentation link. The Branch D cases extend this pattern into undocumented territory.

### Pattern 2: Undisclosed platform and architecture prerequisites

The product assumes infrastructure conditions (Windows OS, Windows VM agent, Launcher installed, antivirus exclusions in place) that are never communicated before setup begins. Users discover these requirements through failure.

Evidence:
- **#1** — No guidance that a working DB with accessible tables is required before DI module use
- **#3** — No indication that SQLite file path must be on the agent machine, not the local Mac
- **#5** — Entire DI setup flow assumes Windows; Mac users hit multiple silent blockers with no in-product guidance (see scope note below)
- **#10** — Launcher installs browser extension with no prior warning; all 6 issues in this pattern are currently tagged Major
- **#14** — No in-product guidance that corporate security policies may block execution
- **#17** — ToscaDataIntegrityExecutor folder not created automatically on first run

All 6 are currently tagged Major. All are Help & guidance type. The root cause is the same in each case: architectural assumptions embedded in the product that are never surfaced as onboarding or prerequisite guidance.

> **Scope note — #5:** Issue #5 ("Entire DI connection setup assumes Windows") is the broadest issue in the set. It overlaps with #1, #3, #6, and #17, all of which are also Mac-surface failures. #5 should be treated as an epic-level meta-finding, with #1, #3, #6, and #17 as its component sub-issues. It should not be tracked as a discrete atomic issue — it cannot be assigned to a single component owner or sprint at its current scope. See Required Corrections in critic-review-1.md.

### Pattern 3: Connection state feedback deficit

The product fails to answer the user's primary trust-building question — "is my connection working?" — at every touchpoint in the setup flow. Five issues and one unlogged branch node all describe failure at this moment.

Evidence:
- **#2** — "SQL command execution timed out" gives no indication of which component failed; user cannot determine if the connection is at fault
- **#3** — SQLite connection string accepted without feedback; fails silently at test time because path was wrong
- **#4** — "Use caching database" checkbox gives no feedback on existence or failure
- **#6** — Test Connection runs server-side with no indication; timeout always misinterpreted as a local connection failure
- **#13** — SQL Editor opens showing "Unknown Connection"; Run SQL button disabled with no tooltip

Together these five issues span the DI Connections form and the SQL Editor, but they all converge on the same user need: confirm that the connection is valid before proceeding. The product provides no reliable confirmation at any of these touchpoints.

---

## 3. Most Affected Flow Steps

| Flow step | Issues | Major count |
|---|---|---|
| DI connection setup (ODBC + SQLite) | #1, #2, #3, #4, #5, #6 | 6 |
| Launcher install and TBox execution | #8, #9, #10, #11, #12, #14 | 6 |
| SQL Editor state | #13 | 1 (note: #2 counted above) |
| DI executor first run | #17 | 1 |

DI connection setup and Launcher are tied at 6 issues each, all Major. Both are prerequisite gates with no workaround path. The SQL Editor state (#13) is a persistent issue — it is first encountered in Phase 1 and recurs whenever the connection is re-opened in later phases.

---

## 4. Field Consistency Check

### Tagging resolved

- **#12** canonical tag updated to "Launcher / Builder" (previously listed under both "Launcher" and "Launcher / Builder"). The failure appears in the Builder UI after a Launcher-triggered event — the compound tag is the correct one.

- **#13** feature area "Data Integrity / Builder" revised to "Builder — SQL Editor" for consistency with screen/component column and to clarify it is a Builder-side failure, not a DI-module-side failure.

### Severity questions

- **#10** (Launcher installs extension silently) is tagged Major. The issue description frames the experience as alarming ("First indication is Edge's own security alert") but the alert is issued by Edge, not by a Tosca failure state. The experience is non-blocking on its own — it becomes blocking only when combined with C3 (exe blocked). A downgrade to Minor is defensible; a decision is required. This document retains Major pending a formal triage decision, but the question should not remain open across multiple analysis iterations.

- **#5** (Entire DI setup assumes Windows) is Major and appropriate in severity, but the description spans multiple discrete failures. See Pattern 2 scope note above.

- **#15** (Agents page shows cloud agent with no type indication) is Minor. The severity may be underreported if DI modules silently fail when run on a cloud agent — this dependency is not confirmed in the raw data. Retain Minor pending verification.

### Missing visual cue field

The issue table as ingested has no "Visual Cue" column. If the original Confluence triage table includes a screenshot reference field, it is absent from the ingested data. This is a data gap for all 17 issues.

### Issues with thin descriptions

- **#7** (Search returns no results when querying by Last modified by) — Minor, Usability. Description does not state whether the search field label indicates it is a name-only search. If the label already says "Search by name," the issue severity may be lower than Minor.

---

## 5. Cross-reference with Journey — Undocumented Friction Gaps

The following FigJam branch nodes have no corresponding Confluence issue number. These represent friction that was experienced and mapped but not formally logged as a discrete UX issue.

| Branch node | Description | Phase | Priority | Why it matters |
|---|---|---|---|---|
| D1 | "Exactly 1 source must be defined" error | Phase 3/4 | High | Explicit error message with no documentation path — strong candidate for Feedback/Major issue |
| D2 | "Expected [Select] Actual: Input" error | Phase 3/4 | High | Contradicts D1; two conflicting errors in the same Action mode loop. Note: D1 and D2 may share a root cause — validate before logging as separate issues |
| D3 | No docs on correct Action mode for CSV | Phase 3/4 | High | Absence of documentation — strong candidate for Help & guidance/Major issue |
| E3 | Executor expects no .csv extension in Filename field | Phase 5 | Medium | Counter-intuitive UI requirement; directly caused a test failure requiring expert consultation |
| E2 | File blocked by Windows policy (originated on another machine) | Phase 5 | Medium | Security policy interaction with file provenance — affects reproducibility for any user sharing test assets |
| B2 | Test Connection times out (server-side, SQLite) | Phase 2 | Low | Functionally identical to #6 (same server-side behaviour, SQLite connection type) — warrant a merge note on #6 rather than a separate issue |

**Summary:** 6 branch nodes have no corresponding logged issue. The entire Branch D (D1, D2, D3) is unrepresented in the Confluence table despite being a documented multi-step failure loop. Branch D gaps are the highest priority for new issue creation in the next iteration. D1 and D2 must be validated as same or different root causes before separate issues are filed.
