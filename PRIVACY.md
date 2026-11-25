# Privacy Policy — VictoriaMetrics TSDB for TerraMaster TOS 7

App ID: `victoriametricsdocker`
Package version: 1.157.0-1
Last updated: 2026-11-25

This policy describes what the VictoriaMetrics TSDB application package does with
data on your NAS. The application runs entirely on hardware you control. The
packager (Moechz) does not operate a service that receives your metrics and does
not have access to your NAS.

## 1. Summary

- **No telemetry, analytics, advertising, crash reporting or license activation.**
- **No version check.** The container does not ask any server whether a newer
  VictoriaMetrics release exists.
- **No account and no sign-up.** There is no packager-operated backend.
- Metrics are stored only in the application data folder on your NAS.
- The only network connections are the ones you configure yourself, plus the
  Docker image pull performed by the TOS App Center when you install or update
  the app.

## 2. What data the application processes

VictoriaMetrics is a time-series database. It processes the data that you or
your monitoring agents send to it:

- metric names, label names and label values;
- numeric samples and their timestamps;
- optional metadata (metric type and help text) when the sender provides it;
- technical data needed to operate the service: HTTP request metadata (such as
  the client IP address and request path) and diagnostic messages written to the
  container log (stdout/stderr) when an error or a slow query occurs.

The package does not create user accounts, does not collect names, e-mail
addresses, phone numbers or payment data, and does not read files outside its
own data directory.

## 3. Where the data is stored

All persistent data is stored on the NAS in:

```
/Volume1/DockerAppData/victoriametricsdocker/data
```

This directory is mounted into the container as `/victoria-metrics-data` and
contains the storage parts, the index, metadata and the configuration written by
VictoriaMetrics. The container's own `/tmp` is discarded with the container.
The application does not write anywhere else on the NAS.

The browser-based UI (vmui) may keep interface preferences such as the selected
theme or time range in your browser's local storage. Those values stay in your
browser and are not sent to the packager.

## 4. Retention period and deletion

- **Default retention: 12 months.** The package starts VictoriaMetrics with
  `-retentionPeriod=12` (months), so data older than 12 months is removed
  automatically by the database.
- The retention period can be changed by editing the `-retentionPeriod` argument
  in the application's `docker-compose.yml` and restarting the container.
- Individual time series can be deleted through the VictoriaMetrics admin API
  (`/api/v1/admin/tsdb/delete_series`). Because this package does not configure
  `-deleteAuthKey` or HTTP authentication, anyone who can reach port 8167 can
  call that API, so keep the app on a trusted network.
- **Uninstalling with data removal** deletes
  `/Volume1/DockerAppData/victoriametricsdocker` (including all metrics).
  **Uninstalling without data removal** keeps the folder so that a later
  reinstall can reuse the data. You can also stop the container and delete the
  folder manually at any time.

Backups are the responsibility of the NAS administrator.

## 5. Network connections and third parties

The application makes **no outbound connection by default**. There is no
telemetry endpoint, no analytics SDK, no version check and no license server.

The table lists every endpoint that can be involved, when it is contacted and
what it receives.

| Endpoint | When it is contacted | Data involved | Destination |
|---|---|---|---|
| `registry-1.docker.io` (or the registry mirror configured in DockerEngine) | When the TOS App Center installs or updates the app and pulls the image | Image manifest and layers. No metrics are sent. | Docker Hub, or the mirror you configured |
| A Prometheus scrape target that **you** configure | Only if an administrator adds `-promscrape.config` to the container. This package does **not** enable scraping and ships no scrape configuration. | An HTTP GET to the target's `/metrics` endpoint. The target sees the NAS IP address and the request path. | A host on your own network |
| `-pushmetrics.url` | Only if an administrator explicitly sets this flag. The package does not set it. | The contents of the local `/metrics` page (VictoriaMetrics' own operational counters). | The URL chosen by the administrator |
| Documentation links (`docs.victoriametrics.com`, `victoriametrics.com`, `github.com/VictoriaMetrics/...`) | Only when a user clicks a link in the root page or in vmui. No request is made automatically. | The documentation page request. | Rendered by the user's browser, not by the container |

There is no other sharing. The packager does not sell, rent or disclose data,
because the packager never receives it.

## 6. Your rights

Because the data stays on your NAS, you can exercise the following rights
directly and without contacting the packager:

- **Access** — read the data directory, browse the metrics in vmui, or query the
  API (`/api/v1/query`, `/api/v1/series`).
- **Export / portability** — export raw samples with `/api/v1/export` or
  `/api/v1/export/csv`, or query them with `/api/v1/query_range`.
- **Rectification** — metrics are immutable records. To correct data, delete the
  affected series and ingest corrected samples.
- **Erasure** — delete individual series with
  `/api/v1/admin/tsdb/delete_series`, or delete the whole data directory after
  stopping the container (see section 4).
- **Restriction / objection** — stop the container, remove the data directory, or
  block port 8167 with the NAS firewall.

If you believe the package itself is mishandling data, open an issue at
<https://github.com/Moechz/victoriametrics/issues> and describe the problem
without including sensitive metric values.

## 7. Security measures

- The container runs as the unprivileged user `1000:1000`, never as root.
- All Linux capabilities are dropped (`cap_drop: ALL`) and
  `no-new-privileges:true` is set.
- No privileged mode, no host networking, no Docker socket, no added
  capabilities.
- The upstream image is the official Docker Hub image, pinned to the fixed tag
  `v1.157.0` (multi-architecture).
- The application does not ship or enable authentication. The web UI and the
  ingestion/query APIs are plain HTTP on port 8167. Run the app on a trusted LAN,
  restrict access with the NAS firewall, and put a TLS reverse proxy with
  authentication in front of it before exposing it to the internet.
- Container logs may contain client IP addresses and metric names. They are
  stored by Docker on the NAS and are protected by the NAS account system.
- Keeping the package updated is recommended. The latest release is published at
  <https://github.com/Moechz/victoriametrics/releases>.

## 8. Children

The application is an infrastructure tool. It is not directed at children and
does not knowingly collect personal data from children.

## 9. Changes to this policy

Changes are published in this file and in the repository history at
<https://github.com/Moechz/victoriametrics>. A material change to data handling
will be described in `CHANGELOG.md` and in the release notes of the affected
version.

## 10. Contact

Packager: Moechz
Issue tracker: <https://github.com/Moechz/victoriametrics/issues>

This is an independent community package. It is not affiliated with
VictoriaMetrics or TerraMaster. VictoriaMetrics is a trademark of its owners.
