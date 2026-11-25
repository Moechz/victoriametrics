# Changelog

All notable user-visible changes to the VictoriaMetrics TOS application package.

## 1.157.0-1 — 2026-11-25

Review remediation for the on-hold 1.149.0-1 submissions (x86_64 and aarch64),
plus the upstream upgrade. "Rectified per review comments."

### Changed
- **T1 — upstream upgrade.** Image updated from `victoriametrics/victoria-metrics:v1.149.0`
  to `victoriametrics/victoria-metrics:v1.157.0` (released 2026-11-20), a
  multi-architecture image (`linux/amd64` + `linux/arm64`) used unmodified.
- **F2 — startup.** The launch flags are now passed under `command:` as a YAML
  list and the official binary `/victoria-metrics-prod` is pinned as the
  entrypoint, so PID 1 is never `-storageDataPath=...` and the container no
  longer fails with `runc create failed`.
- **P3 — duplicate name.** Display name changed from "VictoriaMetrics" to
  "VictoriaMetrics TSDB" (zh-cn "VictoriaMetrics 时序库", zh-hk
  "VictoriaMetrics 時序資料庫") so the app can be told apart from the other
  listings of the same name.
- **S1 hardening.** `cap_drop: [ALL]` and `no-new-privileges:true` added while
  the container keeps running as uid 1000.

### Added
- **C3/C4 — privacy policy.** New `PRIVACY.md` covers what is stored, where,
  retention (12 months), user rights, security measures, third-party sharing,
  the deletion channel and contact. The package Help field, `tosapp.json` and
  the README point to it.
- **C8 — third-party disclosure.** PRIVACY.md states that VictoriaMetrics has no
  telemetry, analytics, crash reporting or version check, and lists the only
  endpoints that can be involved (Docker Hub image pull, user-configured scrape
  targets, optional `-pushmetrics.url`, documentation links).
- **C10 — repository docs.** README now has repository layout, privacy, notices,
  build, verification, and data/uninstall sections. New `NOTICE.md` records the
  Apache-2.0 upstream license, the image tag and digest, the trademark and the
  packaging license. The repository stays public.

### Fixed
- **C9 — description.** The description now matches what the package really
  does. The root page returns an HTML page that links to vmui instead of
  redirecting, and the protocol list only names the ingestion endpoints this
  package actually exposes.
- **L01/L03 — language file.** All 23 sections define non-empty
  `name`/`auth`/`descript`/`release_note`/`important`, the file uses LF line
  endings exclusively, and it contains no ASCII semicolon (the App Center parses
  it as an INI comment).
- **Stale metadata.** Removed the `probe` block and the leftover
  `entrypoint_file` entry that pointed at a non-existent script.

## 1.149.0-1 — 2026-11-15

### Added
- First release. VictoriaMetrics (`victoriametrics/victoria-metrics:v1.149.0`,
  Docker Hub official image, unmodified) packaged as a TOS 7 Docker
  application, published on port 8167 and running as a non-root user
  (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/victoriametricsdocker/`.
