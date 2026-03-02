# mipit-observability

Observability stack for the MiPIT PoC — Prometheus, Grafana dashboards, Jaeger tracing, and OpenTelemetry Collector.

## Structure

```
mipit-observability/
├── prometheus/
│   └── prometheus.yml               # Scrape config (core, adapters, RabbitMQ, Postgres)
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── datasources.yaml     # Prometheus + Jaeger datasources
│   │   └── dashboards/
│   │       └── dashboards.yaml      # File-based dashboard provisioner
│   └── dashboards/
│       ├── mipit-overview.json       # Main overview dashboard
│       ├── mipit-latency.json        # Latency by stage (p50/p95/p99)
│       └── mipit-rails.json          # Per-rail metrics (PIX vs SPEI)
├── otel-collector/
│   └── otel-collector.yaml           # OpenTelemetry Collector config
└── alerting/
    └── rules.yaml                    # Prometheus alerting rules
```

## Scrape Targets

| Job                | Target                    | Port  |
|--------------------|---------------------------|-------|
| mipit-core         | `core:8080`               | 8080  |
| adapter-pix        | `adapter-pix:9100`        | 9100  |
| adapter-spei       | `adapter-spei:9100`       | 9100  |
| rabbitmq           | `rabbitmq:15692`          | 15692 |
| postgres-exporter  | `postgres-exporter:9187`  | 9187  |

## Dashboards

- **MiPIT Overview** — Total transactions, success rate, idempotency hits, p95 latency, status breakdown, rail breakdown.
- **MiPIT Latency** — Latency heatmap by stage, percentile time series (p50/p95/p99), per-adapter latency histograms.
- **MiPIT Rails** — Success rate by rail, errors by rail, retries by adapter, top error types.

## Alerting Rules

| Alert                 | Condition                         | Severity |
|-----------------------|-----------------------------------|----------|
| HighErrorRate         | Error rate > 1% for 2 min        | warning  |
| HighLatency           | p95 latency > 2000 ms for 3 min  | warning  |
| RabbitMQQueueBacklog  | Queue > 100 messages for 5 min   | warning  |

## Expected Metrics

### Core (`mipit-core`)

| Metric                           | Type      | Labels                                |
|----------------------------------|-----------|---------------------------------------|
| `mipit_payments_total`           | Counter   | status, origin_rail, destination_rail |
| `mipit_payment_latency_ms`       | Histogram | stage                                 |
| `mipit_translation_errors_total` | Counter   | rail, error_type                      |
| `mipit_routing_decisions_total`  | Counter   | rule, destination_rail                |
| `mipit_idempotency_hits_total`   | Counter   | —                                     |

### Adapters (`adapter-pix`, `adapter-spei`)

| Metric                           | Type      | Labels        |
|----------------------------------|-----------|---------------|
| `mipit_adapter_requests_total`   | Counter   | status, rail  |
| `mipit_adapter_latency_ms`       | Histogram | rail          |
| `mipit_adapter_retries_total`    | Counter   | rail          |
| `mipit_adapter_errors_total`     | Counter   | rail, error   |

## Usage with Docker Compose

These configuration files are mounted as volumes by `mipit-infra`'s `docker-compose.yml`. See that repo for the full orchestration setup.
