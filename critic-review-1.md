# Critic Review — Iteration 1

Reviewed files: `journey-analysis.md`, `issues-analysis.md`
Source of truth: `ingester-dump.md`

---

## 1. Claims Requiring Evidence

**Claim: "No phase in the first four thirds of the journey is resolved by the product itself."**
(journey-analysis.md, Section 1)
Partially unsupported. Node 10 ("TBox unblocked ✅ First execution") is listed in Phase 3/4 in the raw dump. The resolution of the TBox block appears to have involved IT/Zscaler intervention, which is external — but the journey analysis never states this explicitly. The claim is likely correct but not justified. The analysis must state what unblocked node 10 (IT exclusion, Zscaler whitelist, etc.) or remove the implicit claim that node 10 was self-resolved.

**Claim: "12 of the 14 Major issues" come from the two highest-concentration clusters.**
(journey-analysis.md, Section 3)
The table in Section 3 lists: DI connection setup (#1, #2, #3, #4, #5, #6 = 6 Major) + Launcher (#8, #9, #10, #11, #12, #14 = 6 Major) = 12. But the totals column also lists #2 and #13 under "SQL Editor state" and #17 under "First DI test run." Issues #2 and #13 are already counted in the first cluster (#2) and the standalone SQL Editor row. The body text claim of "12 of 14" is numerically correct but the table implies double-counting of #2. The table should not list #2 under both "DI connection setup" and "SQL Editor state" without a note.

**Claim (issues-analysis.md, Pattern 2): "This pattern accounts for 6 issues (all Help & guidance type; all but #10 are Major)."**
#10 is tagged Major in the raw issue table (ingester-dump.md row 10: Severity = Major). The parenthetical "all but #10 are Major" is wrong. #10 is Major. The Analyst then says in Section 4 that #10 is "the weakest Major" and could be downgraded — but that is a recommendation, not the current data state. The description in Pattern 2 must not treat #10 as if it were already Minor.

**Claim: "B2 overlaps with A3 (same server-side Test Connection behaviour, different connection type)."**
(journey-analysis.md, Section 2, Phase 2)
This is a valid inference but it is asserted without citation. The raw dump confirms B2 description ("Test Connection times out (server-side)") and A3 ("Test Connection runs server-side (never stated)"). The inference is correct; the analysis should state it is inferred from matching descriptions, not independently evidenced.

---

## 2. Classification Errors

**#10 — "Help & guidance / Major" is defensible but borderline**
The raw description reads: "Launcher installs browser extension with no prior warning. First indication is Edge's own security alert." This is not a silent blocker — the user receives a signal (the Edge alert). The experience is alarming but not execution-blocking on its own. The classification as Major is the weakest in the set. The case for downgrading to Minor is real. However, no correction is required at this stage unless the Analyst decides to revise severity — the current classification is within range. Flag only: the issues-analysis.md Severity questions section already identifies this. No change needed, but the Analyst should make a decision and not leave it open in both Pattern 2 and Section 4 simultaneously.

**#5 — scoped as a single "issue" but describes a cross-cutting platform finding**
(issues-analysis.md, Section 4, Severity questions)
The Analyst correctly identifies this but does not act on it. #5's description ("Entire DI connection setup assumes Windows. Mac users hit multiple silent blockers with no in-product guidance.") conflates multiple discrete failure points into one issue. It overlaps with #1, #3, #6, and #17 — all of which are also Mac-surface failures with no guidance. Leaving #5 as a standalone issue while the overlapping issues remain separate creates tracking ambiguity. Required: either note explicitly that #5 is a meta-finding / epic-level issue, or split it into the sub-issues already captured in #1, #3, etc. and close #5.

**#13 — classified as "Feedback" but the primary failure is missing affordance**
The raw description: "SQL Editor shows 'Unknown Connection' with no explanation. Run SQL button disabled with no tooltip or guidance." The disabled button with no tooltip is a Help & guidance failure, not a Feedback failure. "Feedback" issues in the existing taxonomy describe cases where the system communicates something but communicates it wrong. Here the system communicates nothing (no tooltip, no explanation). This should be reclassified as Help & guidance or given a split note. The current classification is an error.

---

## 3. Missing Cross-References

**Branch D (D1, D2, D3) — entirely absent from issues-analysis.md Sections 1–3**
The journey-analysis.md Friction Map (Phase 3) correctly lists D1, D2, D3 with "(no direct issue)" notes. However, issues-analysis.md Section 2 (Top 3 Recurring Patterns) and Section 3 (Most Affected Flow Steps) make no mention of Branch D at all. Pattern 1 (silent failure) is a direct match for D1 and D2 — contradictory errors that give no resolution path — but neither D-branch node is cited as evidence. The cross-reference gap table in Section 5 of issues-analysis.md does list D1–D3, but they are buried after all other analysis and never appear in the patterns that would benefit most from their evidence.

**#17 linked to node 15 in journey-analysis.md but E1 branch is described at both nodes 10–11 and 14–15**
The raw dump places E1 under "Branch E — File/executor blockers (from nodes 15–17)". The journey-analysis.md severity concentration table lists #17 under "First DI test run (nodes 10–11)" — which is wrong. Node 10–11 is the CSV Action mode loop. The executor folder error (#17) is a Phase 5 discovery at node 15. The node range cited in the severity table is incorrect.

**#9 (Chrome extension UPDATE_DISABLED) — not cross-referenced to any branch node in journey-analysis.md Phase 3 table**
The Phase 3 Friction Map table maps C2 to #9 correctly. But the narrative below the table ("Nodes 09–11 represent the pivot to CSV after the ODBC/SQLite path failed") refers to journey nodes 09–11, not branch nodes. This creates a numbering collision: branch node C2 maps to issue #9, but the narrative uses "node 09" to mean the journey node "Pivot to CSV source." The Analyst must disambiguate branch node IDs (letter+number) from journey node IDs (number only) throughout all narrative text.

---

## 4. Scope Issues

**#5 is too broad to be actionable as a discrete UX issue.**
Already noted in Classification Errors. At its current scope, #5 cannot be assigned to a single team, component owner, or sprint. It describes a systemic product assumption, not a discrete interaction failure. As written it is an analysis finding, not a ticket. It should be either converted to an epic-level finding with the discrete sub-issues (#1, #3, #6, #17) as children, or closed in favour of those sub-issues.

**Pattern 3 (Test Connection / connection state ambiguity) is too narrow as named.**
The pattern bundles #6, #4, and #13. But the binding logic — "is my connection working?" — also applies to #2 (SQL timeout with no root cause) and #3 (wrong path with no feedback). The pattern name and scope artificially exclude these. The pattern should either be broadened to "Connection state feedback deficit" and include #2 and #3, or the rationale for excluding them must be stated.

**D1 and D2 could be one issue or two — the dump does not resolve this.**
The Branch D sticky note says "Errors contradict each other." If D1 ("Exactly 1 source must be defined") and D2 ("Expected [Select] Actual: Input") are produced by the same Action mode misconfiguration at different attempts, they may be the same root cause. If they come from different configurations, they are separate issues. The Analyst must determine which before logging them as new issues in the next iteration.

---

## 5. Confirmed Findings

The following are well-supported, correctly classified, and should be preserved:

- **Pattern 1 (Silent failure with no root cause)** — all 6 cited issues (#2, #6, #8, #11, #12, #17) are correctly identified, accurately described, and consistently classified as Feedback/Major. The structural observation ("the message stops at symptom identification") is supported by the raw descriptions.
- **Phase 5 analysis (Resolution Path)** — the observation that every Phase 5 fix was externally discovered is correct and directly evidenced by the node labels (node 14: "Sync with Michael", node 15: "Caching folder missing — created", etc.). The conclusion that "the path to correct configuration is entirely invisible to a new user without expert consultation" follows directly from the data.
- **The Section 5 cross-reference gap table in issues-analysis.md** — all 6 unlogged branch nodes (B2, D1, D2, D3, E2, E3) are correctly identified as having no corresponding Confluence issue. The priority ranking (D-branch highest) is well-reasoned.
- **Feature area distribution (DI = 6, Launcher = 6)** is accurate against the raw issue table.
- **#14 classification as Help & guidance / Major** — the framing that this is a product gap (no in-product guidance about corporate policy blockers) rather than an IT issue is correct.
- **Journey arc summary** ("attempted self-service → repeated silent failure → forced external consultation → resolution → success") maps accurately to the node sequence in the raw dump.

---

## 6. Required Corrections

1. **Clarify what resolved node 10 ("TBox unblocked").** The narrative currently implies no product-side resolution occurred anywhere in phases 1–4, but node 10 is a success state. State explicitly what intervention (IT/Zscaler exclusion, user action, etc.) unblocked it, or note that this was also an external resolution.

2. **Fix the double-counting of #2 in journey-analysis.md Section 3 severity table.** Remove #2 from the "SQL Editor state" row or add a "(also counted above)" note. The claim "12 of 14 Major issues" must be arithmetically traceable from the table.

3. **Correct the Pattern 2 parenthetical in issues-analysis.md.** Change "all but #10 are Major" to "all are currently tagged Major" — and then either retain #10 as Major or formally downgrade it with a rationale. Do not leave the severity question open in both Pattern 2 and Section 4 simultaneously.

4. **Reclassify #13 from Feedback to Help & guidance.** The disabled Run SQL button with no tooltip is a missing affordance, not a wrong message. Update the issue type in the distribution tables accordingly (Feedback drops to 6, Help & guidance rises to 8).

5. **Correct the node range for #17 in journey-analysis.md Section 3.** Change "First DI test run (nodes 10–11)" to "First DI test run (node 15 / Phase 5)" to match the raw dump.

6. **Disambiguate branch node IDs from journey node IDs in all narrative text.** Use "journey node N" and "branch node XN" consistently. The Phase 3 narrative currently uses "nodes 09–11" to mean journey nodes, which collides with branch node numbering.

7. **Resolve the scope of #5.** Choose one: (a) mark #5 as an epic-level meta-finding and note that #1, #3, #6, #17 are its component issues; or (b) close #5 and distribute its content into the existing sub-issues. Do not leave it as a standalone atomic issue at current scope.

8. **Add D1 and D2 cross-references to Pattern 1 in issues-analysis.md.** Both involve error messages that provide no resolution path. They are stronger evidence of Pattern 1 than several currently cited issues. Add them as un-logged examples with a note that they are candidates for new issues.

9. **Expand Pattern 3 scope or justify the exclusions.** Either rename to "Connection state feedback deficit" and add #2 and #3 as evidence, or explicitly state why they are excluded from this pattern.

10. **Add a note to the Branch D gap analysis** indicating that D1 and D2 may share a root cause and must be validated before logging as separate issues. This is a prerequisite for the next iteration's issue-creation step.
