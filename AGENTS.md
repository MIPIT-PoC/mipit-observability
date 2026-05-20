# AGENTS.md

<purpose>
This repository contains the observability configuration for MiPIT-PoC: Prometheus scrape targets, Grafana datasources and dashboards, OpenTelemetry Collector pipeline, and alerting rules.

It is responsible for:
- Prometheus configuration with 5 scrape targets (mipit-core, adapter-pix, adapter-spei, rabbitmq, postgres-exporter),
- Grafana provisioning: 2 datasources (Prometheus, Jaeger) and 3 dashboards (overview, latency, rails),
- OpenTelemetry Collector pipeline: OTLP receivers → batch processor → Jaeger (traces) + Prometheus (metrics),
- Alerting rules for high error rate, high latency, and RabbitMQ queue backlog.

Treat shipped configuration as the primary source of truth.
When config and documents disagree, prefer:
1. current repo configuration files,
2. current architecture/design artifacts in mipit-docs.
</purpose>

<project_scope>
This repo contains configuration files only — no application code.
It provides the observability stack for the PoC demo.
Dashboards should visualize real metrics from mipit-core and adapters.
Alerts are informational for the PoC, not production-grade.
</project_scope>

<instruction_priority>
- User instructions override default style, tone, and initiative preferences.
- Safety, honesty, privacy, and permission constraints do not yield.
</instruction_priority>

<workflow>
  <phase name="clarify">
  - Before changes, clarify: which component (Prometheus, Grafana, OTel Collector, alerting)?
  - Does the change affect scrape targets, dashboard panels, trace pipelines, or alert thresholds?
  </phase>

  <phase name="research">
  - Check prometheus/prometheus.yml for current scrape targets and intervals.
  - Check grafana/dashboards/*.json for current panel definitions.
  - Check otel-collector/otel-collector.yaml for receiver/processor/exporter config.
  - Check alerting/rules.yaml for current alert rules.
  - Cross-reference metric names with mipit-core and adapter metrics.ts files.
  </phase>

  <phase name="implement">
  - Keep Prometheus config aligned with actual service ports and metric endpoints.
  - Keep Grafana dashboards using real metric names from mipit-core/adapter metrics.ts.
  - Keep OTel Collector config minimal and correct for the PoC stack.
  - Keep alert thresholds reasonable for PoC load (not production tuned).
  </phase>

  <phase name="verify">
  - After changes, verify Prometheus targets are UP in Prometheus UI (:9090/targets).
  - Verify Grafana dashboards load without errors in Grafana UI (:3000).
  - Verify OTel Collector receives traces (check Jaeger UI :16686).
  - Verify alerts fire correctly under simulated conditions.
  </phase>
</workflow>

<observability_rules>
- Metric names must match exactly what mipit-core and adapters export (mipit_payments_total, mipit_payment_latency_ms, etc.).
- Scrape targets use Docker service hostnames (core:8080, adapter-pix:9100, etc.).
- Grafana dashboards use Prometheus as default datasource and Jaeger for trace links.
- OTel Collector receives OTLP on ports 4317 (gRPC) and 4318 (HTTP), exports to Jaeger and Prometheus.
- Alert rules use PromQL queries against mipit_* metrics.
</observability_rules>

<default_commands>
- No build/run commands — these configs are mounted as volumes in Docker Compose.
- Validate Prometheus config: `promtool check config prometheus/prometheus.yml`
- Validate alert rules: `promtool check rules alerting/rules.yaml`
</default_commands>
