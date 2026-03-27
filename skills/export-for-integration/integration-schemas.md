# Integration Schemas

Detailed export format specifications for each target tool.

## Aviary Export Schema

Aviary uses OpenAeroStruct as a pre-mission subsystem for wing mass and drag estimation.

### Wing Geometry File (`wing_geometry.json`)
```json
{
  "wing_span_m": 58.0,
  "wing_aspect_ratio": 9.0,
  "wing_taper_ratio": 0.3,
  "wing_sweep_deg": 25.0,
  "wing_t_over_c": [0.12, 0.11, 0.10, 0.09, 0.08],
  "wing_twist_deg": [3.0, 2.0, 1.0, 0.0, -1.5],
  "wing_area_m2": 374.0
}
```

Maps to Aviary's `Aircraft.Wing.*` hierarchy.

### Drag Polar File (`drag_polar.csv`)
```
alpha_deg,CL,CD,CM
-5.0,-0.12,0.0180,-0.02
0.0,0.28,0.0195,0.01
5.0,0.52,0.0280,0.03
10.0,0.75,0.0450,0.05
```

Used by Aviary's `AerodynamicsTable` component.

### Structural Mass (`structural_data.json`)
```json
{
  "wing_structural_mass_kg": 25000,
  "wing_weight_ratio": 2.0,
  "total_wing_mass_kg": 50000
}
```

## OpenConcept Export Schema

### CL-CD Lookup Table (`aero_table.csv`)
```
CL,CD
-0.12,0.0180
0.28,0.0195
0.52,0.0280
0.75,0.0450
```

### Summary (`oas_summary.json`)
```json
{
  "structural_mass_kg": 25000,
  "design_CL": 0.5,
  "design_CD": 0.025,
  "wing_area_m2": 374.0
}
```

## AeroSandbox Export Schema

### Mesh and Cp Data (`oas_aero_data.json`)
```json
{
  "mesh_x": [[...], [...]],
  "mesh_y": [[...], [...]],
  "mesh_z": [[...], [...]],
  "CL": 0.52,
  "CD": 0.028,
  "sectional_CL": [0.35, 0.42, 0.50, 0.48, 0.40],
  "span_stations_m": [-29.0, -14.5, 0.0, 14.5, 29.0],
  "wing_params": {
    "span_m": 58.0,
    "root_chord_m": 8.0,
    "taper": 0.3,
    "sweep_deg": 25.0
  }
}
```

## SUAVE Export Schema

### Vehicle Data (`oas_vehicle.json`)
```json
{
  "wings": {
    "main_wing": {
      "spans": {"projected": 58.0},
      "areas": {"reference": 374.0},
      "aspect_ratio": 9.0,
      "taper": 0.3,
      "sweeps": {"quarter_chord": 25.0},
      "twists": {"root": 3.0, "tip": -1.5},
      "mass_properties": {"mass": 25000}
    }
  },
  "aerodynamics": {
    "CL_design": 0.52,
    "CD_design": 0.028,
    "L_over_D_design": 18.6
  }
}
```

## RCAIDE Export Schema

### Standardized Analysis Results (`oas_results.json`)
```json
{
  "format_version": "1.0",
  "source_tool": "OpenAeroStruct",
  "run_id": "...",
  "geometry": {
    "type": "wing",
    "span_m": 58.0,
    "aspect_ratio": 9.0,
    "taper_ratio": 0.3,
    "sweep_deg": 25.0,
    "t_over_c_distribution": [0.12, 0.10, 0.08],
    "twist_distribution_deg": [3.0, 1.0, -1.5]
  },
  "aerodynamics": {
    "CL": 0.52,
    "CD": 0.028,
    "L_over_D": 18.6,
    "CL_alpha_per_rad": 5.5,
    "drag_polar": {"alphas": [...], "CLs": [...], "CDs": [...]}
  },
  "structures": {
    "structural_mass_kg": 25000,
    "failure_metric": -0.15,
    "thickness_distribution_m": [0.05, 0.04, 0.03]
  },
  "mission": {
    "fuelburn_kg": 95000,
    "range_m": 14300000
  }
}
```

## Generic CSV Export

For tools without a specific schema:

```csv
metric,value,unit
CL,0.52,
CD,0.028,
L_over_D,18.6,
structural_mass,25000,kg
fuelburn,95000,kg
span,58.0,m
aspect_ratio,9.0,
```
