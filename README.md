# Monitoring Stack: Prometheus + Grafana + Loki

A full observability stack runnable locally via Docker Compose — metrics,
dashboards, logs, and alerting in one command. Modeled on the monitoring
infrastructure I built that reduced incident response time by 60% across
20+ production services.

## Problem

Without centralized monitoring, finding out a server is struggling means
either a user complaining first or manually SSHing in to check. There's no
historical view of what happened before an incident.

## Solution

Four components working together:

- **Prometheus** scrapes time-series metrics (CPU, memory, uptime) on a
  15-second interval and evaluates alert rules continuously
- **node-exporter** exposes host-level metrics to be scraped — stands in for
  "the production servers" in this demo
- **Loki** aggregates logs cheaply (indexes only labels, not full text,
  unlike Elasticsearch) — **Promtail** is the agent that ships logs into it
- **Grafana** is the single dashboard surface for both metrics and logs,
  pre-wired to both data sources via provisioning files (no manual setup)

## Architecture

```
node-exporter ──scrape──▶ Prometheus ──┐
                                         ├──▶ Grafana (dashboards)
/var/log ──▶ Promtail ──push──▶ Loki ──┘
```

## Tech Used

Prometheus, Grafana, Loki, Promtail, node-exporter, Docker Compose, PromQL (alert rules)

## Usage

```bash
docker compose up -d
```

- Prometheus: http://localhost:9090 — check **Status → Targets** to confirm
  both targets are `UP`
- Grafana: http://localhost:3000 (login: `admin` / `admin`) — Prometheus and
  Loki data sources are already connected
- Loki logs: explorable from inside Grafana's **Explore** tab, data source = Loki

## Alert Rules Included

- `InstanceDown` — fires if a target is unreachable for 1 minute
- `HighCPUUsage` — fires if CPU stays above 85% for 5 minutes
- `HighMemoryUsage` — fires if memory stays above 90% for 5 minutes

These mirror the kind of proactive alerting that catches issues before
users report them — the actual mechanism behind the "60% reduction in
incident response time" result in production.

## Notes

This demo uses `node-exporter` as a stand-in target. In the production setup
this scaled to 20+ real services, with alert rules routed to Slack via
Alertmanager (not included here to keep the demo self-contained).

## Cleanup

```bash
docker compose down -v
```
