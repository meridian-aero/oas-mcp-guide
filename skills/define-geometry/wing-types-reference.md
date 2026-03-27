# Wing Types Reference

## Available Wing Types

### `rect` — Rectangular Planform
- Uniform chord from root to tip
- Specify `span` and `root_chord` to set geometry
- Apply `taper`, `sweep`, `dihedral` to modify
- Best for: parametric studies, educational use, clean comparisons

### `CRM` — NASA Common Research Model
- Realistic transonic transport wing geometry
- Built-in sweep, taper, and twist distribution
- Default span ≈ 58 m (can override with `span`)
- `num_twist_cp` controls the number of CRM twist interpolation points
- Best for: single-point transport aircraft studies, validation against published data

### `uCRM_based` — Updated CRM for Multipoint
- Extended CRM with additional parameters for multipoint optimization
- Supports `S_ref_type`, `c_max_t`, `wing_weight_ratio`, `struct_weight_relief`, `distributed_fuel_weight`
- Best for: multipoint optimization studies, fuel volume constraints, Aviary integration

## Parameter Ranges

| Parameter | Typical Range | Notes |
|---|---|---|
| `span` | 10–80 m | 58 m for transport class |
| `root_chord` | 1–15 m | Only used for `rect` |
| `taper` | 0.2–1.0 | 1.0 = no taper; 0.3 typical for transport |
| `sweep` | 0–35 deg | LE sweep; 25 deg typical for transonic |
| `dihedral` | 0–10 deg | 5–7 deg typical for transport |
| `num_y` | 3–35 | Must be **odd**; 7 for quick, 13+ for production |
| `num_x` | 2–5 | 2 sufficient for VLM; more for wave drag accuracy |

## Structural Model Selection

| Model | When to Use | Required Params |
|---|---|---|
| `None` (aero-only) | Aerodynamic-only analysis, drag polar, stability | None |
| `tube` | Quick structural sizing, tube spar studies | E, G, yield_stress, mrho, thickness_cp |
| `wingbox` | Realistic structural sizing, spar+skin optimization | E, G, yield_stress, mrho, spar_thickness_cp, skin_thickness_cp |

## Multi-Surface Configurations

For TBW or canard configurations, call `create_surface` multiple times with different names:

```
create_surface(name="wing", wing_type="CRM", ...)
create_surface(name="strut", wing_type="rect", span=20, offset=[25, -10, -3], ...)
```

Surface names must match exactly when passed to analysis tools via the `surfaces` parameter.
