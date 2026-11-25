# VictoriaMetrics TSDB for TerraMaster TOS 7

> TerraMaster TOS 7 App Center **Docker application** package for
> [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics), the
> fast, low-memory time-series database and monitoring solution.

This is an independent community package by **Moechz**. It is not affiliated
with VictoriaMetrics or TerraMaster.

## What this is

A four-file TOS Docker application archive (`victoriametricsdocker.tar.gz`) that
installs the official VictoriaMetrics single-node image on TOS 7 through the App
Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id
  `victoriametricsdocker`, display name "VictoriaMetrics TSDB")
- `victoriametricsdocker.lang` — 23-language superset (TOS requires 14)
- `victoriametricsdocker.svg` — application icon
- `docker-compose.yml` — one service running the official image unmodified

The application receives metrics over Prometheus remote-write and the HTTP
ingestion endpoints (InfluxDB line protocol, OpenTelemetry, DataDog), stores them
locally, and serves PromQL/MetricsQL queries plus the vmui web UI on one port.

## Version and image

| | |
|---|---|
| Package version | `1.157.0-1` |
| Upstream version | VictoriaMetrics `v1.157.0` (released 20 Nov 2026) |
| Container image | `victoriametrics/victoria-metrics:v1.157.0` (Docker Hub, used unmodified) |
| Architectures | `x86_64` and `aarch64` packages, built from one multi-architecture image |
| Open | `http://<NAS-IP>:8167/` (container port 8428). The root page links to `vmui`; the UI is also at `/vmui/`. |
| Runs as | non-root, `user: "1000:1000"`, `cap_drop: ALL`, `no-new-privileges` |
| Persistent data | `/victoria-metrics-data` → `/Volume1/DockerAppData/victoriametricsdocker/data` |
| Logs | `docker logs victoriametricsdocker` (stdout/stderr) |

## How it works

- **Startup flags are arguments, not an entrypoint.** The official binary
  `/victoria-metrics-prod` is pinned as the entrypoint and the flags
  (`-storageDataPath`, `-httpListenAddr`, `-retentionPeriod`) are passed under
  `command:`. Docker therefore never tries to exec a flag as PID 1.
- **One port for everything.** Remote-write, the HTTP ingestion endpoints, the
  query API and vmui all use port 8428. The package publishes host port 8167.
- **No telemetry.** The binary has no analytics, crash-reporting or version-check
  endpoint. `-selfScrapeInterval` and `-pushmetrics.url` are left unset, so the
  container makes no outbound connection unless you configure scrape targets.
- **Health check** probes the listening port (8428 = `0x20EC`) in `/proc/net/`,
  which needs only `sh` and `grep` and works on every architecture.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip
  without a timestamp, then `tools/verify.py` runs the review-standards
  self-check.

## Repository layout

```text
.
├── src/                           # the four files that go into the package
│   ├── config.ini
│   ├── docker-compose.yml
│   ├── victoriametricsdocker.lang
│   └── victoriametricsdocker.svg
├── scripts/
│   ├── build.sh                   # build one architecture
│   └── build-dual.sh              # build x86_64 + aarch64, official check
├── tools/
│   └── verify.py                  # review-standards self-check (build gate)
├── test/
│   └── install-and-verify.sh      # on-device E2E (dedicated test NAS only)
├── tosapp.json                    # package specification (version, image, metadata)
├── PRIVACY.md                     # privacy policy
├── NOTICE.md                      # licenses, image digest, trademarks
├── CHANGELOG.md
├── LICENSE
└── README.md
```

The repository is public so that the package can be reproduced from source.

## Privacy

The application runs entirely on your NAS. It does **not** include analytics,
telemetry, advertising or a version check, and the packaging contains no code
that sends data to the packager. Metrics are stored only in the application data
folder. The only network connections are the Docker image pull performed by the
App Center at install/update time and the endpoints you configure yourself.

The complete policy — what is stored, where, for how long, which third-party
endpoints can be involved, your rights, security measures and the deletion
channel — is in [PRIVACY.md](PRIVACY.md). The package Help link points there.

## Notices

VictoriaMetrics is distributed under **Apache-2.0**. The package runs the
official Docker Hub image unmodified and redistributes no upstream source code.
The icon is the upstream logo, used only to identify the software. The packaging
itself is MIT-licensed. See [NOTICE.md](NOTICE.md) and [LICENSE](LICENSE) for the
full attribution, image tag and digest, and trademark statement.

## Build

```bash
scripts/build.sh 1.157.0-1            # -> out/victoriametricsdocker.tar.gz + .sha256
scripts/build.sh 1.157.0-1 aarch64    # build one architecture explicitly
scripts/build-dual.sh 1.157.0-1       # both architectures + official check
```

`build-dual.sh` writes:

```text
out/victoriametricsdocker_x86_64.tar.gz
out/victoriametricsdocker_x86_64.tar.gz.sha256
out/victoriametricsdocker_aarch64.tar.gz
out/victoriametricsdocker_aarch64.tar.gz.sha256
```

Every build stages the four files, substitutes the version/platform
placeholders, normalises line endings, then runs `tools/verify.py`. That check
covers the archive layout, JSON validity, required fields, 14+ language
coverage, version consistency, Docker-Hub-only images, a fixed tag,
reserved/recommended host ports, the non-root user, per-service
healthcheck/TZ/restart, the data path below the application data root, the
`/Volume*` wildcard, comments inside sequence blocks, `x-app-meta` position,
literal secrets and the SVG size/element limits.

## Verification

1. **Build self-check** — `tools/verify.py`, run by `scripts/build.sh`.
2. **Official package check** — both archives must pass:

   ```bash
   ./TOSAppSelfTestingTool -c out/victoriametricsdocker_x86_64.tar.gz
   ./TOSAppSelfTestingTool -c out/victoriametricsdocker_aarch64.tar.gz
   ```

3. **Isolated staging** (from the `issue-watch` workspace, not this repository):

   ```bash
   python3 stage.py work/victoriametrics
   ```

   It rewrites the compose to random loopback ports and a staging data folder,
   runs `docker compose up -d`, asserts running/healthy/HTTP/non-root, then
   removes everything. It never touches the production data directory.

`test/install-and-verify.sh` is an on-device E2E script for a **dedicated test
NAS**. It installs the package and deletes
`/Volume1/DockerAppData/victoriametricsdocker` at the end, so do not run it on a
NAS that has a live instance.

## Data and uninstall

- Data lives in `/Volume1/DockerAppData/victoriametricsdocker/data`.
- A normal uninstall (or `docker compose -p victoriametricsdocker down`) keeps
  that folder, so a reinstall reuses the metrics.
- Uninstalling **with data removal**, or deleting the folder manually, erases all
  metrics. Back up the folder first if you need the data.
- Retention defaults to 12 months (`-retentionPeriod=12`). Change the flag in
  the compose file and restart the container to change it.
- The web UI and API are plain HTTP without authentication. Keep the app on a
  trusted LAN, or put a TLS reverse proxy with authentication in front of it.

## Store submission

1. Public GitHub repository with the package sources and this README.
2. Release tag `v1.157.0-1` with assets
   `victoriametricsdocker_x86_64.tar.gz` / `.sha256` and
   `victoriametricsdocker_aarch64.tar.gz` / `.sha256`.
3. Submit version `1.157.0-1` for both architectures and compare the platform's
   `file_hash` with the local `sha256sum`.

## License / branding

VictoriaMetrics is distributed by its upstream authors under **Apache-2.0**. The
container image is the official image from Docker Hub and is used unmodified; no
upstream source code is redistributed here. This packaging is an independent
community submission by Moechz and is not affiliated with the upstream project
or TerraMaster. The packaging files are MIT-licensed.

## Limitations

- It stores metrics; exporters or agents must send data to it.
- The UI is plain HTTP and has no built-in authentication; use a TLS reverse
  proxy and network restrictions before exposing it.
- Retention and disk usage are your responsibility; watch free space on the NAS.
- Metrics are stored as received; deleting individual series requires the admin
  API.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
