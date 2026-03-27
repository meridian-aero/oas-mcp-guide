# Objective Scaling Guide

Proper scaling is critical for optimizer convergence. The objective value should be O(1) after scaling.

## Objective Scalers

| Objective | Typical Baseline Value | Recommended `objective_scaler` | tolerance |
|---|---|---|---|
| CD | 0.02–0.04 | 30–50 (or `1/baseline_CD`) | 1e-6 |
| fuelburn | 50,000–150,000 kg | 1e-5 | 1e-9 |
| structural_mass | 15,000–40,000 kg | 3e-5 to 5e-5 | 1e-9 |

## Best Practice: Compute Scaler from Baseline

Run a baseline analysis first, then set `objective_scaler = 1.0 / baseline_value`:

1. `run_aero_analysis` or `run_aerostruct_analysis` → get baseline CD or fuelburn
2. `objective_scaler = 1.0 / baseline_CD` (for aero-only)
3. `objective_scaler = 1.0 / baseline_fuelburn` (for aerostruct)

This approach is more robust than using fixed scalers because it adapts to the specific problem.

## Design Variable Scalers

DV scalers are set via the `scaler` field in the DV dict. They scale the DV value seen by the optimizer.

| DV | Typical Range | Recommended `scaler` |
|---|---|---|
| twist | -15 to +15 deg | 1 (no scaler needed) |
| alpha | -10 to +15 deg | 1 |
| chord | 0.5 to 3.0 m | 1 |
| thickness (tube) | 0.003 to 0.25 m | 100 |
| spar_thickness (wingbox) | 0.001 to 0.05 m | 1000 |
| skin_thickness (wingbox) | 0.001 to 0.05 m | 1000 |
| t_over_c | 0.06 to 0.20 | 10 |

## Convergence Tolerance

| Use Case | `tolerance` | Notes |
|---|---|---|
| Quick screening | 1e-4 | Fast, approximate |
| Production | 1e-6 | Default, good for most problems |
| High accuracy | 1e-8 | Slow; needed for gradient validation |

## Max Iterations

Default is 200. Increase to 400–500 for multipoint or wingbox problems with many DVs.
