# Tosca Cloud UX Dogfooding — Self-Analysis

Self-analysis of a Tosca Cloud Data Integrity dogfooding exercise, produced by a multi-agent Claude Code pipeline as part of a UX adoption challenge set by Peter Muka (Tricentis UX team).

## What this is

A three-agent pipeline (Ingester → Analyst → Critic) reads the raw dogfooding artifacts — a FigJam experience map and a Confluence UX issues table — and produces structured analysis outputs without manual authoring of analysis content.

**Source artifacts:**
- FigJam experience map: 19 journey nodes across 5 phases, 5 friction branches (A–E), 17 branch sub-nodes
- Confluence issue repository: 17 UX issues logged during the dogfooding session

## Output files

| File | Agent | Description |
|------|-------|-------------|
| `ingester-dump.md` | Ingester | Raw structured dump of FigJam and Confluence data — no interpretation |
| `journey-analysis.md` | Analyst + Critic | Flow overview, friction map per phase, severity concentration, blocking/non-blocking classification |
| `issues-analysis.md` | Analyst + Critic | Issue distribution, top patterns, most affected steps, field consistency gaps |

## Supporting files

| File | Description |
|------|-------------|
| `agent-log.md` | Full pipeline record: model, input, output summary, iteration count per run |
| `critic-review-1.md` | Critic's corrections from iteration 1 — classification errors, scope issues, structural fixes |

## Pipeline

```
Ingester → ingester-dump.md
                ↓
           Analyst → journey-analysis.md v1
                     issues-analysis.md v1
                          ↓
                     Critic → corrections → v2 outputs
                     (loop ran 1 of 5 iterations; accepted)
```

All agents: `claude-sonnet-4-6`. No analysis sentences were written manually.

## Context

- **Dogfooder:** Franc González, Senior UX Designer, Tricentis
- **Module tested:** Tosca Cloud — Data Integrity, Complete Row by Row Comparison
- **Exercise set by:** Peter Muka
- **Deadline:** July 1, 2026
