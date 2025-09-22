# Prometheus + Grafana + Jaeger Monitoring and Tracing Stack

A Docker Compose setup for a self-contained monitoring stack:
- Prometheus for metrics collection
- Node Exporter for host metrics
- cAdvisor for container metrics
- Grafana for dashboards and visualization
- Alertmanager for alert routing
- Fake SMTP server (Mailcatcher-compatible) for testing alert emails
- Jaeger (OpenTelemetry Collector + Query/UI) for distributed tracing, with span metrics exported to Prometheus for Grafana dashboards

## Prerequisites
- Docker and Docker Compose installed
- Ports available on your host:
  - Prometheus: 9090
  - Node Exporter: 9100
  - cAdvisor: 8080
  - Grafana: 3000
  - Alertmanager: 9093
  - Fake SMTP UI: 1080, SMTP: 1025
  - Jaeger UI: 16686
  - OpenTelemetry (OTLP) endpoints: 4317 (gRPC), 4318 (HTTP)
    // ... existing code ...

## Services & URLs
- Prometheus: http://localhost:9090
- Node Exporter: http://localhost:9100/metrics
- cAdvisor: http://localhost:8080
- Grafana: http://localhost:3000
  - Default login: admin / admin
  - Sign-ups disabled; change the admin password after first login
- Alertmanager: http://localhost:9093
- Fake SMTP (Mailcatcher UI): http://localhost:1080
  - SMTP endpoint: localhost:1025 (no auth, for testing only)
- Jaeger UI (traces): http://localhost:16686
- OTLP ingest endpoints (for your apps to send traces/metrics/logs):
  - gRPC: http://localhost:4317
  - HTTP: http://localhost:4318
    // ... existing code ...

## Directory Structure
- prometheus/
  - prometheus.yml — scrape configs and alerting targets
  - rules/ — alerting and recording rules
- grafana/
  - provisioning/
    - datasources/ — preconfigured Prometheus datasource
    - dashboards/ — dashboard provisioning definitions
  - dashboards/ — JSON dashboards (imported automatically)
- alertmanager/
  - config.yml — routing, receivers (email via the fake SMTP)
- jaeger/
  - config.yml — OpenTelemetry Collector/Jaeger configuration (OTLP receivers, spanmetrics, storage/query)
- docker-compose.yml — service definitions
- LICENSE — license file
  // ... existing code ...

## Configuration
- Prometheus main config: prometheus/prometheus.yml
- Prometheus rules: prometheus/rules/*.yml
- Grafana provisioning:
  - grafana/provisioning/datasources/datasources.yml
  - grafana/provisioning/dashboards/dashboards.yml
- Grafana dashboards: grafana/dashboards/*.json
- Alertmanager config: alertmanager/config.yml
- Jaeger/OTel Collector config: jaeger/config.yml
  - Receives traces via OTLP on 4317 (gRPC) and 4318 (HTTP)
  - Exports span metrics to Prometheus (usable for tracing SLO dashboards)
  - Provides Jaeger query/UI over the configured UI port

After editing Prometheus configs or rules, you can hot-reload Prometheus without restarting the container:
- curl -X POST http://localhost:9090/-/reload
  // ... existing code ...

## Common Tasks
- Check Prometheus targets: Prometheus UI → Status → Targets
- Explore metrics:
  - Prometheus UI → Graph → execute queries like node_cpu_seconds_total
  - cAdvisor: container metrics by container name/ID
- Import/adjust dashboards:
  - Place JSON files in grafana/dashboards
  - Ensure they’re referenced in grafana/provisioning/dashboards/dashboards.yml
- Test alert emails:
  - Configure a receiver in alertmanager/config.yml pointing to SMTP host mailcatcher on port 1025
  - Trigger a test alert or use amtool to send a test notification
  - Verify email in http://localhost:1080
- Send traces to Jaeger (via OTLP):
  - Point your OpenTelemetry SDK/agent to the OTLP endpoint:
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318 (HTTP) or http://localhost:4317 (gRPC)
  - Ensure tracing is enabled in your app and a sampler is configured (e.g., parentbased_always_on for dev)
- Explore traces:
  - Open Jaeger UI at http://localhost:16686
  - Find your service and inspect trace timelines and spans
- Tracing metrics in Grafana:
  - Span metrics exported by the collector can be visualized via Grafana dashboards (e.g., latency, error rate, RPS)
    // ... existing code ...

## Security Notes
- This stack is for local/dev environments by default.
- Do not expose services publicly without:
  - Securing Grafana with strong credentials and HTTPS
  - Restricting access to Prometheus, cAdvisor, and Node Exporter
  - Hardening Alertmanager and SMTP configuration
  - Restricting access to Jaeger UI and OTLP ingest ports (4317/4318); prefer private networks, mTLS, and auth proxies for production
    // ... existing code ...

## Troubleshooting
- No metrics in Grafana:
  - Check Prometheus targets are UP
  - Validate datasource in Grafana (provisioned automatically)
- Prometheus config error:
  - Check prometheus.yml syntax
  - Use the /-/reload endpoint after changes
  - View logs: docker compose logs -f prometheus
- Alerts not sent:
  - Verify Alertmanager status and routes
  - Check fake SMTP UI for received messages
- Permission issues on Linux:
  - Ensure the Docker daemon has access to host paths mounted by exporters
- No traces in Jaeger:
  - Verify your app exports to the correct OTLP endpoint and protocol
  - Check sampler settings (traces may be dropped if sampling rate is 0)
  - Confirm the collector is running and receiving spans
  - Ensure time sync between services to avoid out-of-window spans

## License
See LICENSE file for details.
