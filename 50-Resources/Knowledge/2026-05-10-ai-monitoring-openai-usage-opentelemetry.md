---
type: reference
status: active
created: 2026-05-10
updated: 2026-05-10
source_url: https://youtu.be/jF72yDvAPQ4?si=x5HwlKyO2WxGtFX7
source_type: youtube
source_author: ''
source_title: Monitoring OpenAI Usage with OpenTelemetry, Alloy, Prometheus, and Grafana
source_published: ''
source_duration: 7:52 video
topics:
- ai
- tech
- automation
summary: This video guides users through setting up a comprehensive monitoring solution
  for OpenAI (OpenClaw) usage. It demonstrates how to utilize OpenTelemetry, Alloy,
  Prometheus, and Grafana to track key metrics like token usage and cost, providing
  essential visibility into OpenAI activity.
key_points:
- Prerequisites include OpenClaw and Docker installed and running.
- Configure Alloy to act as an OpenTelemetry receiver (HTTP on port 4318) and remote
  write metrics to Prometheus.
- Enable the OpenTelemetry plugin under the 'diagnostics' key in OpenClaw's configuration
  to export metrics.
- Prometheus is configured to enable remote writes to receive metrics from Alloy.
- Visualize OpenClaw metrics using Grafana, with a starter dashboard providing insights
  into token distribution, cost, and usage patterns.
relevance: 3
access_count: 2
last_accessed: '2026-05-20'
ingested_by: kb-ingest.py
kb_id: kb-20260510-a325d4
---
# Monitoring OpenAI Usage with OpenTelemetry, Alloy, Prometheus, and Grafana

## Summary
This video guides users through setting up a comprehensive monitoring solution for OpenAI (OpenClaw) usage. It demonstrates how to utilize OpenTelemetry, Alloy, Prometheus, and Grafana to track key metrics like token usage and cost, providing essential visibility into OpenAI activity.

## Key Points
- Prerequisites include OpenClaw and Docker installed and running.
- Configure Alloy to act as an OpenTelemetry receiver (HTTP on port 4318) and remote write metrics to Prometheus.
- Enable the OpenTelemetry plugin under the 'diagnostics' key in OpenClaw's configuration to export metrics.
- Prometheus is configured to enable remote writes to receive metrics from Alloy.
- Visualize OpenClaw metrics using Grafana, with a starter dashboard providing insights into token distribution, cost, and usage patterns.

## Notable
> This is all brought to you by the power of open telemetry, a standard for getting open source observability.

## My Notes
<!-- Add personal notes here -->

## Source
[Monitoring OpenAI Usage with OpenTelemetry, Alloy, Prometheus, and Grafana](https://youtu.be/jF72yDvAPQ4?si=x5HwlKyO2WxGtFX7) |  | 
