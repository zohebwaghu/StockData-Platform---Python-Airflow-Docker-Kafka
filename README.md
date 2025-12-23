Stock Data Platform (Airflow + Kafka + TimescaleDB)
====================================================

Overview
--------
End-to-end stock data pipeline with streaming (Kafka) and orchestration (Airflow) landing data into TimescaleDB/PostgreSQL. Producer fetches live prices via yfinance and streams to Kafka; consumer writes into fact tables. Airflow manages batch loads and aggregates.

Stack
-----
- Docker Compose: spins up TimescaleDB, Kafka/Zookeeper, Airflow webserver + scheduler, producer, consumer.
- Airflow: DAGs under `Dags/` for dimension loads and aggregates.
- Kafka: topic `stock-data` (producer → consumer).
- DB: `stockdw` (user `data226`, password `12345678`).

Quick start (Docker)
--------------------
1) Prereqs: Docker Desktop running.  
2) From project root:
```
docker compose up --build
```
3) Airflow UI: http://localhost:8081 (user/pass: admin / admin). Unpause DAGs as needed.  
4) DB access: `localhost:5432`, user `data226`, password `12345678`, db `stockdw`.  
5) Kafka broker: `localhost:9092` (internal name `stock-data-platform-kafka:9092`). Topic `stock-data`.

Project layout
--------------
- `docker-compose.yml` – services wiring and Airflow bootstrap.
- `Dockerfile.producer` / `Dockerfile.consumer` – streaming components.
- `kafka_to_postgres.py` – consumer logic (Kafka → TimescaleDB).
- `live_from_kafka.py` – producer logic (yfinance → Kafka).
- `Dags/` – Airflow DAGs for dims/facts/aggregates; includes sample CSVs.
- `SQL/` – SQL helpers (e.g., monthly aggregates).
- `requirements.txt` – Python deps for Airflow images and local dev.

Operating notes
---------------
- Data directory `data/` (mounted volume) is git-ignored.
- Credentials are dev-only; switch to secrets/env vars for any real deployment.
- If yfinance rate-limits, producer backs off and retries.

Local dev without Docker (optional)
-----------------------------------
```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python live_from_kafka.py      # producer (needs Kafka running)
python kafka_to_postgres.py    # consumer (needs Kafka + DB running)
```

Next steps
----------
- Add CI for lint/tests and Docker image builds.
- Add data quality checks (Great Expectations/dbt tests).
- Expose dashboards (e.g., Superset/Metabase) over the warehouse.


