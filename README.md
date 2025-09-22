# Prometheus + Grafana Monitoring Stack

A Docker Compose setup for a self-contained monitoring stack:
- Prometheus for metrics collection
- Node Exporter for host metrics
- cAdvisor for container metrics
- Grafana for dashboards and visualization
- Alertmanager for alert routing
- Fake SMTP server (Mailcatcher-compatible) for testing alert emails

## Prerequisites
- Docker and Docker Compose installed
- Ports available on your host:
    - Prometheus: 9090
    - Node Exporter: 9100
    - cAdvisor: 8080
    - Grafana: 3000
    - Alertmanager: 9093
    - Fake SMTP UI: 1080, SMTP: 1025

## Quick Start
- Start the stack:
    - docker compose up -d
- Stop the stack:
    - docker compose down
- View logs:
    - docker compose logs -f
- Recreate after config changes:
    - docker compose up -d --force-recreate

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
- docker-compose.yml — service definitions
- LICENSE — license file

## Data Persistence
- Prometheus data: persisted in the docker volume prometheus_data
- Grafana data: persisted in the docker volume grafana_data
- You can back up volumes with:
    - docker run --rm -v prometheus_data:/data -v "$PWD":/backup alpine tar czf /backup/prometheus_data.tar.gz -C / data
    - docker run --rm -v grafana_data:/data -v "$PWD":/backup alpine tar czf /backup/grafana_data.tar.gz -C / data

## Configuration
- Prometheus main config: prometheus/prometheus.yml
- Prometheus rules: prometheus/rules/*.yml
- Grafana provisioning:
    - grafana/provisioning/datasources/datasources.yml
    - grafana/provisioning/dashboards/dashboards.yml
- Grafana dashboards: grafana/dashboards/*.json
- Alertmanager config: alertmanager/config.yml

After editing Prometheus configs or rules, you can hot-reload Prometheus without restarting the container:
- curl -X POST http://localhost:9090/-/reload

## Environment and Defaults
- Grafana admin user: admin
- Grafana admin password: admin
- Sign-ups: disabled
- Prometheus retention: 15 days

Change sensitive defaults for production usage.

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

## Security Notes
- This stack is for local/dev environments by default.
- Do not expose services publicly without:
    - Securing Grafana with strong credentials and HTTPS
    - Restricting access to Prometheus, cAdvisor, and Node Exporter
    - Hardening Alertmanager and SMTP configuration

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

## License
See LICENSE file for details.
