# Changelog

All notable user-visible changes to the VictoriaMetrics TOS application package.

## 1.149.0-1 — 2026-11-15

### Added
- First release. VictoriaMetrics (`victoriametrics/victoria-metrics:v1.149.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8167 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/victoriametricsdocker/`.
