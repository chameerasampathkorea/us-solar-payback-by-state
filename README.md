# US residential solar payback by state

Break-even periods for a 6 kW residential solar system in all 50 US states,
calculated with **no federal tax credit** following the expiry of Section 25D on
31 December 2025.

Most published payback figures still assume the 30% credit, which makes them
roughly three to five years shorter than the current position.

## Range

| | State | Years |
|---|---|---|
| Fastest | Hawaii | 4.1 |
| | New York | 6.2 |
| | Illinois | 6.8 |
| | Massachusetts | 7.0 |
| Slowest | Idaho | 18.1 |
| | Louisiana | 20.5 |
| | Alabama | 24.5 |

Alabama is the outlier because Alabama Power bills solar customers $5.41 per kW
per month — about $390 a year on a 6 kW system — a charge a federal judge upheld
in March 2026.

## Inputs

| Field | Source |
|---|---|
| `annualKwh` | NREL PVWatts v8, largest metro per state, south-facing at 20° |
| `retailRate` | EIA API v2, residential, latest month |
| `exportFactor` | The serving utility's own tariff, as a fraction of retail |
| `nettingPeriod` | `instantaneous`, `monthly` or `annual` |
| `costPerWatt` | EnergySage / SEIA, with per-state overrides |
| incentives | State rebates, tax credits and SREC prices via DSIRE |
| `federalItcPct` | 0. Section 25D expired 31 December 2025 |

Every state carries `sourceUrl` and `checked`, recording what the export terms
were verified against and when.

## The variable most comparisons omit

`nettingPeriod` matters as much as `exportFactor`, and almost no published
comparison records it. Before a utility compensates exported power it nets
exports against imports. Over a monthly billing period, power exported at midday
offsets power drawn at 7pm at the full retail rate, and only the month-end
surplus is discounted. Over fifteen-minute intervals, almost nothing offsets.

Iowa and Oregon both credit exports at 100% of retail. Iowa nets in fifteen-minute
intervals, Oregon monthly — and Oregon, with weaker irradiance, still pays back
faster. Nevada credits at only 75% of retail but nets monthly, and beats Iowa's
nominal 100%.

## Structure

```json
{
  "updated": "2026-09-04",
  "assumptions": { "systemKw": 6.0, "federalItcPct": 0.0, "...": "..." },
  "coverage": { "published": 50, "totalStates": 50 },
  "sources": { "...": "..." },
  "states": [
    {
      "code": "HI",
      "name": "Hawaii",
      "annualKwh": 9730,
      "retailRate": 0.52,
      "exportFactor": 0.32,
      "nettingPeriod": "monthly",
      "netCost": 19012,
      "yearOneSaving": 4544,
      "paybackSimple": 4.2,
      "paybackAccurate": 4.1,
      "netReturn25": 128877,
      "sourceUrl": "https://www.hawaiianelectric.com/...",
      "checked": "2026-08-26"
    }
  ]
}
```

## Use

```python
import json, urllib.request
d = json.load(urllib.request.urlopen("https://energycostmap.com/states.json"))
for s in sorted(d["states"], key=lambda x: x["paybackAccurate"] or 99)[:5]:
    print(s["name"], s["paybackAccurate"])
```

The file is refreshed monthly as EIA rates change, and whenever a state tariff
does. It is the same file the site is built from, not a separate export, so the
two cannot disagree.

## Method

Full formula, assumptions and stated omissions:
https://energycostmap.com/methodology/

Per-state reasoning, including the tariff detail behind each `exportFactor`:
https://energycostmap.com/

## Licence

CC BY 4.0. Free for any use including commercial and republication, with
attribution to energycostmap.com.

## Known limitations

- One utility is modelled per state, normally the largest by residential
  customers. Cooperatives and municipal utilities frequently differ.
- The retail rate is a state average from the EIA, not a specific tariff. In
  states served by several utilities this is the largest source of error.
- Self-consumption is assumed rather than measured, and is the weakest
  assumption in the model.
- Loan interest and dealer fees, the resale premium, leases and PPAs, batteries
  and roof replacement are all excluded. Each is set out in the method.
