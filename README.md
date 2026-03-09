# NYC Taxi Data Pipeline (Docker + Postgres + Python)

![Python](https://img.shields.io/badge/python-3.13-blue)
![Docker](https://img.shields.io/badge/docker-compose-2496ED)
![PostgreSQL](https://img.shields.io/badge/postgresql-18-336791)
![Project Type](https://img.shields.io/badge/project-data%20engineering-success)

A portfolio project that demonstrates an end-to-end data ingestion workflow: downloading real NYC Yellow Taxi trip data, streaming it in chunks, and loading it into PostgreSQL using a containerized Python pipeline.

## Project Snapshot

- Built a reproducible local data platform with Docker Compose (`Postgres + pgAdmin`).
- Implemented a parameterized ingestion CLI in Python (`click`, `pandas`, `SQLAlchemy`, `psycopg`).
- Used chunked loading to handle larger datasets efficiently.
- Packaged the ingestion job in Docker for consistent execution across environments.

## Why This Project

This project showcases core data engineering fundamentals:
- Environment reproducibility
- Data ingestion from an external source
- Type-aware parsing and table loading
- Practical local orchestration and database inspection

## Tech Stack

- Python 3.13
- Pandas
- SQLAlchemy + Psycopg
- PostgreSQL 18
- pgAdmin 4
- Docker / Docker Compose
- uv (dependency + lockfile management)

## Architecture

1. `pgdatabase` (Postgres) runs via Docker Compose.
2. `ingest_data.py` pulls `yellow_tripdata_YYYY-MM.csv.gz` from DataTalksClub releases.
3. Data is parsed and streamed in chunks to Postgres.
4. `pgadmin` provides UI-based data exploration.

## Repository Layout

- `pipeline/ingest_data.py` - Main ingestion CLI.
- `pipeline/docker-compose.yaml` - Postgres + pgAdmin services and volumes.
- `pipeline/Dockerfile` - Containerized runtime for ingestion script.
- `pipeline/pyproject.toml` - Python dependencies.
- `pipeline/pipeline.py` - Simple parquet-output example script.

## Quick Demo (Docker)

### 1. Start services

```bash
cd pipeline
docker compose -f docker-compose.yaml up -d
```

- Postgres: `localhost:5432`
- pgAdmin: `http://localhost:8085`

Default credentials:
- Postgres user/password: `root` / `root`
- Database: `ny_taxi`
- pgAdmin user/password: `admin@admin.com` / `root`

### 2. Build ingestion image

```bash
cd pipeline
docker build -t taxi-ingest .
```

### 3. Ingest one month

```bash
cd pipeline
docker run --rm \
  taxi-ingest \
  --pg-user root \
  --pg-pass root \
  --pg-host host.docker.internal \
  --pg-port 5432 \
  --pg-db ny_taxi \
  --target-table yellow_taxi_data \
  --year 2021 \
  --month 1 \
  --chunksize 100000
```

## Local Run (without Docker image)

```bash
cd pipeline
uv sync --locked
uv run python ingest_data.py --year 2021 --month 1
```

Common flags:
- `--pg-user`, `--pg-pass`, `--pg-host`, `--pg-port`, `--pg-db`
- `--target-table`
- `--chunksize`

## Key Implementation Notes

- First chunk creates/replaces target table schema.
- Remaining chunks append rows.
- Postgres + pgAdmin state is persisted with named Docker volumes.
- Container build is lockfile-driven (`uv sync --locked`) for reproducibility.

## Troubleshooting

- `connection refused`:
  - Confirm services are up with `docker compose -f pipeline/docker-compose.yaml ps`.
  - Ensure host port `5432` is not already in use.
- Ingestion container cannot reach DB:
  - Use `--pg-host host.docker.internal`.
- Need a clean reset:
  - `docker compose -f pipeline/docker-compose.yaml down -v`

## Next Improvements

- Add incremental loading strategy (upsert/deduplication).
- Add data quality checks (null rates, schema validation).
- Add orchestration (Prefect/Airflow) and scheduled runs.
- Add analytics models and dashboard layer.
