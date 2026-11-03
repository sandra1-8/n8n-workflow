# Session 4: practice project "Three deliveries"

No/Low Code and AI-Assisted Development, Hochschule Kufstein Tirol. Synthetic data only.

An import that goes wrong twice, and how a key, "Insert or Update" and a check before saving fix it.

```
files/orders-day*.csv  ->  n8n (import)  ->  PostgreSQL (table orders, view kpi_by_store)
```

## Start

Windows:

```
copy .env.example .env
docker compose up -d
docker compose ps
```

macOS: the same, but `cp .env.example .env` in the first line. `docker compose ps` should show both services as "Up".

Open http://localhost:5678 (n8n). Stop everything with `docker compose down`.
Start again from an empty database: `docker compose down -v`, then `docker compose up -d`.

## Files

- `files/orders-day1.csv`: delivery 1, 30 clean orders
- `files/orders-day2.csv`: delivery 2, 3 orders from day 1 again (one is now cancelled) and 5 new ones
- `files/orders-day3.csv`: delivery 3, 8 orders, 3 of them break the rules
- `db/init.sql`: the table `orders` with its key and rules, and the view `kpi_by_store`
- `workflows/`: three n8n workflows to import (n8n: Workflows → ⋯ → Import from File)
- `AGENTS.md`: rules for your AI agent

## Connect n8n to PostgreSQL

n8n → Credentials → Create credential → Postgres:

| Field | Value |
|---|---|
| Host | `postgres` (the service name in docker-compose.yml, not `localhost`) |
| Database | `shop` |
| User | `app` |
| Password | the value from your `.env` |
| Port | `5432` |

Name the credential **Postgres shop**. Inside a container, `localhost` is the container itself.
Same compose file: use the service name. A database in another compose file: `host.docker.internal`.

## Look into the database

```bash
docker compose exec postgres psql -U app -d shop -c "SELECT COUNT(*) FROM orders"
docker compose exec postgres psql -U app -d shop -c "SELECT * FROM kpi_by_store"
```

## Good to know

- PostgreSQL already installed on your laptop (port 5432 taken)? Set `PG_PORT=5433` in `.env`. Nothing else changes: n8n and `docker compose exec` do not use that port.
- Changed the password in `.env` after the first start? It only applies to a new database: `docker compose down -v`, then `docker compose up -d`.
- Workflow 3 writes `files/errors.csv` only when there are rejected rows. A delivery without errors leaves the old file in place.
- "Insert or Update" needs a key (PRIMARY KEY or UNIQUE) on the column it matches on. With automatic mapping, the CSV headers must equal the table's column names.
- Edit delivery files in VS Code, not in Excel: Excel may save them with `;` and a different encoding.

- On Linux, n8n may not be allowed to write `files/errors.csv`. Fix: `chmod 777 files`.
- Same tool, same check: n8n's "Check rules" node and a Python script can do the same job. Compare effort, readability and testing.

# n8n-workflow
