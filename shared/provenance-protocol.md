# Provenance Protocol

Every OAS workflow must follow this provenance protocol to create an auditable record of decisions and analyses.

## Workflow

1. **`start_session(notes="...")`** — call once at the beginning of every workflow. The `notes` field should describe the study goal.
2. **`log_decision(...)`** — call at each decision point (see table below). Always pass `prior_call_id` when the decision is directly informed by a specific tool result (use the `_provenance.call_id` from that result's envelope).
3. **`export_session_graph(session_id=..., output_path="...")`** — call once at the end to save the provenance DAG as JSON.

## Decision Types

| `decision_type` | When to call |
|---|---|
| `mesh_resolution` | After choosing num_x / num_y / wing_type |
| `dv_selection` | Before optimization: which design variables and bounds |
| `constraint_choice` | Before optimization: which constraints and targets |
| `result_interpretation` | After any analysis: summarize what results mean and what to do next |
| `convergence_assessment` | After optimization: did it converge, is the result trustworthy |

## Required Fields

```python
log_decision(
    decision_type="result_interpretation",
    reasoning="CL=0.52, CD=0.028, L/D=18.6 — close to target CL",
    selected_action="proceed to drag polar sweep",
    prior_call_id="<_provenance.call_id from the analysis>",
    confidence="high"   # optional: "high", "medium", "low"
)
```

## Viewing the Graph

- **Browser**: `http://localhost:7654/viewer?session_id=<id>` (viewer starts automatically)
- **Offline**: open `oas_mcp/provenance/viewer/index.html` and drop the exported JSON file onto the page
