---
name: configuration-comparison
description: >
  Compare two or more OAS analysis runs side by side with quantitative deltas.
  Use when the user wants to compare results, see what changed between runs,
  evaluate before-vs-after optimization, or compare different wing designs.
  Triggers on: "compare these runs", "what changed", "before vs after",
  "side by side comparison", "which design is better", "compare results",
  "delta between runs".
argument-hint: "[run_ids]  e.g. 'compare run1 vs run2' or 'last two runs'"
---

# Configuration Comparison

Compare two or more OAS analysis runs with a quantitative side-by-side table.

## Workflow

### 1. Identify Runs

Accept any of:
- Two explicit run_ids from `$ARGUMENTS`
- "last two runs" → call `list_artifacts()` and use the two most recent
- "before and after" → use runs from before and after an optimization

### 2. Retrieve Data

For each run, call in parallel:
```
get_artifact(run_id="<run_id_1>")
get_artifact(run_id="<run_id_2>")
```

Extract `metadata.analysis_type`, `results`, and `metadata.parameters` from each.

### 3. Build Comparison Table

| Metric | Run 1 | Run 2 | Delta | Delta % |
|---|---|---|---|---|
| CL | ... | ... | ... | ... |
| CD | ... | ... | ... | ... |
| L/D | ... | ... | ... | ... |
| CM | ... | ... | ... | ... |
| fuelburn (kg) | ... | ... | ... | ... |
| structural_mass (kg) | ... | ... | ... | ... |
| failure | ... | ... | ... | ... |

**Highlight rows with >5% change** using bold or a marker.

Only include metrics that exist in both runs (aero runs won't have structural_mass).

### 4. Compare Design Variables

If both runs are optimization results, compare `results.optimized_design_variables`:
- Which DVs changed the most?
- Did any DV hit its bound?

### 5. Qualitative Comparison

For deeper insight, call `get_detailed_results(run_id, "standard")` for each run and note:
- Whether the lift distribution became more/less elliptical
- Whether the stress distribution changed significantly
- Any shift in the drag breakdown (CDi vs CDv vs CDw)

### 6. Summarize

Write a 3-5 sentence summary: what changed, by how much, and what it means for the design. Make a recommendation.

```
log_decision(
    decision_type="result_interpretation",
    reasoning="<summary and recommendation>",
    selected_action="<recommended design choice>"
)
```

### 7. Export

```
export_session_graph(output_path="comparison_provenance.json")
```
