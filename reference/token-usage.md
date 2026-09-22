# Token usage (inform the user)

ARPES work can use many LLM tokens when the agent dumps large arrays, walks
whole beamtime folders in-chat, or re-reads big logs. **Warn the user before
expensive steps** — not to discourage, but so they can choose scope / batching.

Tone: calm, factual. Offer a lighter path when useful.

## When to warn (say this pattern)

> **Token note:** This next step may use a lot of context/tokens because
> [reason]. I can instead [lighter option]. Proceed with the full step,
> or prefer the lighter option?

Wait for a clear preference when the step is clearly high-cost. For mild
cost, a one-line note + continue is enough.

## High-cost steps (warn before starting)

| Step | Why tokens spike | Lighter alternative |
|------|------------------|---------------------|
| Catalog **many** `.h5` files in one reply | Metadata + shapes × N in chat | Script writes `analysis/*_catalog.md`; chat only summary table |
| Paste **full array / DataArray** into chat | Huge numeric dumps | Print shape, coords, min/max/mean; save `.nc` / plot PNG under `analysis/` |
| Re-load whole **measurement log** repeatedly | Long CSV in context | Cache summary once; query by run number |
| **Broadcast fits** over full 2D/3D maps | Long fit reports × many curves | Fit one EDC/MDC first; then scripted broadcast → save params CSV |
| Full **k / kz conversion** volumes + prose dump | Large grids in text | Convert in script; plot or save; chat = axes + assumptions only |
| Attach / describe **many PNG** overviews | Image tokens add up | Few representative figures; rest on disk |
| Install + long **pip/conda logs** in chat | Noisy build output | Run install quietly; report only success/fail + env path |
| Re-read entire `reference/` every turn | Skill context bloat | Read one matching reference file when needed |

## Low-cost steps (usually no warning)

- One file: load → dims/coords/units sanity print  
- One mid-cut or near-EF map plot saved to `analysis/figures/`  
- One EDC/MDC extract + one peak fit  
- Short checklist / yes-no questions  

## Agent habits that save tokens

1. Prefer **scripts under `analysis/`** that write reports/figures; summarize results in chat.
2. Never paste raw intensity arrays into the conversation.
3. Cap catalogs: default to a **sample** (e.g. 3–5 files) before offering full-folder runs.
4. After a long tool log, reply with a **short** status — do not echo the whole log.
5. Keep PyARPES env path in one line; do not re-paste install recipes every turn.

## Example one-liners

- Full Blue folder catalog (25 files):  
  *“Token note: cataloging all 25 files in-chat is heavy. I’ll write a catalog script + markdown under `analysis/` and only paste a short summary here — OK?”*

- Broadcast MDC fits across a cut:  
  *“Token note: broadcast fits produce long reports. I’ll fit one MDC here, then run the rest in a script and save `widths.csv` — OK?”*
