# victoriametrics-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics).

## What this is

A four-file TOS Docker application archive (`victoriametricsdocker.tar.gz`) that installs
VictoriaMetrics on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `victoriametricsdocker`)
- `victoriametricsdocker.lang` — 23-language superset (TOS requires 14)
- `victoriametricsdocker.svg` — application icon (1113 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `victoriametrics/victoria-metrics:v1.149.0`

## How it works

| | |
|---|---|
| Container image | `victoriametrics/victoria-metrics:v1.149.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8167/` (container port 8428) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/victoria-metrics-data` → `/Volume1/DockerAppData/victoriametricsdocker/data` |
| Logs | `docker logs victoriametricsdocker` (stdout/stderr) |

`/victoria-metrics-data` (mounted from `DockerAppData/victoriametricsdocker/data`) holds the storage parts, index and metadata.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 8428.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Health check without HTTP tools.** The image has `sh` and `grep` but no curl/wget, so the check greps the listening port (8428 = 0x20EC) in `/proc/net/`.
- **One port for API and UI.** Prometheus remote-write, PromQL API and vmui share port 8428.

## Privacy

This application runs entirely on your NAS. It does not include analytics, telemetry or advertising SDKs, and the packaging contains no code that sends data to the packager. Data is only exchanged with the services you configure yourself (for example media servers, notification endpoints or model APIs), and only from the container. Everything the application stores lives in the application data folder described above; removing the app with its data deletes it.

## Third-party components

The package runs the upstream image `victoriametrics/victoria-metrics:v1.149.0` unmodified. Its components and their licences are those of the upstream project (https://github.com/VictoriaMetrics/VictoriaMetrics), which is distributed under **Apache-2.0**. No upstream source code is redistributed in this repository; only the four packaging files are original work.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/victoriametricsdocker/data` | VictoriaMetrics application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/victoriametricsdocker.tar.gz + .sha256
scripts/build.sh 1.149.0-2   # packaging iteration bump
scripts/build.sh 1.149.0-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/victoriametricsdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v1.149.0-1`**, assets `victoriametricsdocker.tar.gz` +
   `victoriametricsdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `victoriametricsdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `1.149.0-1` → automated validation → review.

## License / branding

VictoriaMetrics is distributed by its upstream authors under **Apache-2.0**. The container image is the official image from Docker Hub (`victoriametrics/victoria-metrics:v1.149.0`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- It stores metrics; exporters or agents must send data to it.
- The UI is plain HTTP; use a TLS reverse proxy before exposing it to the internet.
- Retention and disk usage are your responsibility; watch free space on the NAS.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
