# Material Database

Property values for use with `create_surface`. The `mrho` values include typical fastener/manufacturing overhead.

## Metallic Materials

| Material | E (Pa) | G (Pa) | yield_stress (Pa) | mrho (kg/m3) | Typical Use |
|---|---|---|---|---|---|
| Aluminum 7075-T6 | 70e9 | 30e9 | 500e6 | 3000 | OAS default; transport wing primary structure |
| Aluminum 2024-T3 | 73e9 | 28e9 | 345e6 | 2780 | Lower strength, lower cost; secondary structure |
| Titanium Ti-6Al-4V | 114e9 | 42e9 | 950e6 | 4430 | High strength-to-weight; high temperature |
| Steel 4340 | 200e9 | 80e9 | 1100e6 | 7850 | Landing gear, engine mounts (not for wings) |

## Composite Materials — Simplified (Quasi-Isotropic)

Use with `use_composite=False` (default). OAS treats the material as isotropic with effective properties.

| Material | E (Pa) | G (Pa) | yield_stress (Pa) | mrho (kg/m3) |
|---|---|---|---|---|
| CFRP quasi-isotropic | 70e9 | 30e9 | 900e6 | 1600 |

## Composite Materials — Full Laminate

Use with `use_composite=True` and `fem_model_type="wingbox"`. Requires specifying ply-level properties.

### Typical CFRP Laminate Properties

```
E1=135e9         # Longitudinal modulus (fiber direction)
E2=10e9          # Transverse modulus
nu12=0.3         # Major Poisson's ratio
G12=5e9          # In-plane shear modulus
```

### Strength Allowables

```
sigma_t1=1500e6  # Longitudinal tensile strength
sigma_c1=1200e6  # Longitudinal compressive strength
sigma_t2=50e6    # Transverse tensile strength
sigma_c2=250e6   # Transverse compressive strength
sigma_12max=70e6 # Maximum shear strength
```

### Standard Layup

```
ply_angles=[0, 45, -45, 90]
ply_fractions=[0.4, 0.25, 0.25, 0.1]  # must sum to 1.0
```

## Material Selection Guidance

- **Aluminum 7075-T6**: Start here. Good balance of strength, stiffness, and cost. OAS defaults match this material.
- **Titanium**: When weight savings at high stress are needed (≈50% stronger than Al at 1.5× density).
- **Simplified composite**: For quick comparison vs metals. The 1600 kg/m3 density with 900 MPa yield gives substantial weight savings.
- **Full laminate composite**: When you need to study ply orientation effects or use Tsai-Wu failure. Only works with wingbox model.

## Safety Factor

OAS default `safety_factor=2.5` applied to yield_stress. The failure metric uses `yield_stress / safety_factor` as the allowable.
