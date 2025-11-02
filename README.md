LGTM + OTEL Docker Stack
APM endpoints (additional):
- Zipkin: `http://localhost:9411/api/v2/spans`
- Jaeger (Thrift HTTP): `http://localhost:14268/api/traces`
- Jaeger (gRPC): `localhost:14250`
- OTLP logs: `http://localhost:4318/v1/logs` (HTTP), `localhost:4317` (gRPC)

This repository provides a ready‑to‑run local LGTM stack (Loki, Grafana, Tempo, Mimir) with an OpenTelemetry Collector, Prometheus, cAdvisor, Node Exporter, and Promtail — adapted from Grafana's docker‑otel‑lgtm example but flattened into the project root.

What’s included
- Grafana at http://localhost:6969 (admin/admin)
- Tempo (traces): `tempo:3200` (OTLP gRPC at `tempo:4317`, HTTP at `tempo:4318`)
- Mimir (metrics store): `mimir:9009`
- Prometheus (scraper + remote_write to Mimir): `prometheus:9090`
- Loki (logs): `loki:3100`
- OpenTelemetry Collector (ingest): OTLP gRPC `4317`, OTLP HTTP `4318`
- cAdvisor and Node Exporter for host/container metrics
- Grafana provisioning for datasources and a starter RED dashboard

Run
1) Start: `docker compose up -d`
2) Open Grafana: http://localhost:6969 (user: `admin`, pass: `admin`)
3) Dashboards: the starter dashboard “OTel RED Metrics” is pre‑provisioned.

Send test telemetry
- Traces: point your app’s OTLP exporter at `http://localhost:4318` (HTTP) or `localhost:4317` (gRPC). Tempo receives via the Collector and is queryable in Grafana’s Explore.
- Metrics: Prometheus scrapes the Collector (spanmetrics at `:8889`) and remote‑writes to Mimir for Grafana queries.
- Logs: Promtail tails Docker container logs and ships to Loki.

File map
- `docker-compose.yaml` — service definitions for the whole stack
- `otel-collector-config.yaml` — OTEL pipelines for traces + span‑derived metrics
- `prometheus-scrape-config.yaml` — scrape targets + remote_write to Mimir
- `loki-config.yaml`, `mimir-config.yaml`, `tempo-config.yaml` — storage/runtime config
- `grafana/provisioning/datasources` — prewired datasources (Mimir, Loki, Tempo)
- `grafana/provisioning/dashboards` — dashboard provider
- `grafana/dashboards` — starter dashboards

Notes
- Mimir datasource is configured at `http://mimir:9009/prometheus` for queries.
- Tempo <-> Loki linking is enabled; if your logs include a `trace_id`/`traceID` field, Grafana links traces ↔ logs automatically.
- On Windows hosts, Promtail’s Docker log path mount may need adjustment if Docker Desktop stores logs in a non‑default location.
