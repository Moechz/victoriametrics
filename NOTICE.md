# Notices — VictoriaMetrics TSDB package

## Upstream software

- Project: **VictoriaMetrics** (single-node `victoria-metrics`)
- Homepage: <https://github.com/VictoriaMetrics/VictoriaMetrics>
- Documentation: <https://docs.victoriametrics.com/>
- Copyright: the VictoriaMetrics authors and contributors
- License: **Apache License 2.0**
  (<https://github.com/VictoriaMetrics/VictoriaMetrics/blob/master/LICENSE>)

No upstream source code is redistributed in this repository. The package runs
the official container image unmodified.

## Container image

- Image: `docker.io/victoriametrics/victoria-metrics:v1.157.0`
- Tag: `v1.157.0` (fixed, never `latest`)
- Multi-architecture: `linux/amd64` and `linux/arm64` were both verified when
  this package was built. Docker reported the multi-architecture image digest
  `sha256:322a7a92fdcc269280e9c38af94c83bc76827a31b792acbc60e44394e7f29c20`
  for this tag.
- The image is used as published on Docker Hub. Only the TOS packaging files in
  `src/` are original work.

## Trademarks and icon

"VictoriaMetrics" and the VictoriaMetrics logo are trademarks of their owners.
The icon `src/victoriametricsdocker.svg` reproduces the upstream logo solely to
identify the software in the TerraMaster App Center. This package is an
independent community submission by Moechz and is **not** affiliated with,
endorsed by, or sponsored by VictoriaMetrics or TerraMaster.

## Packaging license

The packaging files (the four files in `src/`, the build scripts and the
documentation in this repository) are released under the MIT License. See
[`LICENSE`](LICENSE).

## Bundled validation tool

`TOSAppSelfTestingTool` is an official TerraMaster utility, redistributed here
unmodified for local package validation. It is not part of the installed
application and is not covered by the MIT license above.

## Privacy

The application sends no telemetry and performs no version check. See
[`PRIVACY.md`](PRIVACY.md) for the complete data-handling description.
