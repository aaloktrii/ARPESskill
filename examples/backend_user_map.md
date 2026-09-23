# Example: fill a user backend capability map

Follow `reference/backend-capability-map.md`. Use only after the user **declines**
`.venv-arpes` (or explicitly asks to map project code).

## Outline

1. Offer PyARPES 3.8 venv first — if declined, continue here.
2. Search the project for load / EF / k / fit / gap helpers.
3. Propose a map (IDs → callables); **wait for confirm**.
4. Optional: write `analysis/backend_map.json` in the **user project**.
5. Run the requested ARPES workflow using confirmed callables only.
6. Unmapped ID → ask or skip; do not invent.

```json
{
  "backend": "user-map",
  "confirmed": true,
  "map": {
    "load_spectrum": "mypackage.io.load",
    "fit_fermi_edge": "mypackage.fermi.fit_edge",
    "convert_k": "mypackage.kspace.convert"
  }
}
```

Report backend = `user-map` and which IDs were used. Physics assumptions
(EF, Γ, V₀, T, resolution, …) still required.
