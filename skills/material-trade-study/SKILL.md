---
name: material-trade-study
description: >
  Compare different materials (aluminum, titanium, composite, steel) for wing
  structural sizing. Use when the user asks about material selection, wants to
  compare metallic vs composite, or wants to understand how material properties
  affect structural mass and fuel burn. Triggers on: "material comparison",
  "aluminum vs composite", "titanium wing", "material trade",
  "best material for", "metallic vs composite", "material selection".
argument-hint: "[materials]  e.g. 'aluminum vs titanium vs composite'"
context: fork
---

# Material Trade Study

Compare how different material choices affect structural mass, fuel burn, and failure margins for the same wing geometry.

## Workflow

### 1. Start Session

```
start_session(notes="Material trade study: <materials>")
```

### 2. Define Materials

Parse `$ARGUMENTS` for material list. Default comparison: aluminum, titanium, simplified composite.

Look up properties in `shared/material-database.md`:

| Material | E (Pa) | G (Pa) | yield_stress (Pa) | mrho (kg/m3) |
|---|---|---|---|---|
| Aluminum 7075-T6 | 70e9 | 30e9 | 500e6 | 3000 |
| Titanium Ti-6Al-4V | 114e9 | 42e9 | 950e6 | 4430 |
| CFRP quasi-isotropic | 70e9 | 30e9 | 900e6 | 1600 |

### 3. Run Analysis for Each Material

For each material:

```
reset()  # Required: clear cached problem between materials

create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="tube",
    E=<material_E>, G=<material_G>,
    yield_stress=<material_yield>, mrho=<material_mrho>
)

run_aerostruct_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38,
    reynolds_number=1e6, speed_of_sound=295.4, W0=120000.0
)

log_decision(
    decision_type="result_interpretation",
    reasoning="<material>: struct_mass=<>, fuelburn=<>, failure=<>",
    selected_action="continue to next material",
    prior_call_id="<_provenance.call_id>"
)
```

### 4. Optional: Optimize Each Material

For a fair comparison, optimize the thickness distribution for each material:

```
run_optimization(
    surfaces=["wing"], analysis_type="aerostruct", objective="fuelburn",
    design_variables=[
        {"name": "twist", "lower": -10, "upper": 15},
        {"name": "thickness", "lower": 0.003, "upper": 0.25, "scaler": 100},
        {"name": "alpha", "lower": -5, "upper": 10}
    ],
    constraints=[
        {"name": "L_equals_W", "equals": 0.0},
        {"name": "failure", "upper": 0.0},
        {"name": "thickness_intersects", "upper": 0.0}
    ],
    objective_scaler=1e-5
)
```

This ensures each material uses its optimal thickness — a more meaningful comparison than using the same thickness for all materials.

### 5. Build Comparison Table

| Material | E (GPa) | yield (MPa) | mrho | struct_mass (kg) | fuelburn (kg) | failure | weight savings vs Al (%) |
|---|---|---|---|---|---|---|---|
| Aluminum | 70 | 500 | 3000 | ... | ... | ... | baseline |
| Titanium | 114 | 950 | 4430 | ... | ... | ... | ... |
| Composite | 70 | 900 | 1600 | ... | ... | ... | ... |

### 6. Analyze and Recommend

```
log_decision(
    decision_type="result_interpretation",
    reasoning="<Pareto analysis: which materials dominate on which metrics>",
    selected_action="<recommended material and rationale>"
)
```

### 7. Export

```
export_session_graph(output_path="material_trade_provenance.json")
```

## Notes

- **Simplified composite** (`use_composite=False`, low mrho, high yield) gives a quick estimate of composite benefits without needing ply-level detail.
- **Full laminate composite** (`use_composite=True`) requires `fem_model_type="wingbox"` and full ply properties. Use only when the user specifically asks for laminate-level analysis.
- The comparison is most meaningful when each material is **optimized independently** — otherwise the default thickness may favor one material over another.
