# Folder manifest (session inventory)

Before analyzing files in a data folder, build a **durable inventory** on disk
so later turns can **recall** what exists without re-walking raw data or
pasting catalogs into chat.

This is **structured project memory** (JSON + optional Markdown summary) — not
vector RAG. Agents **read/query the file**; they do not embed the beamtime.
**Future (optional):** semantic retrieval (e.g. embeddings or a hosted file-search
API) may sit on top of this manifest without replacing it as the source of truth.

## When

| Situation | Action |
|-----------|--------|
| User points at a **folder** / beamtime / many files | Build or refresh manifest **first** |
| Manifest exists and raw mtimes/hashes unchanged | **Reuse**; do not rebuild unless asked |
| New files added / mtimes changed / user says refresh | Rebuild or patch changed rows |
| Single known file path | Manifest optional; still OK to add one row |

Give a **token note** before a full-folder deep load (`reference/token-usage.md`).
Prefer: list cheap metadata first; deep-load shapes only as needed (or sample
3–5 files, then offer full pass).

## Where

```text
analysis/
  manifest.json          # source of truth (required)
  manifest.md            # short human table (optional but recommended)
  figures/…              # overview PNGs linked from rows when present
  kspace/…               # converted products linked when present
```

Paths are relative to the **analysis project root** (not inside the raw data
tree unless the user keeps analysis there).

## `manifest.json` schema

Top level:

```json
{
  "schema_version": 1,
  "created_utc": "ISO-8601",
  "updated_utc": "ISO-8601",
  "data_root": "/absolute/or/relative/raw/folder",
  "skill_ref": "arpes",
  "files": [ /* FileEntry */ ]
}
```

### `FileEntry` fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `stem` | string | yes | Filename without extension |
| `path` | string | yes | Preferred raw path to load |
| `path_alt` | string \| null | no | Sibling `.h5` / `.fits` if both exist |
| `ext` | string | yes | e.g. `.fits`, `.h5` |
| `preferred_loader` | string | yes | e.g. `arpes.io.load_data` + `location=MAESTRO` |
| `size_bytes` | int | yes | |
| `mtime_utc` | string | yes | File mtime ISO-8601 |
| `content_sha256` | string \| null | no | Optional; recompute on refresh if cheap |
| `log_comment` | string \| null | no | From measurement log if matched; comment only |
| `kind` | string | yes | See kind enum below |
| `kind_confidence` | string | no | `high` \| `medium` \| `low` |
| `kind_clues` | string[] | no | e.g. `["swept","span_12eV"]` |
| `shape` | int[] \| null | no | After successful peek load |
| `dims` | string[] \| null | no | |
| `hv_eV` | number \| null | no | Scalar hv if known |
| `energy_span_eV` | number \| null | no | \|Emax−Emin\| if known |
| `energy_axis` | string \| null | no | `Ek` \| `Eb` \| `E-EF` \| `ambiguous` |
| `load_ok` | bool \| null | no | null = not peeked yet |
| `load_error` | string \| null | no | |
| `overview_paths` | string[] | no | Relative paths to PNG(s) under `analysis/` |
| `product_paths` | string[] | no | e.g. `analysis/kspace/<stem>_k.npz` |
| `notes` | string \| null | no | Short free text |

### Kind enum

Use these strings (extend only if needed; document in `notes`):

`cut` · `core_level_2d` · `fermi` · `hv` · `xy` · `unknown` · `load_error`

Classify with existing rules: dims first; core-as-2D heuristics; log is comment
only (`reference/default-overview-plots.md`).

## Build / refresh algorithm

1. Enumerate data files under `data_root` (prefer `.fits` when sibling `.h5`
   exists — record both; set `path` to preferred).
2. Match measurement log by stem if a log is available → `log_comment`.
3. Cheap row: stem, paths, ext, size, mtime (and hash if enabled).
4. Optional peek load (PyARPES): fill shape, dims, hv, energy_span, energy_axis,
   kind guess, `load_ok` / `load_error`.
5. Write `manifest.json`; write `manifest.md` summary table (stem, kind, hv,
   shape, load_ok, comment).
6. Chat: **short summary only** (counts by kind + path to manifest) — not the
   full JSON.

### Invalidation

Rebuild or update a row when:

- `mtime_utc` or `content_sha256` differs from the stored entry, or
- user requests refresh, or
- `data_root` changed.

Unchanged rows: keep overview/product links.

## Recall later (required habit)

Before opening raw files for a follow-up task:

1. Load `analysis/manifest.json` if present.
2. Filter by `kind` / stem / hv / `load_ok` as the user asked.
3. Open only the matching `path`s (or linked `product_paths` if analysis
   products suffice).
4. If manifest missing or stale → rebuild (with token note if large).

Do **not** re-paste the full catalog into chat when the manifest is enough.

## Lazy depth (recommended)

| Pass | What | Cost |
|------|------|------|
| **A — listing** | Paths, size, mtime, log comment, preferred ext | Low |
| **B — peek** | Load header/spectrum briefly → shape, kind, hv | Medium |
| **C — overview** | Write PNGs per `default-overview-plots.md`; set `overview_paths` | Higher |

Default for a new folder: **A**, then **B** (or sample **B** then offer full).
**C** when user wants a quick report / catalog with figures.

## Agent rules

1. Manifest before multi-file analysis when a folder is in scope.
2. Prefer recall from manifest over rediscovering the folder.
3. Never dump full intensity arrays into the manifest or into chat.
4. Update `overview_paths` / `product_paths` when those artifacts are written.
5. Keep schema_version bumped if fields change incompatibly.

## Failure modes

See `reference/failure-modes.md` (rebuild every turn; paste full catalog;
ignore stale mtimes; treat log type as kind without dims).
