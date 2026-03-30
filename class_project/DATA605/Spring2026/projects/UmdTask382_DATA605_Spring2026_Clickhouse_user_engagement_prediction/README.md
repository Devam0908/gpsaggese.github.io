# UmdTask382 — ClickHouse user engagement prediction

Containerized analytics pipeline: clickstream → ClickHouse → SQL features → ML → Grafana.

## Layout

- `docker/` — Compose stack (ClickHouse, Jupyter, Grafana), `Dockerfile`, `docker_build.sh`
- `sql/` — `01_schema.sql`, `02_feature_engineering.sql`
- `src/` — `db`, `ingestion`, `features`, `models`
- `notebooks/` — `clickhouse.API.ipynb`, `clickhouse.example.ipynb`
- `data/` — optional extra copies of CSVs
- `clickstream+data+for+online+shopping/` — **e-shop clothing 2008** (semicolon CSV + description)
- `grafana/provisioning/` — datasources and dashboards (team adds JSON/YAML)

## Dataset (e-shop clothing 2008)

File: `clickstream+data+for+online+shopping/e-shop clothing 2008.csv`  
Delimiter: `;` — columns are year/month/day, session, country, product page fields, price, etc. There is **no** global `user_id`; ingestion uses **`user_id = country × 10_000_000 + session ID`**, **`session_id = "{country}-{session}"`**, **`ts = date + order` seconds** (within-day ordering), **`event_type = click`**, **`product_id = CRC32(clothing model code)`**, **`category`** from main category (trousers / skirts / blouses / sale).

Citation (from dataset description): Łapczyński M., Białowolski S. (2013), *Studia Ekonomiczne* nr 151 — see `e-shop clothing 2008 data description.txt`.

Ingest from Python:

```python
from pathlib import Path
from ingestion.data_loader import load_eshop_clothing_2008_to_clickhouse

root = Path("..")  # project root when cwd is notebooks/
csv_path = root / "clickstream+data+for+online+shopping" / "e-shop clothing 2008.csv"
n = load_eshop_clothing_2008_to_clickhouse(csv_path)
print("rows inserted:", n)
```

## Quick start

1. `cd docker && chmod +x docker_build.sh && ./docker_build.sh && docker compose up -d`
2. Open Jupyter at port 8888 (check `docker compose logs jupyter` for token) or run notebooks locally with ClickHouse on `localhost:8123`.
3. Run `notebooks/clickhouse.API.ipynb` first, then ingest CSV and feature SQL.

## Environment

Optional: `CLICKHOUSE_HOST`, `CLICKHOUSE_PORT`, `CLICKHOUSE_USER`, `CLICKHOUSE_PASSWORD`.
