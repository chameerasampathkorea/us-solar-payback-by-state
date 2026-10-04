[README.md](https://github.com/user-attachments/files/33026153/README.md)
# US residential solar payback by state

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22305313.svg)](https://doi.org/10.5281/zenodo.22305313)

Break-even periods for a 6 kW residential solar system in all 50 US states, calculated with **no federal tax credit** following the expiry of Section 25D on 31 December 2025.

Most published payback figures still assume the 30% credit, which makes them roughly three to five years shorter than the current position.

Generated from `states.json` on 2026-10-04. Every figure in this file is computed from that data; none is typed by hand.

## Range

| | State | Years |
|---|---|---|
| Fastest | Hawaii | 4.5 |
| | New York | 6.2 |
| | Illinois | 6.5 |
| | Massachusetts | 6.8 |
| Slowest | North Dakota | 22.2 |
| | Tennessee | 22.5 |
| | Alabama | 26.2 |

Hawaii is fastest because the retail rate there is 48.0c against a national picture nearer 19.1c.

Alabama is the clearest outlier at the slow end: a non-bypassable charge of $390 a year applies to solar customers there regardless of output, which no comparison that stops at the export rate will show.

## Inputs

| Field | Source |
|---|---|
| `annualKwh` | NREL PVWatts v8, largest metro per state, south-facing at 20 degrees |
| `retailRate` | EIA API v2, residential, latest month |
| `exportFactor` | The serving utility's own tariff, as a fraction of retail |
| `nettingPeriod` | `instantaneous`, `monthly` or `annual` |
| `costPerWatt` | State median, overridden per state where local pricing differs |
| incentives | State rebates, tax credits and SREC prices via DSIRE |
| `federalItcPct` | 0%. Section 25D expired 31 December 2025 |

Every state carries `sourceUrl` and `checked`, recording what the export terms were verified against and when.

## The variable most comparisons omit

`nettingPeriod` matters as much as `exportFactor`, and almost no published comparison records it. Before a utility compensates exported power it nets exports against imports. Over a monthly billing period, power exported at midday offsets power drawn at 7pm at the full retail rate, and only the month-end surplus is discounted. Over fifteen-minute intervals, almost nothing offsets.

Among the states that credit exports at the full retail rate, the netting period still separates them: Iowa (instantaneous netting, 13.6 years); Connecticut (monthly netting, 10.7 years); New York (annual netting, 6.2 years).

## Structure

```json
{
  "updated": "2026-10-04",
  "assumptions": {
    "systemKw": 6.0,
    "federalItcPct": 0.0,
    "...": "..."
  },
  "coverage": {
    "published": 50,
    "totalStates": 50,
    "awaitingVerification": []
  },
  "states": [
    {
      "code": "HI",
      "name": "Hawaii",
      "annualKwh": 9730,
      "retailRate": 0.48,
      "exportFactor": 0.32,
      "nettingPeriod": "monthly",
      "netCost": 19012.5,
      "yearOneSaving": 4193.89,
      "paybackAccurate": 4.5,
      "sourceUrl": "https://tax.hawaii.gov/geninfo/renewable/",
      "checked": "2026-08-26"
    }
  ]
}
```

`states.csv` carries the same rows, flattened, for spreadsheet use.

## Use

```python
import json, urllib.request
d = json.load(urllib.request.urlopen("https://energycostmap.com/states.json"))
for s in sorted(d["states"], key=lambda x: x["paybackAccurate"] or 99)[:5]:
    print(s["name"], s["paybackAccurate"])
```

The file is refreshed monthly as EIA rates change, and whenever a state tariff does. It is the same file the site is built from, not a separate export.

## Method

Full formula, assumptions and stated omissions: https://energycostmap.com/methodology/

Per-state reasoning, including the tariff detail behind each `exportFactor`: https://energycostmap.com/

## Licence

CC BY 4.0. Free for any use including commercial and republication, with attribution to energycostmap.com.

## Known limitations

- One utility is modelled per state, normally the largest by residential customers. Cooperatives and municipal utilities frequently differ.
- The retail rate is a state average from the EIA, not a specific tariff. In states served by several utilities this is the largest source of error.
- Self-consumption is assumed rather than measured (instantaneous 65%, monthly 85%, annual 95%) and is the weakest assumption in the model.
- Loan interest and dealer fees, the resale premium, leases and PPAs, batteries and roof replacement are all excluded. Each is set out in the method.
