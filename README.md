<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="MariaDB logo" width="128">
  </picture>
</p>

<h1 align="center">aac-kea</h1>

<p align="center">Self-hosted deployment definition for the global database backend of ISC Kea DHCP (MariaDB).</p>

<p align="center">
  <a href="https://kea.isc.org"><img alt="Upstream" src="https://img.shields.io/badge/upstream-ISC%20Kea-blue"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-mariadb%3A11.4-003545?logo=mariadb&logoColor=white">
  <a href="https://github.com/MariaDB/server/blob/main/COPYING"><img alt="License" src="https://img.shields.io/badge/license-GPL--2.0-green"></a>
</p>

## About

This repository defines a single MariaDB container that acts as a shared, central database for
ISC Kea DHCP servers. Kea stores host reservations and active leases in a SQL backend; this
deployment provides that backend so that several Kea servers can use one database.

The repository contains only the database. It does not run a `kea-dhcp4` daemon or a Stork agent.
The Kea servers themselves are deployed separately and connect to this database over the network.

Key features:

- MariaDB 11.4 with a persistent data volume
- Database and user created on first start from environment variables
- Container healthcheck via `mariadb-admin ping`
- Port published for remote Kea servers to reach the database

## Upstream

- Project (Kea DHCP): <https://kea.isc.org>
- Database project (MariaDB): <https://mariadb.org>, source at <https://github.com/MariaDB/server>
- Container image: `mariadb:11.4` (Docker Hub, official image)
- License: GPL-2.0 for MariaDB (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine.
`service.yml` declares no external networks, so no network needs to be created for a manual run.

```bash
# Provide the secrets listed under "Configuration" in .env (see below)
docker compose up -d
docker compose ps
```

Secrets are not stored in this repository. Provide them through the deployment pipeline or
create a `.env` file yourself. Do not commit it.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `CHANGE_ME` (secret) | Password of the MariaDB `root` user |
| `MYSQL_DATABASE` | `kea` | Database created on first start |
| `MYSQL_USER` | `kea` | Database user created on first start |
| `MYSQL_PASSWORD` | `CHANGE_ME` (secret) | Password of the database user |
| `TZ` | `Europe/Berlin` | Time zone |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `3333` (host) -> `3306` (container) | TCP | MySQL/MariaDB protocol for remote Kea servers and monitoring |

Volumes:

| Mount | Description |
|---|---|
| `db` -> `/var/lib/mysql` | MariaDB data directory (host reservations and leases) |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment, volumes
   and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-kea>.

## Credits and license

Kea DHCP is developed by ISC and MariaDB by the MariaDB Foundation. This repository only contains
the deployment definition for the MariaDB container and is not a fork of either project.
The logo is provided by the selfh.st icon collection; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md)
for icon sources and licenses.
