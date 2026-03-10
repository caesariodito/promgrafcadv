# Victoria Stack (No Grafana)

This branch runs a Grafana-free observability stack focused on metrics and logs:

- `victoria-metrics` for metrics storage and querying
- `victoria-logs` for logs storage and querying
- `alloy` as the collection agent (Docker logs + cAdvisor container metrics)

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
- Alloy cAdvisor metrics -> VictoriaMetrics remote_write API:
  - `http://victoria-metrics:8428/api/v1/write`

## Quick Checks

Metrics instant query:

```bash
curl -s http://localhost:8428/prometheus/api/v1/query -d 'query=up'
```

Logs query API:

```bash
curl -sG 'http://localhost:9428/select/logsql/query' \
  --data-urlencode 'query=* | limit 20'
```

## Notes

- Grafana/Tempo/Mimir/Loki provisioning was intentionally removed from this branch.
- This setup intentionally keeps only Docker logs and cAdvisor metrics to reduce moving parts.
