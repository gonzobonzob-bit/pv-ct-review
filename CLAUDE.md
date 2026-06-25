# pv-ct-review

Single-file browser diagnostic tool for solar PV CT data. Deployed to GitHub Pages:
https://gonzobonzob-bit.github.io/pv-ct-review/

## Workflow

The user iterates on features in the Claude.ai web app, downloads the result, and brings it here.
The CSS in the downloaded file is often the old navy/amber theme — always check the diff before overwriting.

**Standard flow:**
1. User brings updated `index.html` from Downloads
2. Diff against current project file
3. If CSS regressed: merge JS/HTML only, keep existing CSS
4. Commit with the user's exact commit message, push to main
5. Confirm Pages deploy and give URL
6. Remind user to re-upload the committed file to their Claude.ai Project

## CSS design system — preserve exactly

Font: Inter (Google Fonts) + system fallback. DO NOT replace with --sans or system-only stack.

CSS variables (defined in `:root`):
- `--blue` #1d4ed8, `--blue-light` #eff6ff, `--blue-mid` #bfdbfe
- `--red` #dc2626, `--red-light` #fef2f2, `--red-mid` #fecaca
- `--green` #16a34a, `--green-light` #f0fdf4, `--green-mid` #bbf7d0
- `--amber` #b45309, `--amber-light` #fffbeb, `--amber-mid` #fde68a
- `--bg` #f1f5f9, `--card` #fff, `--ink` #0f172a, `--ink-2` #334155
- `--muted` #64748b, `--line` #e2e8f0, `--line-2` #f1f5f9
- `--mono` ui-monospace stack, `--shadow` subtle box-shadow

Status banners: border-left coloring on light background (NOT solid fills).
Nav tab active state: `--blue` (not `--navy`).
Do NOT introduce `--navy`, `--navy-light`, `--sans`, `--red-bg`, `--green-bg`, `--amber-bg`.

## Current features

**Tab 1 — System CT Review**
- CSV upload (drag-and-drop supported)
- Detects schema: load_with_solar, tesla_derived, legs_only
- Checks: negative-PV, Grid=Load−PV balance, cross-talk correlation (differenced Pearson), flatline/dropout (absolute + relative-span), per-day production anomaly, per-leg consumption flatline (legs_only mode)
- `INSUFFICIENT_DATA` status returned when < 4 usable points

**Tab 2 — Microinverter Troubleshooting**
- Side-by-side CSV upload for two inverters
- Groups output by time-of-day, finds morning/afternoon gap skew
- Electrical signature check: shading = current collapses, voltage holds
- Five verdicts: no_meaningful_gap, likely_shading, directional_gap_not_shading_signature, directional_gap_no_telemetry, uniform_gap_check_hardware

## Key JS invariants — do not simplify away

- `parseTimeSafe()` has a special case for GraphList `"Jun 25, 2026 (15:15)"` format — native `Date()` silently returns midnight for these
- `checkFlatline()` uses both absolute (`minHours`) AND relative (`minFractionOfSpan=0.85`) thresholds — partial-day exports need the relative path
- `renderReport()` handles three statuses via a `BANNER` lookup object
- `parseMicroCSV` uses `parseCSV_shared` (not `parseCSV`) to avoid redeclaring the function
- Everything runs client-side; no fetch/API calls anywhere

## Deploy

GitHub Pages, branch `main`, root `/`. Push to main triggers automatic redeploy (~30s).
Check with: `gh run watch $(gh run list --limit 1 --json databaseId -q '.[0].databaseId') --exit-status`
