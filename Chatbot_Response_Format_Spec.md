# Chatbot Response Format Rules: Table vs. Filterable HTML Dashboard

## Step 1 — Classify Query Scope Before Answering

Classify the question along four dimensions:

| Dimension | Narrow → Table | Broad → Dashboard |
|---|---|---|
| Project count | Single project, or small named set (≤3) | Multiple/unspecified, "portfolio", "all programs" |
| Phase scope | One phase named (e.g. Phase 1) | Multiple phases, unspecified, "across phases" |
| Modality scope | One modality named | Multiple/unspecified, "across modalities" |
| Metric count | One metric asked | Multiple/comparative, or open-ended ("summarize") |

**Rule:** ALL narrow → table. ANY broad/unspecified → dashboard candidate (go to Step 2).

## Step 2 — Handle Ambiguity Before Generating

If scope is broad but not explicit, ask one clarifying question before building anything. Do not silently default to "all data."

Example prompts:
- "Do you want this across all phases, or a specific one?"
- "Should I include all modalities, or just [X]?"
- "This spans multiple programs — full portfolio or a subset?"

Proceed only after scope is confirmed, or user says "everything"/"whole portfolio."

## Step 3 — Output Logic

**Table when:** single project + single phase + single modality + single metric. Simple markdown/plain table, no filters/chart, near-real-time return.

**Dashboard when:** multiple projects/phases/modalities in scope, or a comparative/summary question ("which project has the longest...", "how does X compare across..."). Output is HTML per component structure below.

## Step 4 — Guardrails Against Over-Engineering

- Reuse a fixed HTML/JS template — re-populate data, don't regenerate bespoke HTML per query (fixes slow generation).
- Cap complexity: only pre-approved visual components (bar chart, table-with-filters, deviation-from-standard chart) — no invented chart types per query.
- A "broad" query with genuinely small output (e.g. 4 rows) can downgrade to table — rule is about information density, not literal keywords.

## Step 5 — Dashboard Component Structure

Based on the existing "CMC Task Duration Detail & Var." dashboard. A dashboard = one HTML page, three stacked components:

### 1. Filter bar (always present)
- Modality dropdown
- Campaign/task type dropdown (e.g. DS Campaign)
- Site dropdown
- Standard/benchmark reference selector (e.g. "Standard MSD ± interval")
- Free-text task search
- Toggle checkboxes for edge cases ("only decompressed kickoffs", "only wells with multiple standards")

### 2. Chart
- Quarter/time-bucketed bar chart for trend/forecast queries (e.g. kickoff count by start quarter)
- Omit if query is purely comparative/ranking with no time dimension
- Include legend key (quarter, current quarter highlighted, forecast bucket)

### 3. Task-level detail table (always included below chart)
- Columns: Program, Phase, Function, Modality, Start (QF), Start (approx.), End (approx.), Actual (full), Std (full)
- Derived variance columns: vs. Std (avg), vs. Std (annotated) — ratio + annotated text
- Pace, Logged as, Classification, Grant/task description
- Conditional formatting: ahead of pace = green, behind = red
- Sortable/filterable in sync with filter bar

### Key rule
Chart = summary lens, table = source of truth. Never return a chart without the backing detail table — a bar chart alone lacks context.

## Step 6 — Test Cases

| Query | Classification | Output |
|---|---|---|
| "Which project in small molecule Phase 1 has the longest DS campaign?" | Narrow | Table |
| "Which project in small molecule has the longest DS campaign?" | Broad (all phases) | Dashboard |
| "How many DS/DTK campaigns anticipated in next two years?" | Broad, forecast | Dashboard + quarterly chart + annotation |
| "Summarize the portfolio" | Broad, ambiguous | Clarify first |
| "DS campaign duration for Project X?" | Narrow | Table |

## Related Requirements (for context, not in scope of this doc)

- **Benchmark/deviation logic:** pull archetype standards by modality + phase; calculate and surface variance from standard.
- **Forecasting logic:** quarter-bucketed campaign counts by start date, portfolio-wide, across modalities.
- **Access-aware data presentation:** only surface data the requesting user has permission to see, evaluated per user.
