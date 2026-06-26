# Ingester Dump — Tosca DI Dogfooding UX Analysis

## 1. Journey Nodes

| # | Node | Phase |
|---|---|---|
| 01 | Start: DI chosen as primary feature | PHASE 1: Setup & ODBC |
| 02 | Navigate to DI Connections | PHASE 1: Setup & ODBC |
| 03 | Create ODBC connection | PHASE 1: Setup & ODBC |
| 04 | SQL Editor: Run SQL blocked | PHASE 1: Setup & ODBC |
| 05 | Consult tester (Michael J.) | PHASE 2: SQLite |
| 06 | Create SQLite connection | PHASE 2: SQLite |
| 07 | Install Tricentis Launcher | PHASE 3: Launcher & CSV |
| 08 | Run test: TBox blocked | PHASE 3: Launcher & CSV |
| 09 | Pivot to CSV source | PHASE 3: Launcher & CSV |
| 10 | TBox unblocked ✅ First execution | PHASE 3: Launcher & CSV / PHASE 4: Current state |
| 11 | CSV Action mode loop — failing | PHASE 3: Launcher & CSV / PHASE 4: Current state |
| 12 | 16 UX issues logged ✅ | PHASE 4: Current state |
| 13 | Next: ask Michael for CSV config | PHASE 4: Current state |
| 14 | Sync with Michael — Root cause found | PHASE 5: Resolution |
| 15 | Caching folder missing — created | PHASE 5: Resolution |
| 16 | File blocked by Windows — Unblocked | PHASE 5: Resolution |
| 17 | Filename: no .csv extension needed | PHASE 5: Resolution |
| 18 | Test case passes ✅ 100 rows, Delta: 0 | PHASE 5: Resolution |
| 19 | Playlist passes ✅ Exercise complete | PHASE 5: Resolution |

## 2. Friction Branches

### Branch A — ODBC blockers (from node 03)

| ID | Description |
|---|---|
| A1 | No "ready to use" guidance in-product |
| A2 | SQL timeout: no root cause shown |
| A3 | Test Connection runs server-side (never stated) |

Sticky: No in-product hint that a working DB connection with accessible tables is required before setup.

---

### Branch B — SQLite blockers (from node 06)

| ID | Description |
|---|---|
| B1 | No file path guidance (agent vs local Mac) |
| B2 | Test Connection times out (server-side) |
| B3 | "Use caching DB" no feedback on failure |
| B4 | Mac assumed to be Windows |

Sticky: SQLite file must be placed on the Windows VM agent. This is never documented or communicated in-product.

---

### Branch C — Launcher/antivirus (from nodes 07–08)

| ID | Description |
|---|---|
| C1 | TBox quarantined by Zscaler |
| C2 | Chrome extension fails — no fallback suggested |
| C3 | Edge extension disconnects (exe blocked) |
| C4 | Silent test failure: no error in Builder |
| C5 | Windows Security locked by IT |

Sticky: Launcher is a hard prerequisite for any test execution. This is not communicated before starting.
Sticky: Switching module or connection type won't help — TBox is the agent blocker, not the data config.

---

### Branch D — CSV Action mode loop (from node 11)

| ID | Description |
|---|---|
| D1 | "Exactly 1 source must be defined" |
| D2 | "Expected [Select] Actual: Input" error |
| D3 | No docs on correct Action mode for CSV |

Sticky: Multiple Action mode combinations tried. Errors contradict each other. No documentation exists for the correct CSV Local source configuration.

---

### Branch E — File/executor blockers (from nodes 15–17)

| ID | Description |
|---|---|
| E1 | ToscaDataIntegrityExecutor folder not created on first run |
| E2 | File from another machine — blocked by Windows policy |
| E3 | Executor expects no file extension in Filename field |

Stickies: Each has an explanatory note about the missing guidance.

---

## 3. Issue Table

| # | Environment | Feature | Screen/Component | Flow/Step | Issue Type | Severity | Description |
|---|---|---|---|---|---|---|---|
| 1 | Dev | Data Integrity | Builder - SQL Editor | Builder → add module → set Connection → Edit SQL → Run SQL | Help & guidance | Major | No in-product guidance explaining what data setup is needed before the module can be used. |
| 2 | Dev | Data Integrity | Builder - SQL Editor | Builder → add module → set Connection → Edit SQL → Run SQL | Feedback | Major | "SQL command execution timed out" gives no indication of whether the problem is the query, the connection, or the server. |
| 3 | Dev | Data Integrity | DI Connections - Create connection form - SQLite | DI Connections → Create → SQLite → enter connection string | Help & guidance | Major | No indication that the SQLite file path must be on the agent machine. Mac users will naturally try a local path with no feedback. |
| 4 | Dev | Data Integrity | DI Connections - Create connection form - SQLite | DI Connections → Create → SQLite → Use caching database | Feedback | Major | "Use caching database" checkbox gives no feedback on what it points to, whether it exists, or why it fails. |
| 5 | Dev | Data Integrity | DI Connections - Create connection form | Full DI connection setup flow on Mac | Help & guidance | Major | Entire DI connection setup assumes Windows. Mac users hit multiple silent blockers with no in-product guidance. |
| 6 | Dev | Data Integrity | DI Connections — Test Connection button | DI Connections → open connection → Test Connection | Feedback | Major | "Test Connection" runs server-side but gives no indication of this. Users placing SQLite file on VM agent always get timeout. |
| 7 | Dev | Data Integrity / Builder | All Assets — Test Cases search | All Assets → Test Cases → search by name | Usability | Minor | Search returns no results when querying by "Last modified by". Only works by "Name". No indication of which field is searched. |
| 8 | Dev | Launcher | Tricentis Launcher — component installation | Run test → Launcher opens → installs components | Feedback | Major | TBox quarantined by antivirus: UI shows only "Component executable not found". Actual cause visible only in log file. |
| 9 | Dev | Launcher | Launcher — browser extension install | Run test → install browser extension → Chrome | Feedback | Major | Chrome extension fails with UPDATE_DISABLED. No explanation, no suggestion to try a different browser. |
| 10 | Dev | Launcher | Launcher — browser extension install (Edge) | Run test → Launcher silently installs Edge extension | Help & guidance | Major | Launcher installs browser extension with no prior warning. First indication is Edge's own security alert. |
| 11 | Dev | Launcher | Tosca browser extension — Disconnected state | Run test → extension installed → Tosca status: Disconnected | Feedback | Major | TricentisHtmlEngineExtensionHelper.exe blocked by antivirus. Error names the file but provides no actionable steps. |
| 12 | Dev | Launcher / Builder | Builder — after failed run | Run test → extension blocked → returns to Builder | Feedback | Major | When browser extension fails to connect to agent, test run silently fails. User lands back on Builder with no error. |
| 13 | Dev | Builder — SQL Editor | SQL Editor — connection state | Builder → SQL Statement → Edit SQL → SQL Editor opens | Feedback | Major | SQL Editor shows "Unknown Connection" with no explanation. Run SQL button disabled with no tooltip or guidance. |
| 14 | Dev | Launcher | Windows Security — Virus & threat protection | Attempt to add antivirus exclusion for Tricentis Launcher | Help & guidance | Major | Windows Security settings IT-managed and locked. No in-product guidance that corporate security policies may block execution. |
| 15 | Dev | Agents / Run tests | Agents page — cloud agent | Run tests → select agent → cloud agent available | Help & guidance | Minor | Agents page shows cloud agent but no indication of which test types it supports. DI modules not mentioned. |
| 16 | Dev | Playlists / Run tests | Playlist — Test run dropdown | Playlist → Run → Test run dropdown | Usability | Minor | Test run dropdown only shows "Run on personal agent" and "Schedule run". No option to select a specific agent. |
| 17 | Dev | Data Integrity / Launcher | Tosca DI Executor — first run setup | Run test case with DI module for the first time | Help & guidance | Major | ToscaDataIntegrityExecutor folder not created automatically. Error names the path but provides no instruction to create it. |

---

## 4. Quick Counts

### By Severity

| Severity | Count |
|---|---|
| Major | 14 |
| Minor | 3 |
| **Total** | **17** |

### By Issue Type

| Issue Type | Count |
|---|---|
| Feedback | 7 |
| Help & guidance | 7 |
| Usability | 3 |
| **Total** | **17** |

### By Feature Area

| Feature Area | Count |
|---|---|
| Data Integrity | 6 |
| Launcher | 6 |
| Data Integrity / Builder | 1 |
| Launcher / Builder | 1 |
| Builder — SQL Editor | 1 |
| Data Integrity / Launcher | 1 |
| Agents / Run tests | 1 |
| **Total** | **17** |
