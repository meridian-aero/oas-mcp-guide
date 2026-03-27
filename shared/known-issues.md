# Known Issues and Workarounds

## `load_factor` Caching

Changing `load_factor` between runs in the same session may not update the cached OpenMDAO problem. **Workaround**: Call `reset` before re-running with a different load factor.

## Design Variable Name Mismatches

Incorrect DV names in `run_optimization` fail silently — the optimizer runs but doesn't vary the intended variable. **Workaround**: Always verify DV names against the documented names:

- All models: `twist`, `chord`, `sweep`, `taper`, `alpha`, `t_over_c`
- Tube only: `thickness`
- Wingbox only: `spar_thickness`, `skin_thickness`
- Multipoint only: `alpha_maneuver`, `fuel_mass`

## `num_y` Must Be Odd

Passing an even `num_y` to `create_surface` raises a validation error. Always use odd values: 3, 5, 7, 9, 11, 13, 15, 21, 35, etc.

## Composite Requires Wingbox

`use_composite=True` is only supported with `fem_model_type="wingbox"`. Using it with `fem_model_type="tube"` raises an error.

## Ground Effect Constraints

`groundplane=True` requires `symmetry=True` and is incompatible with sideslip (`beta != 0`). Using both raises an error.

## Angular Velocity (`omega`) Changes Model Topology

The first call with `omega` (angular velocity for rotating wings/propellers) builds a new OpenMDAO problem. Subsequent calls reuse that rotational-enabled cache. Switching between omega=0 and omega>0 triggers a rebuild.

## Control Point Ordering

All `*_cp` arrays (`twist_cp`, `chord_cp`, `thickness_cp`, `spar_thickness_cp`, `skin_thickness_cp`) are **root-to-tip** in the MCP interface. The server automatically converts to OAS internal tip-to-root order.

## Visualization in CLI

MCP `ImageContent` renders as `[image]` in CLI. Use `configure_session(visualization_output="file")` or `configure_session(visualization_output="url")` to get useful output.
