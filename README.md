# Steam Condensation on a Vertical Cold Wall — VOF + Heat Transfer in ANSYS Fluent

A transient CFD study of laminar film condensation on a vertical cold wall, simulated using the Volume of Fluid (VOF) multiphase model with the Lee Evaporation-Condensation model in ANSYS Fluent 2026 R1. The simulation captures the formation and growth of a liquid condensate film on a wall maintained below the saturation temperature of steam.


## Table of Contents
- [What Was Done](#what-was-actually-done)
- [Setup / Software Used](#setup--software-used)
- [Reproducing the Results](#reproducing-the-results)
- [Results](#results)
- [Method, Briefly](#method-briefly)
- [Repository Structure](#repository-structure)
- [Author](#author)

## What was actually done

- Perform a transient 2D VOF analysis for condensation of steam on a vertical wall (0.1 m x 0.5 m rectangle)
- The Lee Evaporation-Condensation model was used with saturation temperature of 373.15 K constant, and condensation and evaporation frequency coefficients set to 100 s⁻¹ after using default value 0.1 s⁻¹ gave no phase change.
- Enabled gravity (Y = −9.81 m/s²) and buoyancy effects, which drive the condensate film downward along the wall under its own weight
- Applied a cold wall boundary at 350 K on the left vertical edge, with steam entering at 380 K from the bottom at 0.1 m/s
- Built inflation layers on the cold wall (first layer 0.1 mm, 10 layers, growth rate 1.2) to resolve the thin liquid film that forms there
- Ran over 20,000 transient iterations at Δt = 0.001 s to develop the condensate film to a quasi-steady state
- Confirmed condensation with a peak liquid volume fraction of 0.273 at the wall, a clear thermal boundary layer in the temperature contour, and a characteristic velocity profile showing accelerating condensate near the bottom of the wall
- Debugged a "Latent Heat cannot be less than zero" error by correcting the phase direction convention in the Lee model mass transfer setup

## Setup / Software Used

* ANSYS Designermodeler (geometry — 2D rectangle, 100 mm × 500 mm)
* ANSYS Meshing 2026 R1 — quadrilateral mesh, global element size 5 mm, inflation on cold wall
* ANSYS Fluent 2026 R1, Student version — transient pressure-based VOF solver
* Lee Evaporation-Condensation model for phase change mass transfer
* PISO pressure-velocity coupling; PRESTO! pressure discretisation; Compressive VOF scheme

## Reproducing the Results

1. Open ANSYS Workbench -> Fluid Flow (Fluent)
2. Create a 2D rectangular geometry of dimensions 100 mm x 500 mm using ANSYS designermodeler
3. In meshing, the global element size is kept 0.005 m, and inflation is added to the left edge (cold wall). First Layer Thickness = 0.0001 m, 10 layers, growth rate 1.2; names for all the four edges: cold_wall (left), adiabatic_wall (right), inlet (bottom), outlet (top)
4. In Fluent General settings: set solver to Transient; enable gravity with Y = −9.81 m/s²
5. Models: activate VOF (2 phases, Implicit, Sharp interface, Implicit Body Force on); turn Energy on; keep Viscous as Laminar
6. Materials: load water-vapor and water-liquid from the Fluent database; do not manually change enthalpy values — use defaults to avoid the negative latent heat error
7. Phases: Phase 1 = water-vapor (primary), Phase 2 = water-liquid (secondary); Phase Interaction → Heat, Mass, Reactions → set Number of Mass Transfer Mechanisms to 1, From = water-liquid, To = water-vapor, Mechanism = Evaporation-Condensation (Lee), both frequency coefficients = 100 s⁻¹, saturation temperature = 373.15 K; surface tension = 0.0589 N/m with Wall Adhesion on
8. Boundary conditions: inlet — velocity 0.1 m/s in +Y, temperature 380 K, Phase 2 volume fraction = 0; outlet — pressure outlet 0 Pa, 373 K; cold_wall — temperature 350 K; adiabatic_wall — heat flux 0 W/m²
9. Solution Methods: PISO, PRESTO! pressure, Second Order Upwind momentum and energy, Compressive volume fraction
10. Initialize from inlet; run 1000 time steps at Δt = 0.001 s to start, then extend as needed
11. Post-process: check Volume Fraction (secondary) contour zoomed into the left wall — the liquid film should be visible as a red/orange strip; also check Static Temperature and Velocity Magnitude contours

## Results

**Simulation parameters**

| Parameter | Value |
|---|---|
| Domain | 0.1 m × 0.5 m (2D) |
| Wall temperature | 350 K |
| Steam inlet temperature | 380 K |
| Saturation temperature | 373.15 K |
| Inlet velocity | 0.1 m/s |
| Lee model coefficients | 100 s⁻¹ |
| Time step | 0.001 s |
| Total iterations | ~20,000+ |

**Key results**

Peak liquid volume fraction at the cold wall reached **0.273**, confirming successful film condensation. The liquid film is spatially confined to the immediate vicinity of the cold wall, with the bulk domain remaining at near-zero liquid fraction — consistent with physical expectation.

The velocity contour shows peak mixture velocity of **0.487 m/s** concentrated near the bottom of the cold wall, where condensate accelerates under gravity before draining — the same pattern described in Nusselt's analytical film condensation theory. Wall shear stress ranged from 8.32×10⁻⁵ Pa at the top of the wall to 0.264 Pa near the bottom, reflecting the increasing film momentum as condensate accumulates along the wall height.

The temperature contour shows a narrow thermal boundary layer confined to a few millimetres from the cold wall surface, with the bulk steam remaining near 380 K throughout — confirming that the liquid film is acting as the dominant thermal resistance between the wall and the vapor, again consistent with Nusselt theory.

All contour plots are in `results/contours/`.

## Method, briefly

- **Geometry:** Build the geometry as a flat 2D rectangle in DesignModeler — 0.1 m × 0.5 m. Tall and narrow aspect ratio chosen to give the condensate film room to develop along the wall height.
- **Mesh:** Quadrilateral elements, used 5 mm as the global size, made a 10-layer inflation on cold wall edge (first layer 0.1 mm, growth rate 1.2).
- **Multiphase model:** VOF with Sharp interface formulation. Primary phase = water-vapor, secondary = water-liquid. Implicit Body Force enabled to stabilise the solution with gravity active.
- **Phase change:** Lee Evaporation-Condensation model. With the default frequency coefficient of 0.1 s-1, there was no condensation over 1,000 time steps, but increasing it to 100 s-1 resulted in film formation. The phase direction convention (From = liquid, To = vapor) was picked to prevent the negative latent heat error which happens when the enthalpy difference is computed in the wrong direction.
- **Solver:** Transient method, solution method is PISO, PRESTO! for pressure (required with VOF and body forces), Compressive scheme for volume fraction to maintain a sharp liquid-vapor interface.
- **Boundary conditions:** Cold wall at 350 K drives condensation; steam enters from the bottom at 380 K, 23 K above saturation, to ensure a consistent vapor supply throughout the domain height.

## Limitations

The Lee model frequency coefficient of 100 s⁻¹ is higher than values used in some published studies (which range from 0.1 to ~10 s⁻¹ depending on the system). The higher value was needed to trigger condensation at this mesh resolution and time step; a proper sensitivity study varying the coefficient and comparing predicted condensation rates against Nusselt's analytical solution would be the right next step before treating these results as quantitatively validated.

No formal grid convergence study was done — mesh independence was checked qualitatively by confirming that the film appeared consistently across the wall rather than as isolated patches, but a GCI-based Richardson extrapolation study was not completed in this version.

## Repository Structure

```
├── README.md
├── geometry/
│   └── (geometry — 2D rectangle 100 mm × 500 mm)
├── mesh/
│   └── (mesh)
├── results/
│   ├── contours/
│   │   ├── static_temperature_contour
│   │   ├── velocity_magnitude_contour
│   │   ├── volume_fraction_contour
│   │   ├── wall_shear_stress_on_cold_wall
│   └── residuals/
│       └── scaled_residuals
└── case_files/
    └── steam condensation on a vertical wall_files
    └── steam condensation on a vertical wall 
    └── Report
```

## Author

**Pratyush Dash**

B.Tech Chemical Engineering, KIIT University, Bhubaneswar
