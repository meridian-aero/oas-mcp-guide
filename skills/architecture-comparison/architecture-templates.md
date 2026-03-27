# Architecture Templates

Pre-validated surface definitions for common configurations. Copy and adapt these.

## Conventional Transport (CRM, AR ≈ 9)

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=11,
    symmetry=True, with_viscous=True, with_wave=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

## High-Aspect-Ratio Wing (AR ≈ 13.5)

Based on the uCRM-13.5 benchmark:

```
create_surface(
    name="wing", wing_type="uCRM_based", span=75.0, num_x=2, num_y=13,
    symmetry=True, with_viscous=True, with_wave=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0,
    distributed_fuel_weight=True, struct_weight_relief=True
)
```

## Truss-Braced Wing (TBW)

Wing + strut as two surfaces:

**Main wing** (high AR, reduced root bending):
```
create_surface(
    name="wing", wing_type="uCRM_based", span=80.0, num_x=2, num_y=13,
    symmetry=True, with_viscous=True, with_wave=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0,
    distributed_fuel_weight=True, struct_weight_relief=True
)
```

**Strut** (attaches at ~60% span):
```
create_surface(
    name="strut", wing_type="rect", span=24.0, root_chord=1.5,
    num_x=2, num_y=7, symmetry=True, with_viscous=True,
    offset=[25.0, -10.0, -3.0],
    fem_model_type="tube",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

The `offset` positions the strut below and behind the wing root. Adjust:
- `offset[0]` (x): chordwise position of strut root relative to wing
- `offset[1]` (y): spanwise position (negative = inboard from wingtip)
- `offset[2]` (z): vertical offset (negative = below wing)

Analysis call includes both surfaces:
```
run_aerostruct_analysis(surfaces=["wing", "strut"], ...)
```

## Blended Wing Body (Simplified)

OAS models BWB as a single wide-chord, low-taper surface. No fuselage model.

```
create_surface(
    name="wing", wing_type="rect", span=60.0, root_chord=15.0,
    taper=0.15, sweep=30.0, num_x=3, num_y=11,
    symmetry=True, with_viscous=True, CD0=0.010,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

**Caveat**: This only captures the wing aerodynamics. BWB body lift, cabin volume, and fuselage weight are not modeled.

## Rectangular Baseline (Educational)

```
create_surface(
    name="wing", wing_type="rect", span=10.0, root_chord=1.0,
    num_x=2, num_y=7, symmetry=True, with_viscous=True
)
```
