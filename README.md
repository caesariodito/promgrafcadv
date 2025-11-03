LGTM + Alloy Docker Stack

APM endpoints (OTLP):
- OTLP: `http://localhost:4318` (HTTP), `localhost:4317` (gRPC)

This repository provides a ready‑to‑run local LGTM stack (Loki, Grafana, Tempo, Mimir) with Grafana Alloy replacing Prometheus, the OpenTelemetry Collector, and Promtail. Alloy accepts OTLP, generates RED spanmetrics, scrapes cAdvisor and Node Exporter, remote‑writes to Mimir, and tails Docker logs to Loki.

What’s included
- Grafana at http://localhost:6969 (admin/admin)
- Tempo (traces): `tempo:3200`; Alloy exports traces to Tempo
- Mimir (metrics store): `mimir:9009`; Alloy remote_writes Prometheus metrics
- Loki (logs): `loki:3100`; Alloy ships Docker logs
- Alloy (ingest/scrape/ship): OTLP gRPC `4317`, OTLP HTTP `4318`, spanmetrics `8889`
- cAdvisor and Node Exporter for host/container metrics
- Grafana provisioning for datasources and a starter RED dashboard

Run
1) Start: `docker compose up -d --remove-orphans`
2) Open Grafana: http://localhost:6969 (user: `admin`, pass: `admin`)
3) Dashboards: the starter dashboard “OTel RED Metrics” is pre‑provisioned.

Send test telemetry
- Traces: point your app’s OTLP exporter at `http://localhost:4318` (HTTP) or `localhost:4317` (gRPC). Alloy receives and forwards to Tempo.
- Metrics: Alloy emits spanmetrics at `:8889`, scrapes cAdvisor/Node Exporter, and remote‑writes to Mimir.
- Logs: Alloy tails Docker container logs (`/var/lib/docker/containers/*/*-json.log`) and ships to Loki.

File map
- `docker-compose.yaml` — service definitions for the whole stack
- `alloy-config.river` — Alloy config replacing Prometheus/Collector/Promtail
- `loki-config.yaml`, `mimir-config.yaml`, `tempo-config.yaml` — storage/runtime config
- `grafana/provisioning/datasources` — prewired datasources (Mimir, Loki, Tempo, Local Prometheus)
- `grafana/provisioning/dashboards` — dashboard provider
- `grafana/dashboards` — starter dashboards

Notes
- Prometheus service has been removed; use the “Mimir” datasource for PromQL queries. The provisioned “Local Prometheus” datasource will be offline unless you re‑enable Prometheus.
- Tempo <-> Loki linking is enabled; if your logs include a `trace_id`/`traceID` field, Grafana links traces ↔ logs automatically.
