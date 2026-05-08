# geolocation-dbs

Mirror of MaxMind's [GeoLite2-City](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data/)
database, refreshed twice weekly and consumed by the openZro management
server's geolocation lookup (peer IP → country/city, used by posture
checks and dashboard analytics).

> **MaxMind retains ownership of the GeoLite2 data.** This repo only
> redistributes the upstream archives. Use of the data is governed by
> the [GeoLite2 End User License Agreement](https://www.maxmind.com/en/geolite2/eula).
> Operators who prefer to fetch directly from MaxMind can pass
> `--maxmind-license-key` to `openzro management` instead — see the
> [self-hosted geolocation guide](https://docs.openzro.io/selfhosted/geolocation).

## How it's used

The openZro management server, on cold boot, downloads the MMDB and CSV
archives from this repo's latest release. URLs:

```
https://github.com/openzro/geolocation-dbs/releases/latest/download/GeoLite2-City.tar.gz
https://github.com/openzro/geolocation-dbs/releases/latest/download/GeoLite2-City.tar.gz.sha256
https://github.com/openzro/geolocation-dbs/releases/latest/download/GeoLite2-City-CSV.zip
https://github.com/openzro/geolocation-dbs/releases/latest/download/GeoLite2-City-CSV.zip.sha256
```

The management server verifies the SHA-256, extracts the archive, and
stamps the on-disk file with the current date (`GeoLite2-City_YYYYMMDD.mmdb`).

## Refresh cadence

A scheduled GitHub Actions workflow (`.github/workflows/refresh.yml`)
runs every Tuesday and Friday at 06:00 UTC, ~2 hours after MaxMind's
own publish window. Each run:

1. Downloads the latest tarball + checksum from
   `download.maxmind.com/app/geoip_download` using
   `secrets.MAXMIND_LICENSE_KEY` (scoped to the `production`
   environment).
2. Verifies the checksum against MaxMind's published SHA-256.
3. Re-emits the `.sha256` files with un-dated filenames to match our
   asset names (so `sha256sum -c` works directly).
4. Publishes a release tagged `geolite2-YYYYMMDD`, marked as `latest`.

Manual runs are available via `Actions → Refresh GeoLite2 → Run workflow`.

## Why this exists

The openZro fork inherited `pkg.openzro.io/geolocation-dbs/` from a
NetBird-era constant that had never been built. Air-gapped operators
would hit a 404 on first boot, and the dashboard would lack country
labels until the operator manually staged an mmdb. This repo is the
mirror that constant was always pointing at.

NetBird Inc. ships their own equivalent at `pkgs.netbird.io/geolocation-dbs/`
with the same posture (mirror + EULA disclaimer). We use a public
GitHub Releases endpoint instead of running our own CDN to keep the
infrastructure surface small.

## Repo license vs data license

The contents of this repo (workflow, README) are BSD-3-Clause. The
GeoLite2 archives published as release assets are MaxMind's data and
are governed by their EULA, **not** the repo's BSD-3 license. The
LICENSE file at the root applies only to the source files in the git
tree, not to the binary release assets.
