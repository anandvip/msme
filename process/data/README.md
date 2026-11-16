# data/ — figures behind the briefs

One JSON file per brief, named by the brief date: `data/YYYY-MM-DD.json`.
Each file holds the key figures **exactly as printed in that brief**. Nothing is added that the brief doesn't say.
Over time these files become a dataset we can study: trends, mandi vs MSP, stocks vs buffer, credit costs, and so on.

## File shape

```json
{
  "schema_version": "1.0",
  "brief_date": "2026-11-15",
  "brief_url": "https://msme.vipulanand.in/briefs/2026-11-15.html",
  "brief_file": "briefs/2026-11-15.html",
  "compiled": "2026-11-15",
  "timezone": "Asia/Kolkata",
  "extraction_note": "…",
  "gaps": [ { "section": "grain", "note": "…", "status": "flag" } ],
  "figures": [ { …figure… } ]
}
```

## Figure fields

| Field | Type | Meaning |
|---|---|---|
| `id` | string | Stable snake_case key. **Reuse the same id for the same measure every week**. This is what makes studies possible. |
| `group` | string | `macro`, `markets`, `inputs`, `savings`, `grain_stocks`, `grain_prices`, `production`, `trade` (add new groups as needed). |
| `label` | string | Human-readable name. |
| `value` | number \| null | Single value, or the midpoint/central value if the brief gives one. `null` if the brief only gives a range. |
| `value_min`, `value_max` | number | Optional. Present when the brief gives a range. |
| `approx` | boolean | `true` when the brief writes "~" or "about". |
| `unit` | string | `%`, `pp`, `Mt`, `t`, `INR/kg`, `INR`, `USD/t`, `USD/bbl`, `USD bn`, `index pts`, `x`, `Mt/yr`. Use `INR` for rupees. |
| `as_of` | string | Date the figure refers to (`YYYY-MM-DD`, or `YYYY-MM` for a monthly print). This is not the brief date. |
| `value_text` | string | The figure as printed on the page, for audit. |
| `source` | string \| null | Source named in the brief. `null` when the brief names none. |
| `source_tier` | string \| null | `T1` official (MoSPI, RBI, BSE/NSE/CCIL), `T2` press, `T3` indicative quotes. |
| `source_url` | string \| null | Direct link to the source. `null` until a link is confirmed. Never guess one. |
| `brief_anchor` | string | Path plus section anchor in the brief where the figure appears. |
| `status` | string[] | Pills shown on the page (see below). |
| `calc` | string | Required when `status` includes `derived`: the arithmetic as shown on the brief. |
| `note` | string | Optional caveat, copied from or summarising the brief. |

### Status values

- `fixed`: confirmed print or official (T1) figure.
- `revised`: changed since the last print or policy update.
- `derived`: computed on the brief from the inputs shown.
- `flag`: uncertain, inconsistent or needs checking.
- `reported`: stated in the brief without one of the four pills above.

A figure can carry more than one, e.g. `["derived","flag"]`.

## Adding a new week

1. Publish `briefs/YYYY-MM-DD.html`.
2. Add `data/YYYY-MM-DD.json` using the same `id`s where the measure repeats.
3. Add one entry to the top of `briefs/briefs.json`. The home page reads it.
4. Add one line to the root `README.md`.

## Using the data

Every file has the same shape, so a short script can stack them:

```python
import json, glob
rows = [dict(f, brief=d["brief_date"])
        for p in sorted(glob.glob("data/*.json"))
        for d in [json.load(open(p))] for f in d["figures"]]
# e.g. filter id == "central_pool_wheat" to see stocks week by week
```
