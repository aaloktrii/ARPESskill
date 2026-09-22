# Failure modes

Common agent mistakes in ARPES analysis and the correct behavior. Cross-check against
`SKILL.md` hard rules before reporting results.

| Failure | Correct |
|---------|---------|
| Plot angle axis labeled as k | Convert first with `convert_to_kspace`, or label axes in **degrees** (°) |
| hv scan plotted as kz without V₀ | Set or ask for `inner_potential`; state uncertainty if V₀ unknown |
| Swap binding ↔ kinetic | Check PyARPES convention (binding often ≤0 below EF); state which is used |
| Invent MAESTRO motor names | Read coords/attrs from file — never guess `phi`, `theta`, etc. |
| "Γ is at image center" | No — state method to find Γ (manual pick, fit, symmetry, model) |
| Fit without lineshape | Name Gaussian, Lorentzian, or Voigt (+ background if used) |
| Use TensorSpec APIs in v1 | Defer; use PyARPES for load, reduce, fit, and k/kz |
| Launch QtTool as only path | Prefer scripted PyARPES + matplotlib; GUIs are optional |

## Additional guidance

- **Angle vs momentum:** detector or manipulator angles are in degrees until
  `convert_to_kspace` produces k coordinates. See `reference/k-and-kz-conversion.md`.
- **Inner potential:** absolute kz from hv scans requires V₀ in
  `spectrum.attrs["inner_potential"]`. If unknown, report relative kz or ask one
  sharp question.
- **Γ (gamma point):** the Brillouin-zone center is not assumed at the image center.
  Identify it by stated method (symmetry, known sample orientation, band structure
  model, or user-provided reference).
- **Fits:** every reported fit must name the lineshape and any background model. See
  `reference/edc-mdc-fitting.md`.
- **Stack policy:** v1 uses PyARPES only for analysis. TensorSpec and Qt-based tools are
  out of scope unless the user explicitly defers to inspect-only or future work.

When in doubt, read the matching `reference/` file and ask the user one sharp question
rather than inventing axes, units, or physics assumptions.
