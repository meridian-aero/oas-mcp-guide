---
name: workflow-builder
description: >
  Interactively design a multi-step OAS study: architecture comparison,
  parametric sweep, design optimization campaign, or technology assessment.
  Use when the user describes a complex study involving multiple configurations,
  parameter variations, or multi-step analysis. This skill asks structured
  questions to define the study, then generates a plan recommending which
  analysis skills to invoke. Triggers on: "design a study", "plan a study",
  "architecture study", "design of experiments", "compare 4 configurations",
  "find best truss placement", "multi-step analysis", "study plan".
argument-hint: "[study_description]  e.g. 'compare tube-and-wing vs TBW vs BWB at 3 ARs'"
---

# Workflow Builder

Design a multi-step OAS study through interactive Q&A, then generate a study plan that orchestrates the appropriate skills.

This skill **plans** the study — it does not execute analyses directly. After presenting the plan, the user approves and then the recommended skills are invoked.

## Question Flow

### Step 1: Study Goal

Ask: "What's the goal of this study?"

Common goals and the skills they map to:
| Goal | Primary Skill | Supporting Skills |
|---|---|---|
| Compare architectures (TBW vs conventional vs ...) | `architecture-comparison` | `configuration-comparison` |
| Sweep a parameter (AR, taper, sweep, ...) | `parameter-sweep` | `mesh-convergence-study` (first) |
| Compare materials | `material-trade-study` | `configuration-comparison` |
| Optimize a design | `run-aero-opt` or `run-aerostruct-opt` | `compute-drag-polar` (before/after) |
| Multipoint sizing | `run-multipoint-opt` | `configuration-comparison` |
| Technology assessment | `parameter-sweep` + `architecture-comparison` | `export-for-integration` |

### Step 2: Configurations or Parameters

Based on the goal:
- **Architecture comparison**: Which configs? (conventional, TBW, HAR, BWB)
- **Parameter sweep**: Which parameter? What range? How many steps?
- **Optimization**: Which objective? Which DVs? What constraints?
- **Technology assessment**: What technology? How does it change the model?

Suggest specific options based on published use cases:
- AR sweep: 6, 9, 12, 15, 18 (covers regional to ultra-high AR)
- TBW strut placement: 40%, 50%, 60%, 70% span
- Material comparison: aluminum, titanium, composite

### Step 3: Metrics

Ask: "What metrics matter most?"
- Fuel burn (most common for transport)
- Structural mass
- L/D (aerodynamic efficiency)
- Stability (static margin)
- Weight savings (vs baseline)

### Step 4: Fidelity

Ask: "Quick screening or production quality?"

| Level | num_y | Use Case |
|---|---|---|
| Quick screening | 5 | Fast exploration, many configs |
| Standard | 7–9 | Good balance of speed and accuracy |
| Production | 11–15 | Publication-quality results |
| Converged | Run mesh-convergence-study first | When accuracy matters most |

### Step 5: Constraints

Ask: "Any mission constraints?" (optional)
- Range requirement
- Maximum structural mass
- Minimum stability margin
- Maximum failure metric

## Generate Study Plan

Based on answers, produce a structured plan:

```
## Study Plan: <title>

### Phase 1: Setup
- Mesh convergence study (if production fidelity requested)
- Define baseline configuration

### Phase 2: Analysis
- <list of analysis steps, mapped to specific skills>
- Expected number of OAS runs: <N>

### Phase 3: Comparison
- Configuration comparison across all runs
- Generate comparison table

### Phase 4: Reporting
- Physical analysis report
- Export for <tool> (if multi-tool workflow)
- Provenance export

### Estimated Scope
- Total OAS runs: <N>
- Skills to invoke: <list>
```

Present the plan and ask the user to confirm before proceeding.

## Multi-Tool Workflow Guidance

When OAS is one tool in a larger chain:

1. **Run OAS analyses** using the appropriate skills above
2. **Export results** using `export-for-integration` with the target tool format
3. **Hand off** to the downstream tool (Aviary, OpenConcept, etc.)
4. **Iterate** if the downstream tool requires modified OAS inputs

The workflow-builder helps plan the OAS portion. The user (or another plugin) handles the downstream tool integration.

## Example Study Plans

### "Compare conventional vs TBW at three aspect ratios"
1. `mesh-convergence-study` → determine num_y
2. `architecture-comparison` with configs: conventional AR=9, conventional AR=13.5, TBW AR=17
3. `configuration-comparison` for pairwise deltas
4. `physical-analysis-report` for final write-up

### "Find optimal truss placement on a TBW"
1. `parameter-sweep` varying strut span fraction: 40%, 50%, 60%, 70%
2. For each: `run-aerostruct-analysis` with wing + strut
3. `configuration-comparison` across all placements
4. Optionally: `run-aerostruct-opt` at the best placement
