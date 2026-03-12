# Victoria Stack (No Grafana)

This branch runs a Grafana-free observability stack focused on metrics and logs:

- `victoria-metrics` for metrics storage and querying
- `victoria-logs` for logs storage and querying
- `alloy` as the collection agent (Docker logs + cAdvisor container metrics + node-exporter-style host metrics)

## Start

```bash
docker compose up -d
```

## Stop

```bash
docker compose down
```

## Endpoints

- VictoriaMetrics UI: `http://localhost:8428/vmui`
- VictoriaMetrics metrics list: `http://localhost:8428/vmui/#/metrics`
- VictoriaLogs UI: `http://localhost:9428/select/vmui`
- Alloy UI: `http://localhost:12345`

## Ingestion Wiring

- Alloy logs -> VictoriaLogs Loki API:
  - `http://victoria-logs:9428/insert/loki/api/v1/push`
- Alloy metrics (cAdvisor + node-exporter-style) -> VictoriaMetrics remote_write API:
  - `http://victoria-metrics:8428/api/v1/write`

## Quick Checks

Metrics instant query:

```bash
curl -s http://localhost:8428/prometheus/api/v1/query -d 'query=up'
```

Node metrics sanity checks:

```bash
curl -s http://localhost:8428/prometheus/api/v1/query -d 'query=node_uname_info'
curl -s http://localhost:8428/prometheus/api/v1/query -d 'query=100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)'
curl -s http://localhost:8428/prometheus/api/v1/query -d 'query=(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100'
```

Logs query API:

```bash
curl -sG 'http://localhost:9428/select/logsql/query' \
  --data-urlencode 'query=* | limit 20'
```

## Notes

- Grafana/Tempo/Mimir/Loki provisioning was intentionally removed from this branch.
- This setup keeps Docker logs, cAdvisor container metrics, and node-exporter-style host metrics.
