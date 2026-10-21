# Smart Mobility Le Mans

Dashboard link: https://ismail-achibat-td-tp-bi.github.io/TD2/

Live monitoring of Le Mans Métropole's SETRAM bus and tram network, built to support one
question: *with limited resources, where and when should public transport be improved first?*

Two automated pipelines feed one dashboard (GitHub Pages, `docs/`):

| Pipeline | Script | Schedule | Output |
|---|---|---|---|
| Static reference | `scripts/build_static_reference.py` | 1st of each month | `data/gtfs_static/` (scheduled trips per stop, routes, commune supply), `docs/data/commune_supply.json` |
| Real-time snapshot | `scripts/collect_snapshot.py` | every 15 min | `data/history/` (per-route delays, alerts), `docs/data/current_state.json` |

Sources (open data, no API key): SETRAM GTFS and GTFS-RT via transport.data.gouv.fr, plus
INSEE commune populations (`data/reference/communes.csv`), which stand in for demand.

**Supply vs. population:** scheduled stop-visits per 1,000 residents on a representative
weekday, per commune. Each stop is assigned to its nearest commune centre.
