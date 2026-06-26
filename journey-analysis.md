# Journey Analysis — Tosca DI Dogfooding
_Iteration 2 — revised per critic-review-1.md_

## 1. Flow Overview

The journey spans 5 phases across 19 nodes, tracing a single user's attempt to use the Data Integrity feature from first contact to passing test execution. Phases 1–3 are dominated by serial blockers: ODBC fails silently, SQLite silently assumes a Windows agent, and the Launcher is quarantined before the user can run a single test. Phase 4 represents a plateau state where the user is unblocked enough to execute tests but is trapped in an Action mode configuration loop with no documentation. Phase 5 is a rapid resolution arc triggered entirely by external consultation (Michael J.), not by in-product affordances.

Node 10 ("TBox unblocked ✅ First execution") is a success state within Phase 3/4, but the unblocking required an IT/Zscaler exclusion — an external intervention, not a product-side affordance. No phase in the journey is resolved by the product itself.

The overall arc is: attempted self-service → repeated silent failure → forced external consultation → resolution → success.

---

## 2. Friction Map

> Notation: "Branch node XN" refers to friction branch IDs from the FigJam (letter + number, e.g. A1, B2). "Journey node N" refers to the numbered flow steps (01–19). These are distinct identifiers.

### Phase 1: Setup & ODBC (journey nodes 01–04)

| Branch node | Description | Linked issues |
|---|---|---|
| A1 | No "ready to use" guidance in-product | #1 |
| A2 | SQL timeout: no root cause shown | #2 |
| A3 | Test Connection runs server-side (never stated) | #6 |

Journey node 04 ("SQL Editor: Run SQL blocked") is the first hard stop. The user cannot continue until an ODBC-compatible DB is accessible — a prerequisite never communicated. Issues #1, #2, and #6 cluster here. Issue #13 (SQL Editor shows "Unknown Connection") is also triggered at this step but logged as a Builder–SQL Editor issue rather than a connection setup issue.

### Phase 2: SQLite (journey nodes 05–06)

| Branch node | Description | Linked issues |
|---|---|---|
| B1 | No file path guidance (agent vs local Mac) | #3 |
| B2 | Test Connection times out (server-side, SQLite) | #6 (inferred — same behaviour as A3, different connection type) |
| B3 | "Use caching DB" no feedback on failure | #4 |
| B4 | Mac assumed to be Windows | #5 |

Journey node 05 requires external consultation before journey node 06 can even be attempted, indicating the product provided no path forward. Branch B4 has the widest blast radius — issue #5 flags the entire DI connection setup flow as Windows-only with no Mac guidance. Branch B2 overlaps with A3: both describe a server-side Test Connection that times out with no indication it is running remotely; this is inferred from the matching descriptions in the raw dump, not independently evidenced.

### Phase 3: Launcher & CSV (journey nodes 07–11)

| Branch node | Description | Linked issues |
|---|---|---|
| C1 | TBox quarantined by Zscaler | #8 |
| C2 | Chrome extension fails — no fallback suggested | #9 |
| C3 | Edge extension disconnects (exe blocked) | #11 |
| C4 | Silent test failure: no error in Builder | #12 |
| C5 | Windows Security locked by IT | #14 |
| D1 | "Exactly 1 source must be defined" | (not logged — new issue candidate) |
| D2 | "Expected [Select] Actual: Input" error | (not logged — new issue candidate; may share root cause with D1) |
| D3 | No docs on correct Action mode for CSV | (not logged — new issue candidate) |

This phase carries the highest friction density. Journey node 08 ("Run test: TBox blocked") is a hard block — no workaround exists within the product. Journey node 10 ("TBox unblocked") was resolved by an IT/Zscaler exclusion, not an in-product action. Journey nodes 09–11 represent the pivot to CSV after the ODBC/SQLite path failed; the CSV path then introduces Branch D's configuration loop.

Issue #10 (Launcher installs Edge extension silently) belongs to the journey node 07–08 transition. Branch node C2 maps to issue #9; note that "journey node 09" (Pivot to CSV source) and "branch node C2" are different identifiers — C2 describes a friction point within the Launcher flow, not the CSV pivot.

### Phase 4: Current state (journey nodes 12–13)

| Branch node | Description | Linked issues |
|---|---|---|
| D1–D3 | CSV Action mode loop | None logged directly (see Branch D above) |

Journey node 12 ("16 UX issues logged") marks where the user stopped trying to self-resolve and began systematic issue capture. Journey node 13 ("Next: ask Michael for CSV config") is another external consultation trigger — the product offered no resolution path for the Action mode loop.

### Phase 5: Resolution (journey nodes 14–19)

| Branch node | Description | Linked issues |
|---|---|---|
| E1 | ToscaDataIntegrityExecutor folder not created on first run | #17 |
| E2 | File blocked by Windows policy (originated on another machine) | (not logged — new issue candidate) |
| E3 | Executor expects no .csv extension in Filename field | (not logged — new issue candidate) |

See Resolution Path section below.

---

## 3. Severity Concentration

The following flow steps concentrate the highest density of Major-severity issues:

| Flow step | Journey nodes | Major issues |
|---|---|---|
| DI connection setup (ODBC + SQLite) | 02–06 | #1, #2, #3, #4, #5, #6 — 6 Major |
| Launcher installation and TBox execution | 07–08 | #8, #9, #10, #11, #12, #14 — 6 Major |
| SQL Editor state and run | 04, 13 | #13 — 1 Major (note: #2 is already counted in DI connection setup above) |
| First DI test run (executor setup) | 15 / Phase 5 | #17 — 1 Major |

The two highest-concentration clusters — DI connection setup and Launcher — together account for 12 of the 14 Major issues. Both are prerequisite gates: neither can be bypassed, and both produce only silent failures or opaque error messages. The remaining 2 Major issues (#13, #17) are each isolated single-touchpoint failures.

---

## 4. Blocking vs. Non-blocking Friction

### Blocking (execution stopped entirely)

| Issue | Branch node | Flow step |
|---|---|---|
| #1 — No prerequisite guidance | A1 | Could not use SQL Editor at all |
| #2 — SQL timeout no root cause | A2 | Run SQL disabled / timed out |
| #3 — SQLite path assumes agent | B1 | Connection string always fails on Mac |
| #6 — Test Connection runs server-side | A3/B2 | Connection always times out |
| #8 — TBox quarantined | C1 | Test execution entirely blocked |
| #9 — Chrome extension UPDATE_DISABLED | C2 | No Chrome path forward |
| #11 — Edge exe blocked by antivirus | C3 | Disconnected, no execution |
| #12 — Silent test failure after extension block | C4 | No signal, user stranded |
| #17 — Executor folder missing | E1 | First DI run fails with path error |

### Non-blocking (degraded experience, user could continue)

| Issue | Branch node | Flow step |
|---|---|---|
| #4 — Caching DB no feedback | B3 | Checkbox can be ignored |
| #5 — Mac assumed to be Windows | B4 | User workarounds exist with guidance |
| #7 — Search only works by Name | — | Search still functions partially |
| #10 — Extension installed silently | C3 | Alarming but not execution-blocking on its own |
| #13 — Unknown Connection in SQL Editor | — | Confusing state, workaround possible |
| #14 — Windows Security locked by IT | C5 | External resolution required; not a product bug but product provides no guidance |
| #15 — Cloud agent type unclear | — | User can still select agent |
| #16 — No specific agent selection in Playlist | — | Run still possible on personal agent |

---

## 5. Resolution Path (Phase 5, journey nodes 14–19)

Phase 5 is a 6-node external-resolution arc. Every fix in this phase was discovered through direct consultation with Michael J., not through in-product guidance, documentation, or error messaging:

- **Journey node 14** (Sync with Michael): Root cause identified externally — the ToscaDataIntegrityExecutor folder was never created.
- **Journey node 15** (E1 resolved): Caching folder manually created. Issue #17 captures this as a Major Help & guidance gap — the error names the path but gives no instruction.
- **Journey node 16** (E2 resolved): File blocked by Windows security policy because it originated on another machine. No in-product mention of this scenario. Not yet logged as a discrete issue — gap identified, candidate for new issue.
- **Journey node 17** (E3 resolved): Filename field must not include the .csv extension. This is undocumented and counter-intuitive. Not yet logged — candidate for new issue. Note: D1 and D2 may share a root cause with the same Action mode misconfiguration; this must be validated before logging as separate issues.
- **Journey node 18**: Test case passes, 100 rows, Delta: 0. First successful execution.
- **Journey node 19**: Playlist passes. Exercise complete.

The resolution arc validates that the product works correctly when configured correctly, but the path to correct configuration is entirely invisible to a new user without expert consultation.
